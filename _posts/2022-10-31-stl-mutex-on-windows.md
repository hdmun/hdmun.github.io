---
layout: post
title: "Microsoft C++ stl mutex 내부 구현 파헤치기"
description: C++ STL `std::mutex` 내부 구현에 대해서 알아봅니다.
categories: cpp
tags: [cpp, stl, windows]
author: hdmun
comments: true
---

Visual C++ 멀티 스레드 환경의 어플리케이션에서 동일한 메모리 접근을 동기화하기 위해 `CriticalSection` API를 많이 사용하실텐데요.

[MSDN/InitializeCriticalSection](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-initializecriticalsection)  
[MSDN/InitializeCriticalSectionAndSpinCount](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-initializecriticalsectionandspincount)  
[MSDN/DeleteCriticalSection](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-deletecriticalsection)  
[MSDN/EnterCriticalSection](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-entercriticalsection)  
[MSDN/LeaveCriticalSection ](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-leavecriticalsection)  

C++11 이후 `STL`에서는 `mutex`라는 클래스를 제공해주고 있습니다.

어느덧 C++11이 나온 지도 10년이 훌쩍 넘어 예전부터 알고는 있었지만 따로 사용해 본 적은 없었는데요.

어떤 방식으로 구현되어있는지 궁금하던 시점에 Microsoft Github에 코드가 공개된 걸 알게 되어 저장소에 공개된 코드를 보면서 의식의 흐름대로 정리해보고자 합니다.

