---
title: "Github Actions 를 통한 테스트 자동화 구축"
description: "팀 프로젝트를 진행하면서 Github Actions를 통해 테스트 자동화 환경을 구축한 기록"
date: 2024-03-01
update: 2024-03-01
tags:
  - ci/cd
  - githubactions
series: "ci/cd"
---


팀 프로젝트를 진행하면서 Github Actions를 통해 테스트 자동화 환경을 구축한 기록

[코드](https://github.com/jinkshower/please-praise) 보러가기

## 학습 계기

팀프로젝트를 진행하면서 팀원들끼리 테스트 코드를 작성했지만, 이를 PR에서 확인할 수 있는 방법이 없었다. 

물론 팀원들을 믿고 있지만,  나조차도 급할때는 테스트를 돌리는 걸 까먹고 push를 한 기억도 있기 때문이다.. 

Github Actions를 통하면 PR마다 혹은 push마다 테스트를 자동화할 수 있는 workflow를 설정할 수 있다.

## 테스트 자동화의 필요성

`제 로컬에서는 잘 돌아가는데요..?`

1. 환경 일관성 보장

각자의 로컬이 아닌 독립된 (Github Actions의 경우 Runner 서버) 환경에서 테스트가 실행되기 때문에 환경 관련 문제를 사전에 감지 할 수 있다.
즉, 배포 환경에 맞는 빌드/테스트 환경을 구축하는데 큰 도움이 된다

2. 커밋 이후 안전성 확인 가능

위에서도 말했지만 PR에 `테스트 다 통과해요~` 라는 말로는 PR에 올라온 코드는 믿을 수 없다. 독립된 환경에서 모든 테스트가 통과하는 코드라면, 안심하고 Approve를 누를 수 있게 도와준다. 

3. 테스트 실패를 팀 모두가 알 수 있음

테스트 자동화 환경이 구축된 협업에서는 참여하는 모두가 테스트 실패시 에러로그를 공유할 수 있기 때문에 디버깅을 모두가 빠르게 할 수 있고, 공통적인 문제점을 캐치하기 쉬워진다. 

## Github Workflow 파일 작성

.github/workflows/{name}.yml로 workflow 설정 파일을 추가해줬다.

```yml
name: PR Test  
  
on:  
  pull_request:  
    branches: [ dev ]  
  
jobs:  
  test:  
    runs-on: ubuntu-latest  
    steps:  
      - name: Set up JDK 17  
        uses: actions/setup-java@v1  
        with:  
          java-version: 17    
  
      - name: Grant execute permission for gradlew  
        run: chmod +x gradlew  
  
      - name: Test with Gradle  
        run: ./gradlew --info test  
  
      - name: Publish Unit Test Results  
        uses: EnricoMi/publish-unit-test-result-action@v1  
        if: ${{ always() }}  
        with:  
          files: build/test-results/**/*.xml
```

한 부분씩 살펴보도록 하자.

`Set up JDK 17`

Runner 서버에 JDK를 설치한다. 

`Grant execute permission for gradlew`

명령어를 통해 gradlew 스크립트에 대한 실행권한을 부여한다.

`Test with Gradle`

실행 권한을 받아 테스트를 실행한다

`Publish Unit Test Results`

다른 사람이 만든 workflow설정도 쓸 수 있다. [링크](https://github.com/EnricoMi/publish-unit-test-result-action)
테스트 결과를 읽기 쉬운 로그로 게시해준다.

하지만... 
![Pasted image 20240219161225](https://github.com/jinkshower/jinkshower.github.io/assets/135244018/1f76a31c-2e77-4ce2-bf41-818fdf1fda07)

## Fail to load ApplicationContext

앞에서도 말했듯이 Github Actions는 별개의 Runner서버에서 실행되기 때문에 로컬에서만 적용되는 환경설정은 오류가 발생한다

나의 경우 설정 파일에 JWT 시크릿 키나 관리자 토큰 값이 있었기 때문에 기본적으로 .gitignore에 설정파일을 추가하고 팀원들도 마찬가지로 각자 로컬환경에서 설정파일을 작성했기 때문에 해당 값을 가져다 쓰는 빈을 생성하는데 문제가 발생했다.

가장 빠른 해결 방법은 물론 설정파일을 깃에 업로드하는 거지만, AWS 키와 같은 절대 공개되어서는 안
되는 정보는 깃에 올릴 수 없다.

이에 대한 해결방법으로 
Github Secrets에 모든 설정을 등록하고 workflow에서 직접 설정파일을 작성하는 job을 추가하는 방법을 찾아봤지만 value를 암호화해야하고, 수정할때마다 secret을 삭제, 추가하는 일이 너무 번거로워 보였다.

## Git Submodule로 설정파일 관리하기

Git Submodule를 이용하면 리포지토리 안에 다른 리포지토리를 분리해서 넣어서 사용할 수 있다고 한다.

별도의 private repository에 설정에 필요한 파일을 두고 메인 리포지토리에서 이를 연동해 사용한다면 팀원들과 설정을 공유하기도 쉽고 이를 workflow에서 사용할 수도 있다.

private repository에 설정파일을 업로드 한 후 메인 리포지토리의 깃에서 
`git submodule add {private-repository-url} config`
를 입력해 서브모듈 리포지토리를 등록해준다.

>[config]
>
>스프링은 설정파일을 가장 먼저 `src/main/resources`에서 찾지만 해당 디렉토리에 설정파일이 없을 경우 `config`디렉토리를 찾아서 적용하기 때문이다. 
>
>나는 설정파일은 깃에 업로드하고 있지 않기 때문에 서브모듈에서 가져올 설정파일을 `config` 경로에 연동했다. 

>[서브모듈 최신화]
>
>서브 모듈은 커밋 후 푸시를 해도 메인 리포지토리에서 자동으로 추적하여 업데이트 하지 않는다.
>
>메인 리포지토리에서 서브모듈의 commit hash값만 참고하는 방식이기 때문에 서브모듈을 업데이트 했다면 `git submodule update --remote`로 최신화 해줘야 한다

.gitmodules 파일에 서브모듈 경로가 잘 적용된 것을 확인했다면 workflow에 해당 서브모듈을 연동함을 명시해 줘야 한다. 
secrets에 다른 리포지토리 접근을 허용하는 access token을 추가한 후

workflow 파일에 해당 작업을 추가해줬다.
```yml
- name: Checkout  
  uses: actions/checkout@v2  
  with:  
    token: ${{ secrets.ACTION_TOKEN }}  
    submodules: true
```

## 마치며

![Pasted image 20240301233610](https://github.com/jinkshower/jinkshower.github.io/assets/135244018/701cd629-2d9a-4185-8157-803e3696b32c)

이렇게 여러 시간의 시도를 통해 PR마다 테스트를 자동화해주었다.

테스트 실패시 PR에 실패한 라인에 코멘트를 달아주는 workflow, 빌드/테스트 실패시 slack에 알림을 보내는 workflow도 있으니 각자의 협업 방식에 맞는 workflow를 추가해주면 된다! 

*틀린 부분이나 부족한 부분에 대한 피드백은 언제나 환영합니다*

