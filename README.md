# hdmun.github.io

저의 기술 블로그 저장소입니다.

지킬 `hydejack` 테마를 사용했습니다.

https://github.com/hydecorp/hydejack


## Requirments

- ruby 3.0.2 


## Jekyll 서버 실행

jekyll 서버를 실행하여 깃허브 페이지에 등록하기 전에 로컬 PC에서 포스팅할 글이 어떻게 보여지는지 확인할 수 있습니다.

먼저, 다음 명령어를 실행하여 `Gemfile`에 명시된 패키지를 설치합니다.

```
bundle install
```

아래 명령어로 jekyll 서버를 실행합니다. `_drafts` 폴더의 글을 노출시키고싶지 않다면 `--drafts` 옵션은 빼고 실행하시면 됩니다.

```
bundle exec jekyll serve --drafts
```

기본 포트는 `4000`으로 세팅되어 있습니다. 브라우저에서 아래 주소로 접속하시면됩니다.

[http://localhost:4000](http://localhost:4000)


## _drafts 폴더

저장소에 반영해도 깃허브 페이지에 노출되지 않습니다.

저는 작성 중인 글을 보관하는 용도로 사용하고 있습니다.