[microsoft/STL Github 저장소](https://github.com/microsoft/STL)

참고로 `std::mutex` 클래스를 기준으로 작성하였으며 아래 클래스들과의 차이점에 대해서는 다루지 않습니다.

- `std::recursive_mutex`
- `std::timed_mutex`
- `std::recursive_timed_mutex`

## mutex 사용법

글의 흐름을 위해 사용법은 간단히 짚고 넘어가겠습니다.

예제 코드는 아래 링크에서 퍼왔습니다.  
<https://en.cppreference.com/w/cpp/thread/mutex>

사용 방법은 간단합니다.  
공유 자원에 접근하기 전에 `lock()` 함수를 호출해 다른 스레드에서 접근했을때 대기하도록 처리하고 접근이 끝났다면 `unlock()` 함수를 호출해 잠금을 풀어주면 됩니다.

```cpp
// 헤더 생략

std::map<std::string, std::string> g_pages;
std::mutex g_pages_mutex;

void save_page(const std::string &url)
{
    // simulate a long page fetch
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::string result = "fake content";
 
    // std::lock_guard<std::mutex> guard(g_pages_mutex);
    // 설명을 위해 명시적 호출로 변경
    g_pages_mutex.lock();  // 잠금
    g_pages[url] = result;
    g_pages_mutex.unlock();  // 해제
}

int main()
{
    std::thread t1(save_page, "http://foo");
    std::thread t2(save_page, "http://bar");
    t1.join();
    t2.join();
}
```

## mutex 클래스

[mutex#L86](https://github.com/microsoft/STL/blob/main/stl/inc/mutex#L86)

`mutex` 클래스는 단순한 구조로 선언되어있습니다.  
`_Mutex_base` 클래스를 상속 받고 디폴트 복사 생성자와 대입 연산자를 삭제하는 코드가 전부인데요.

```cpp
_EXPORT_STD class mutex : public _Mutex_base { // class for mutual exclusion
public:
    /* constexpr */ mutex() noexcept // TRANSITION, ABI
        : _Mutex_base() {}

    mutex(const mutex&)            = delete;
    mutex& operator=(const mutex&) = delete;
};
```

내부 구현이 궁금하므로 `_Mutex_base` 클래스를 파해쳐 보겠습니다.

## _Mutex_base 클래스

정렬된 메모리 구조체 타입의 `_Mtx_storage`를 멤버로 가지고 있으며 `_Mymtx()` 함수를 통해 `_Mtx_t`로 캐스팅 된 상태로 접근이 가능합니다.

`_Mymtx()` 함수로 초기화, 잠금, 해제 함수에 인자로 전달하고 있습니다.

- `_Mtx_init_in_situ`
- `_Mtx_destroy_in_situ`
- `_Mtx_lock`
- `_Mtx_trylock`
- `_Mtx_unlock`

[mutex#L36](https://github.com/microsoft/STL/blob/main/stl/inc/mutex#L36)

```cpp
// 일부 코드는 생략했습니다.

class _Mutex_base {
public:
    _Mutex_base(int _Flags = 0) noexcept {
        _Mtx_init_in_situ(_Mymtx(), _Flags | _Mtx_try);
    }

    ~_Mutex_base() noexcept {
        _Mtx_destroy_in_situ(_Mymtx());
    }


    void lock() {
        _Check_C_return(_Mtx_lock(_Mymtx()));
    }

    _NODISCARD_TRY_CHANGE_STATE bool try_lock() {
         const auto _Res = _Mtx_trylock(_Mymtx());
        switch (_Res) {
        case _Thrd_success:
            return true;
        case _Thrd_busy:
            return false;
        default:
            _Throw_C_error(_Res);
        }
    }

    void unlock() {
        _Mtx_unlock(_Mymtx());
    }

private:
    _Aligned_storage_t<_Mtx_internal_imp_size, _Mtx_internal_imp_alignment> _Mtx_storage;

    _Mtx_t _Mymtx() noexcept {
        return reinterpret_cast<_Mtx_t>(&_Mtx_storage);
    }
};
```

## _Aligned_storage_t 구조

`_Mutex_base` 클래스의 유일한 멤버 변수인 `_Aligned_storage_t` 타입은 선언된 템플릿을 쫓아가다 보면 최종적으로 아래와 같은 타입으로 컴파일 되는걸 확인 할 수 있습니다.

[type_traits#L1022](https://github.com/microsoft/STL/blob/main/stl/inc/type_traits#L1022)

```cpp
template <class _Ty, size_t _Len>
union _Align_type { // union with size _Len bytes and alignment of _Ty
    _Ty _Val;
    char _Pad[_Len];
};
```

## _Mtx_t 구조

`_Mymtx()` 함수에서 리턴해주는 `_Mtx_t` 타입 구조에 대해서도 간단히 확인해보겠습니다.

`using` 키워드로 아래와 같이 선언이 되어있는데요.

[xthreads.h#L55](https://github.com/microsoft/STL/blob/main/stl/inc/xthreads.h#L55)

```cpp
using _Mtx_t = struct _Mtx_internal_imp_t*;
```

정의된 부분을 확인해보면 다음과 같이 4가지 멤버를 가지고 있습니다.

[mutex.cpp#L39](https://github.com/microsoft/STL/blob/main/stl/src/mutex.cpp#L39)

```cpp
struct _Mtx_internal_imp_t {
    int type;
    typename std::_Aligned_storage<Concurrency::details::stl_critical_section_max_size,
        Concurrency::details::stl_critical_section_max_alignment>::type cs;
    long thread_id;
    int count;
    Concurrency::details::stl_critical_section_interface* _get_cs() { // get pointer to implementation
        return reinterpret_cast<Concurrency::details::stl_critical_section_interface*>(&cs);
    }
};
```

구조를 보다보면 `stl_critical_section_interface*`를 반환하는 `_get_cs()` 함수가 보이는데요.

`stl_critical_section_interface`은 아래와 같이 순수 가상함수를 가진 클래스로 선언되어 있습니다.

[primitives.hpp#L13](https://github.com/microsoft/STL/blob/main/stl/src/primitives.hpp#L13)

```cpp
// 일부 코드 생략
namespace Concurrency {
    namespace details {
        class __declspec(novtable) stl_critical_section_interface {
        public:
            virtual void lock()                     = 0;
            virtual bool try_lock()                 = 0;
            virtual bool try_lock_for(unsigned int) = 0;
            virtual void unlock()                   = 0;
            virtual void destroy()                  = 0;
        };
    }
}
```

지금까지 확인된 내용으로보면 다음과 같이 유추할 수 있습니다.

- `std::mutex`는 멤버 변수를 가지고 있지 않으며 `_Mutex_base`를 상속 받는다.
- `_Mutex_base`는 `_Mtx_storage`를 유일한 멤버로 가지며 `_Mtx_t`으로 타입 캐스팅을 통해 접근할 수 있다.
- `_Mtx_t`의 실제 타입인 `_Mtx_internal_imp_t`의 멤버로 잠금 상태를 관리할 것이다.
- `_get_cs()` 함수에서 리턴하는 `stl_critical_section_interface` 타입으로 초기화, 잠금, 해제 동작을 할 것이다.


## init, destory, lock, unlock 구현

`mutex`가 사용하는 데이터 구조를 확인했으니 어떤식으로 동작하도록 구현되어있는지 함수를 쫓아가 보겠습니다.

### 초기화

[mutex.cpp#L53](https://github.com/microsoft/STL/blob/main/stl/src/mutex.cpp#L53)

```cpp
void _Mtx_init_in_situ(_Mtx_t mtx, int type) { // initialize mutex in situ
    Concurrency::details::create_stl_critical_section(mtx->_get_cs());
    mtx->thread_id = -1;
    mtx->type      = type;
    mtx->count     = 0;
}
```

[primitives.hpp#L104](https://github.com/microsoft/STL/blob/main/stl/src/primitives.hpp#L104)

```cpp
namespace Concurrency {
    namespace details {
        inline void create_stl_critical_section(stl_critical_section_interface* p) {
            new (p) stl_critical_section_win7;
        }
    }
}
```

`create_stl_critical_section` 함수를 호출해 `stl_critical_section_win7` 객체를 생성하는 코드가 보이는데요

<details>
<summary>`stl_critical_section_win7` 코드 보기</summary>
<div markdown="1">

[primitives.hpp#L31](https://github.com/microsoft/STL/blob/main/stl/src/primitives.hpp#L31)

```cpp
namespace Concurrency {
    namespace details {
        class stl_critical_section_win7 final : public stl_critical_section_interface {
        public:
            stl_critical_section_win7() {
                InitializeSRWLock(&m_srw_lock);
            }

            ~stl_critical_section_win7()                                           = delete;
            stl_critical_section_win7(const stl_critical_section_win7&)            = delete;
            stl_critical_section_win7& operator=(const stl_critical_section_win7&) = delete;

            void destroy() override {}

            void lock() override {
                AcquireSRWLockExclusive(&m_srw_lock);
            }

            bool try_lock() override {
                return TryAcquireSRWLockExclusive(&m_srw_lock) != 0;
            }

            bool try_lock_for(unsigned int) override {
                // STL will call try_lock_for once again if this call will not succeed
                return stl_critical_section_win7::try_lock();
            }

            void unlock() override {
                ReleaseSRWLockExclusive(&m_srw_lock);
            }

            PSRWLOCK native_handle() {
                return &m_srw_lock;
            }

        private:
            SRWLOCK m_srw_lock;
        };
    }
}
```
</div>
</details>

<br/>

`stl_critical_section_win7` 클래스 코드를 확인해보면 `SRWLock` API를 사용하고 있습니다.

읽기/쓰기 두 가지 잠금 모드를 지원하는 윈도우7부터 추가된 슬림 리더/라이터 락이라고 불리는 API 함수입니다.

위 클래스에서는 쓰기 모드인 `Exclusive` API 함수만 사용하는걸로 확인이 되네요

[Slim Reader/Writer (SRW) Locks](https://learn.microsoft.com/en-us/windows/win32/sync/slim-reader-writer--srw--locks)


### lock, unlock

`lock`의 경우 두 가지 함수가 제공되고 있고 둘 다 `mtx_do_lock` 함수를 호출하고 있습니다.

두 번째 인자 `const xtime* target`의 전달 유무로 호출 방법이 나뉘고 있는데요 `mtx_do_lock` 함수를 보면서 차이점에 대해 확인해보겠습니다.

[mutex.cpp#L162](https://github.com/microsoft/STL/blob/main/stl/src/mutex.cpp#L162)

```cpp
int _Mtx_unlock(_Mtx_t mtx) {
    _THREAD_ASSERT(
        1 <= mtx->count && mtx->thread_id == static_cast<long>(GetCurrentThreadId()), "unlock of unowned mutex");

    if (--mtx->count == 0) {
        mtx->thread_id = -1;
        mtx->_get_cs()->unlock();
    }
    return _Thrd_success;
}

int _Mtx_lock(_Mtx_t mtx) {
    return mtx_do_lock(mtx, nullptr);
}

int _Mtx_trylock(_Mtx_t mtx) {
    xtime xt;
    _THREAD_ASSERT((mtx->type & (_Mtx_try | _Mtx_timed)) != 0, "trylock not supported by mutex");
    xt.sec  = 0;
    xt.nsec = 0;
    return mtx_do_lock(mtx, &xt);
}
```

`mtx_do_lock` 함수는 크게 두 단계 분기처리로 동작하고 있습니다.

[mutex.cpp#L87 mtx_do_lock](https://github.com/microsoft/STL/blob/main/stl/src/mutex.cpp#L87)

1. `type` 플래그 값이 `_Mtx_plain` 일 때와 아닐 때 
2. `const xtime* target` 의 상태 값에 따른 락 처리

`_Mtx_plain` 플래그를 사용하는 곳을 찾을 수가 없어 첫 번째 분기는 스킵하고 `else` 하위 분기 코드만 살펴보겠습니다.

```cpp
static int mtx_do_lock(_Mtx_t mtx, const xtime* target) {
    if ((mtx->type & ~_Mtx_recursive) == _Mtx_plain) {
        // ...
    } else {
        int res = WAIT_TIMEOUT;
        if (target == nullptr) {
            // ...
        } else if (target->sec < 0 || target->sec == 0 & target->nsec <= 0) {
            // ...
        } else {
            // ...
        }

        // ...
    }
```

#### `_Mtx_lock`에서 호출 했을 때

`_Mtx_lock`에서 호출 했을 때 처리되는 코드를 보면 다음과 같은데요

`mtx->thread_id` 값이 호출한 스레드와 다른지 비교 후 `lock` 함수를 호출해 이미 락을 소유한 스레드에서 다시 잠금을 시도 했을 때 잠금을 허용하지 않도록 처리되어 있습니다.

최초 호출이라면 `_Thrd_success`를 리턴해 `_Check_C_return` 함수에서 예외를 발생시키지 않을 것이고 같은 스레드에서 두 번째 호출 부터는 `_Thrd_busy`를 리턴하기 때문에 예외가 발생하게 됩니다.

```cpp
static int mtx_do_lock(_Mtx_t mtx, const xtime* target) {
    if ((mtx->type & ~_Mtx_recursive) == _Mtx_plain) {
        // ...
    } else {
        int res = WAIT_TIMEOUT;
        if (target == nullptr) {
            if (mtx->thread_id != static_cast<long>(GetCurrentThreadId())) {
                mtx->_get_cs()->lock();
            }

            res = WAIT_OBJECT_0;
        } else if (target->sec < 0 || target->sec == 0 & target->nsec <= 0) {
            // ...
        } else {
            // ...
        }

        if (res == WAIT_OBJECT_0 || res == WAIT_ABANDONED) {
            // 최초 호출이라면 `count`는 0 -> 1로 증가
            if (1 < ++mtx->count) {
                if ((mtx->type & _Mtx_recursive) != _Mtx_recursive) {
                    // `std::mutex`는 `_Mtx_recursive` 플래그가 세팅되지 않기 때문에 카운트 감소 후 `WAIT_TIMEOUT` 상태로 변경
                    --mtx->count;
                    res = WAIT_TIMEOUT;
                }
            } else {
                mtx->thread_id = static_cast<long>(GetCurrentThreadId());
            }
        }

        switch (res) {
        case WAIT_OBJECT_0:
        case WAIT_ABANDONED:
            return _Thrd_success;

        case WAIT_TIMEOUT:
            if (target == nullptr || (target->sec == 0 && target->nsec == 0)) {
                return _Thrd_busy;
            } else {
                return _Thrd_timedout;
            }

        default:
            return _Thrd_error;
        }
    }
```


#### `_Mtx_trylock`에서 호출 했을 때

아래와 같이 값을 세팅 후 `mtx_do_lock` 함수를 호출하기 때문에 두번째 분기 코드를 실행하게 됩니다.

```cpp
xt.sec  = 0;
xt.nsec = 0;
```

마찬가지로 잠금을 소유중인 스레드를 체크합니다.

잠금 시도할 땐 내부적으로는 non-blocking 함수인 `TryAcquireSRWLockExclusive` API를 호출하는 `try_lock` 함수를 호출해주고 있습니다.

이후 처리는 `_Mtx_lock` 함수와 동일합니다.

```cpp
static int mtx_do_lock(_Mtx_t mtx, const xtime* target) {
    if ((mtx->type & ~_Mtx_recursive) == _Mtx_plain) {
        // ...
    } else {
        int res = WAIT_TIMEOUT;
        if (target == nullptr) {
            // ...
        } else if (target->sec < 0 || target->sec == 0 & target->nsec <= 0) {
            if (mtx->thread_id != static_cast<long>(GetCurrentThreadId())) {
                if (mtx->_get_cs()->try_lock()) {
                    res = WAIT_OBJECT_0;
                } else {
                    res = WAIT_TIMEOUT;
                }
            } else {
                res = WAIT_OBJECT_0;
            }

        } else {
            // ...
        }

        // ...
    }
```

#### `_Mtx_timedlock`에서 호출 했을 때

`std::mutex`에서 호출되는 함수는 아니지만 `mtx_do_lock` 함수 내부 로직 설명을 위해 추가 했습니다.

> 2022-11-10 저장소 코드 기준으로 사용하는 클래스가 확인되지 않습니다.  
과거에 `timed_mutex` 클래스에서 사용 했던게 아닐까 추측됩니다.

기존에 봤던 함수와는 다르게 `const xtime* xt`을 추가로 전달 받아 `mtx_do_lock`에 전달하고 있습니다.

```cpp
int _Mtx_timedlock(_Mtx_t mtx, const xtime* xt) {
    int res;

    _THREAD_ASSERT((mtx->type & _Mtx_timed) != 0, "timedlock not supported by mutex");
    res = mtx_do_lock(mtx, xt);
    return res == _Thrd_busy ? _Thrd_timedout : res;
}
```

전달된 시간 까지 `while` 루프를 돌면서 잠금 획득 시도를 하고 있습니다.

여기서도 같은 스레드 일 땐 잠금 획득 시도를 하지 않고 루프를 벗어나도록 처리되어있는 걸 볼 수 있습니다.

```cpp
static int mtx_do_lock(_Mtx_t mtx, const xtime* target) {
    if ((mtx->type & ~_Mtx_recursive) == _Mtx_plain) {
        // ...
    } else {
        int res = WAIT_TIMEOUT;
        if (target == nullptr) {
            // ...
        } else if (target->sec < 0 || target->sec == 0 & target->nsec <= 0) {
            // ...
        } else {
            xtime now;
            xtime_get(&now, TIME_UTC);
            while (now.sec < target->sec || now.sec == target->sec && now.nsec < target->nsec) { // time has not expired
                if (mtx->thread_id == static_cast<long>(GetCurrentThreadId())
                    || mtx->_get_cs()->try_lock_for(_Xtime_diff_to_millis2(target, &now))) { // stop waiting
                    res = WAIT_OBJECT_0;
                    break;
                } else {
                    res = WAIT_TIMEOUT;
                }

                xtime_get(&now, TIME_UTC);
            }
        }

        // ...
    }
```

### 왜 같은 스레드일 땐 잠금을 시도하지 않을까?

`std::mutex`에서 사용하는 `SRWLock` API가 동일한 스레드에서의 잠금을 허용하지 않기 때문인데요

```cpp
// winnt.h
typedef struct _RTL_SRWLOCK {
    PVOID Ptr;
} RTL_SRWLOCK, *PRTL_SRWLOCK;

// synchapi.h
typedef RTL_SRWLOCK SRWLOCK, *PSRWLOCK;
```

`SRWLOCK` 구조체를 보면 `PVOID Ptr` 하나만을 멤버로 가지고 있고 소유 중인 스레드 정보를 가지고 있지 않습니다.

따라서 자체적으로 동일한 스레드에서 잠금 처리에 대한 처리를 할 수 없는 구조입니다.

동일한 스레드에서 잠금을 두 번 이상 시도할 경우 교착 상태(데드락)이 발생하게 됩니다.

이와는 다르게 `CriticalSection` 같은 경우 동일한 스레드에서의 잠금을 시도를 허용하고 있는데요 `_RTL_CRITICAL_SECTION` 구조체를 보면 관련된 상태 값을 가지고 있는걸 알 수 있습니다.

```cpp
// winnt.h
typedef struct _RTL_CRITICAL_SECTION {
    PRTL_CRITICAL_SECTION_DEBUG DebugInfo;

    LONG LockCount;
    LONG RecursionCount;
    HANDLE OwningThread;
    HANDLE LockSemaphore;
    ULONG_PTR SpinCount;
} RTL_CRITICAL_SECTION, *PRTL_CRITICAL_SECTION;

// minwinbase.h
typedef RTL_CRITICAL_SECTION CRITICAL_SECTION;
```

## 정리

여기까지 `std::mutex` 내부 구현이 어떻게 이루어져있는지 살펴보았습니다.

확인하다보니 stl lock 관련 클래스 들의 특징과 `SRWLock` API 작동 방식도 궁금해졌는데 이건 다음에 시간이 된다면 정리를 해보려고 합니다.
