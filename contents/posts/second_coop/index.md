---
title: "두번째 협업 회고"
description: "프로젝트 회고"
date: 2024-02-15
update: 2024-02-15
tags:
  - cooperation
  - retrospective
series: "retrospective"
---

프로젝트 기간(2024.02.07 - 2024.02.15)

[완성 레포지토리](https://github.com/jinkshower/OTT_Suggestion)

해당 프로젝트를 진행하며 느낀 점에 대한 기록 

## 지켜야 할 것은 문서로 남기기

[첫 협업](https://jinkshower.github.io/first_team_assignment/) 이후 `문서화`의 중요성을 깨달았고 이번 프로젝트에서는 '이걸 지켜주세요' 라고 말하는게 아니라 지켜야할 것은 문서로 작성하기로 했다. 

코드 컨벤션을 IDE에 적용, 설정하기 위한 레퍼런스를 공유했고, 깃 컨벤션과 브랜치 전략을 프로젝트 초기단계에 모두 같이 모여 정했다. 

![Pasted image 20240215205740](https://github.com/jinkshower/jinkshower.github.io/assets/135244018/327a4bad-38ce-4946-b347-ef77829241d1)

JDK 버전,  gitignore, dependency 설정 등 각자의 로컬환경에서 충돌이 일어날 수 있는 부분을 모두 통일했다.

## 설계와 공통코드 

서로 다른 코드 스타일을 가진 개발자들과 같이 '잘' 코드를 작성하려면 공통된 보일러플레이트가 있어야 한다고 생각했다. 

공통코드를 같이 프로그래밍하고, 그 것을 지켜야할 `규칙`으로 만든다면 push할 때 조금 덜 두려워질 것이라고 생각했다. 

IntelliJ의 CodeWithMe를 사용하여 5명이 동시에 의견을 주고 받으며 프로그래밍을 진행했고, 개인적으로 설계에서 중요하다고 생각하는 패키지구조와 도메인 클래스명들을 몹 프로그래밍으로 구현했다.

`문서화` 와 `공통코드`를 통해 프로젝트 전반적으로 Merge시 충돌이 적었으며 이는 곧 생산성의 증진으로 이어졌다.

## Approve를 누른다는 건

주어진 프로젝트 기간이 길지 않았기 때문에 페어프로그래밍이나 몹프로그래밍으로 구현한 부분은 Approve를 급하게 누르고 진행하려는 경향이 있었다. 

같이 작성한 코드니까 문제가 생기지 않겠다고 생각했지만, 프로젝트 중반이 되는 지점에서 파일 개수가 많아지고 새로운 기능을 구현할수록 이전 코드와 일관성이 떨어지거나, 끼워 맞추는 식의 코드가 발생했다.

해당 문제가 발생할때마다 회의를 통해 공통된 코드에 대한 인식을 리마인드하는 시간을 가져서 문제를 해결했지만 같이 프로그래밍을 한 것이라도, 세세한 코드 리뷰나 PR 기록을 남기지 않으면 결국 내가 그 코드를 마주해야 함을 깨달았다. 

Approve를 누른다는 건, 이 코드를 내가 작성한 것으로 생각하겠다는 허락이라는 생각이 들었다. 

## 협업

이번 프로젝트에서 열정적이고 좋은 팀원들을 만나게 되어 혼자서 했다면 훨씬 오래 걸릴 구현들을 빠르게 진행할 수 있었지만 만약 그렇지 않았다면? 내가 함께 일하게 될 팀원들은 내 이상과 많이 다를 수도 있다. 

결국 협업을 통해 더 좋은 결과를 내고 싶다면, 내가 이러한 부분을 문제로 생각한다면, 그 것을 문서로 남길 수 밖에 없다는 것을 다시금 새기게 되었다. 

또한 내 자신의 코드스타일에 좀 더 유연함을 가져야 함을 스스로 느끼게 되었다.  내가 개인적으로 맞다고 생각하는 방향은 다른 사람이 보기에 이해되지 않는 낯선 방향일수도 있다. 

그리고 이를 내가 설득할 수 없다면, 그건 내가 틀린 것일 수도 있지 않을까? 

