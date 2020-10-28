

~~~cpp

//////////////////////////////////////////////////////////////////////////
// warning level 2
//////////////////////////////////////////////////////////////////////////

#pragma warning(disable:4099)  // 구조체로 선언 된 개체가 클래스로 정의 되거나 클래스로 선언 된 개체가 구조체로 정의 됩니다.
#pragma warning(disable:4146)  // 부호 없는 형식에 단항 빼기 연산자가 적용
#pragma warning(disable:4244)  // 부동 소수점 형식이 정수 형식으로 변환 되었습니다. 데이터 손실이 발생했을 수 있습니다.
#pragma warning(disable:4250)  // 두 개 이상의 멤버가 동일한 이름을 갖습니 되었습니다.
#pragma warning(disable:4251)
#pragma warning(disable:4302)  // 컴파일러가 더 큰 형식에서 더 작은 형식으로의 변환을 발견 했습니다. 정보가 손실 될 수 있습니다.
#pragma warning(disable:4305)  // 'context': 'type1'에서 'type2' (으)로 잘림
#pragma warning(disable:4309)  // 상수 값이 잘렸습니다.
#pragma warning(disable:4311)  // 이 경고는 64비트 포인터 잘림 문제를 검색합니다. 예를 들어 코드가 64비트 아키텍처에 대해 컴파일된 경우 포인터(64비트)의 값을 int(32비트)에 할당하면 값이 잘립니다.
#pragma warning(disable:4312)  // 이 경고는 32비트 값을 64비트 포인터 형식에 할당하려는 식도를 감지합니다. 예를 들어 32비트 int 또는 long을 64비트 포인터로 캐스팅합니다.


//////////////////////////////////////////////////////////////////////////
// warning level 3
//////////////////////////////////////////////////////////////////////////

#pragma warning(disable:4018)  // ' expression ': signed/unsigned가 일치 하지 않습니다.
#pragma warning(disable:4060)  // switch 문에 ' case ' 또는 ' default ' 레이블이 없습니다.
#pragma warning(disable:4065)  // switch 문에 ' default ' 및 ' case ' 레이블이 포함 되어 있지 않습니다.
#pragma warning(disable:4101)  // ' identifier ': 참조 되지 않은 지역 변수입니다.
#pragma warning(disable:4197)  // ' type ': 캐스트의 최상위 volatile은 무시 됩니다.

#pragma warning(disable:4240)  // 비표준 확장이 사용 됨: 'classname'에 대 한 액세스가 이제 'access_specifier1' (으)로 정의 되었습니다. 이전에 'access_specifier2' (으)로 정의 되었습니다.
#pragma warning(disable:4244)  // 'conversion': 'type1'에서 'type2'(으)로 변환하면서 데이터가 손실될 수 있습니다.
#pragma warning(disable:4244)  // 비표준 확장이 사용 됨: 'identifier' 정식 매개 변수가 이전에 형식으로 정의 되었습니다.
#pragma warning(disable:4267)  // 'var' : 'size_t'에서 'type'으로 변환하면서 데이터가 손실될 수 있습니다.
#pragma warning(disable:4334)  // 'shift_operator': 32 비트 시프트 결과가 64 비트로 암시적으로 변환 됩니다 (64 비트 이동 의도?).
#pragma warning(disable:4390)  // '; ': 제어 되는 빈 문이 있습니다. 이것이 의도 입니까?
#pragma warning(disable:4554)  // 'operator': 연산자 우선 순위를 검사 하 여 가능한 오류를 확인 하십시오. 괄호를 사용 하 여 우선 순위를 명확 하 게 설명
#pragma warning(disable:4724)  // 0의 나머지 연산이 발생할 수 있습니다.
#pragma warning(disable:4800)  // 'Type'에서 bool로의 암시적 변환입니다. 가능한 정보 손실
#pragma warning(disable:4995)  // #pragma deprecated(fucntionName) 표시된 함수들
#pragma warning(disable:4996)  /// __declspec (사용 되지 않음) 
#pragma warning(disable:5030)  // 'attribute name' 특성을 인식할 수 없습니다.


/// #import 관련 경고
#pragma warning(disable:4192)  // ' library ' 형식 라이브러리를 가져오는 동안 ' 이름 '을 자동으로 제외 합니다.
#pragma warning(disable:4244)  // ' library ' 형식 라이브러리를 가져오는 동안 ' 이름 '을 자동으로 제외 합니다.
#pragma warning(disable:4278)  // 'identifier': 형식 라이브러리 'tlb'의 식별자가 이미 매크로입니다. ' rename ' 한정자 사용


//////////////////////////////////////////////////////////////////////////
// warning level 4
//////////////////////////////////////////////////////////////////////////

#pragma warning(disable:4063)  // case ' identifier '는 열거형 ' 열거 '의 switch에 사용할 수 없는 값입니다.

#pragma warning(disable:4100)  // 'identifier' : 참조되지 않은 정식 매개 변수
#pragma warning(disable:4127)  // 조건식이 상수입니다.
#pragma warning(disable:4130)  // ' operator ': 문자열 상수의 주소에서 논리 연산을 수행 했습니다.
#pragma warning(disable:4189)  // 'identifier': 지역 변수가 초기화되었으나 참조되지 않았습니다.

#pragma warning(disable:4238)  // 비표준 확장이 사용 됨: 클래스 rvalue가 lvalue로 사용 되었습니다.
#pragma warning(disable:4239)  // 비표준 확장이 사용 됨: ' token ': ' type '에서 ' type ' (으)로 변환 합니다.
#pragma warning(disable:4245)  // ' conversion ': ' type1 '에서 ' type2 ' (으)로의 변환, 부호 있는/부호 없는 불일치

#pragma warning(disable:4310)  // 캐스트할 때 상수 값이 잘립니다.
#pragma warning(disable:4315)  // 'classname': 'member' 멤버의 ' this ' 포인터는 생성자가 예상 하는 '맞춤'에 맞지 않을 수 있습니다.
#pragma warning(disable:4324)  // 'structname': __declspec (align ())으로 인해 구조체가 패딩 되었습니다.
#pragma warning(disable:4366)  // 단항 'operator' 연산자의 결과가 맞춰지지 않을 수 있습니다.
#pragma warning(disable:4389)  // ' operator ': signed/unsigned가 일치 하지 않습니다.

#pragma warning(disable:4456)  // 선언의 '식별자' 이전 로컬 선언을 숨깁니다.
#pragma warning(disable:4457)  // 선언의 '식별자' 숨깁니다 함수 매개 변수
#pragma warning(disable:4458)  // 선언의 '식별자' 클래스 멤버를 숨깁니다
#pragma warning(disable:4459)  // 'identifier' 선언이 전역 선언을 숨깁니다.

#pragma warning(disable:4505)  // 'function': 참조 되지 않은 지역 함수를 제거 했습니다.
#pragma warning(disable:4592)  // 'function': ' constexpr ' 호출을 실행 하지 못했습니다. 런타임에 함수가 호출 됩니다.
#pragma warning(disable:4611)  // ' function '과 C++ 개체 소멸 사이의 상호 작용이 이식 가능 하지 않습니다.

#pragma warning(disable:4701)  // 잠재적으로 초기화 되지 않은 ' name ' 지역 변수를 사용 합니다.
#pragma warning(disable:4702)  // 접근할 수 없는 코드
#pragma warning(disable:4703)  // 잠재적으로 초기화 되지 않은 로컬 포인터 변수 '% s '이 (가) 사용 되었습니다.
#pragma warning(disable:4706)  // 조건식 내에 할당
~~~