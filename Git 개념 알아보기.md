# Git 개념 알아보기

쉬운 개념들이지만 잠깐 짚고 넘어가겠다.

## Repository

말 그대로 파일 등이 저장되는 저장소, 즉 프로젝트 폴더. 저장소의 종류는 다음과 같다

- Remote Repository (원격 저장소) : 원격 서버에 저장된 저장소, 여러 사람이 공유

- Local Repository (개인 저장소) : 개발자의 PC에 저장된 저장소

## Commit

프로젝트의 변경 이력을 말한다.  쉬우니까 넘어가고

## Stage

우선 스테이지에 대해 설명하기 전 Index에 대한 개념을 알아야 한다. Index는 Commit을 통해 변경사항들이 반영되기 전 해당 변경사항의 이력들이 저장되는 공간이다. 따라서 우리가 특정 파일이나 코드를 변경 시 해당 이력은 Index에 기록된다. 이 때 기록되는 행위를 Stage 또는 Staging이라 한다. 변경사항들 중에서 원하는 변경사항만 Stage하고 원하지 않는 변경 사항은 unstage한 뒤 commit을 진행하면 된다.

커밋이 여러 변경사항들의 집합이라 하면, 스테이지는 그 변경사항 하나하나를 반영할지를 결정한다고 보면 된다.

## Branch

여러 명이 같은 코드를 공유하면 협업하는 상황을 생각해보자. 각 개발자들은 여러 커밋을 만들며 코드를 발전시키는데, 이 때 누가 어떤 커밋을 추가했는지 구분이 가능해야 한다. 이 때 사용되는것이 바로 브랜치이다.

브랜치는 특정 커밋으로부터 분기되는 포인터를 말하는 것으로, 각 개발자들이 개발을 진행하고 있는 환경 또는 흐름을 말한다. 새로운 브랜치가 생성되더라도 메인 브랜치는 그대로 남아있다.

## Checkout

현재 위치한 커밋에서 다른 커밋으로 이동하는 것을 checkout이라 한다. checkout을 통해 현재 커밋에서 같은 브랜치 내 다른 커밋으로 이동하거나, 다른 브랜치 내 커밋으로 이동할 수 있다. 결론적으로 체크아웃으로 인해 이전 시점의 버전으로 돌아갈 수 있고, 다른 사람의 브랜치로 전환해 다른 개발자들의 코드 진행 상황을 확인해 볼 수도 있다.

## Merge

나뉘어진 브랜치를 다시 하나의 브랜치로 합치는 것을 말한다.

[3.2 Git Branch - Branch와 Merge의 기초](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-Merge-%EC%9D%98-%EA%B8%B0%EC%B4%88)

[7.8 Git 도구 - 고급 Merge](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-%EA%B3%A0%EA%B8%89-Merge)

중요한 개념이니 추후 자료를 좀더 정리해서 설명하겠다.

##### fast-forward

- 쉽게말해 충돌(conflict)이 일어나지 않는 merge

##### non fast-forward

- 반대로 충돌이 일어나는 merge. merge하면서 새로운 커밋 생성

## Rebase

[3.6 Git Branch - Rebase 하기](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)

## Stashing과 Cleaning

[7.3 Git 도구 - Stashing과 Cleaning](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Stashing%EA%B3%BC-Cleaning#_git_stashing)





## 참고

[시작하기 - 버전 관리란?](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%EB%9E%80%3F)

[Git에 대해 알아보자 1. Git의 개념과 Git Flow](https://cupjoo.tistory.com/6)