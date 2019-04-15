# Git Flow란?

Git Flow란 저장소를 보다 고수준으로 관리하기 위한 브랜칭 기법이다. 굳이 프로젝트를 더 복잡하게 관리할 필요가 있나 싶을 수도 있지만, 프로젝트의 규모가 점점 커지면, 많은 인원들이 코드에 동시에 접근하면서 필연적으로 문제가 발생하게 된다.

[배달의 x족에서도 쓴다더라!](http://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html)



![img](https://t1.daumcdn.net/cfile/tistory/99FE0A385AE809E81D)


[여기 설명이 너무 잘되어있는데](https://gist.github.com/ihoneymon/a28138ee5309c73e94f9)

[영어로 읽고 싶으면 여기](https://nvie.com/posts/a-successful-git-branching-model/)

## 요약하면

일단 stable한 origin/master 브랜치가 있고, 개발용 origin/develop 브랜치가 있다.

develop 브랜치에선 상시로 버그 수정된 커밋들이 추가됨

> 여기서 origin을 주요 원격 저장소라고 하자

보조 브랜치로 feature 브랜치, 출시용 release 브랜치, 긴급수정용 hotfix 브랜치가 있다.

새로운 기능 추가 작업이 있는 경우 feature 브랜치 생성. feature 브랜치는 언제나 develop 브랜치에서 시작됨.

기능 추가가 완료되었다면 feature 브랜치는 develop 브랜치로 merge됨.

devlop 브랜치에 이번 버전에 포함되는 모든 기능이 merge되었따면 QA를 위해 develop 브랜치에서 release 브랜치 생성. QA 진행시 발생한 버그들은 release 브랜치에 수정됨

QA 통과 후 release 브랜치를 master와  develop 브랜치로 merge 함.

