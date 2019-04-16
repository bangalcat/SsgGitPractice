# Git 연습용 프로젝트

### Git을 협업에 제대로 활용하기 위한 연습

Git은 적절히 활용하면 svn보다 더 협업을 잘 도와줄 수 있는 간편한 도구다. 하지만 막상 써보려고 하면 어려운 기능은 기피하고 결국 svn과 똑같이 쓰게 된다. 따라서 Git도 공부가 필요하다.

특히 branch를 관리하는 것은 생각보다 복잡하다. 본 repo에서는 commit, merge, rebase 등 git의 여러 기능을 써보고 어떤식으로 repository를 관리하는지 연습해볼 것이다.

또한 여러 Git의 용어들과 Git-Flow를 설명하고 Git-Flow를 연습해볼 것이다.

1. [Git 기본 개념](git-study/Git%20개념%20알아보기.md)

2. [Git Flow란](git-study/Git%20Flow란.md)

## 실습 방법 

#### 일단 Remote 저장소 만들지 않고 하는 방법

1. `origin/develop` branch를 checkout한다.

2. 새로운 feature branch를 만든다.

   ###### feature branch는 {username}-feature- {기능} 식으로 만든다.

   ex) `donghyun-feature-print`
```console
$ git checkout -b dh-feature origin/develop    
```

   > 당분간은 그냥 자기 이니셜만 붙여도 된다.

3. 작업 수행 후 커밋한다. *squash에 대해선 나중에 추가할 예정*

4. 작업한 feature branch에 develop 브랜치를 merge into 하거나 반대로 feature를 develop 브랜치에 rebase onto 한다. (rebase와 merge의 차이를 비교해볼 수 있다.)

> merge 하는 경우
>```console
>$ git checkout origin/develop
>$ git merge dh-feature
>```

5. feature branch를 develop branch에 merge하는 pull request를 생성한다.

#### 프로젝트 관리자는 다음 버전 기능 개발이 완료되면 `release` 브랜치 생성
1. `release` 브랜치에서 QA 진행

2. 버그 발생시 `develop` 브랜치에서 버그 수정후 다시 merge

3. 기능 개발과 QA가 완료되면 `master`와 `develop`브랜치에서 `release` 브랜치를 merge



#### 급하게 수정해야 될거 같은 부분이 있을 때
1. `hotfix` branch를 새로 checkout한다. 없으면 만들것
```console
$ git checkout -b hotfix origin/master
```
2. 수정 후 commit
3. merge request
```console
$ git checkout master
$ git merge --no-ff hotfix
```
4. branch delete
```console
$ git branch -d hotfix
```


## FAQ

> #### 그래서 IDE로 어떻게 하죠?
> 이클립스나 인텔리제이는 git이 gui로 제공되지만, git console 혹은 terminal도 제공하고있습니다.
> IDE로 하는 방법도 나중에 정리하긴 할거지만 기초를 익히는 만큼 되도록 명령어로 해주세요. 
>
> 예를 들어 pull 하는건 checkout하고 merge를 합쳐놓은 기능이라 할 수 있습니다.
>
> intellij에서는 git console에서 gui로 입력된 git 명령어를 커맨드로 바꿔서 보여주고 있습니다.
