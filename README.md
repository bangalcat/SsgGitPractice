# Git 연습용

### Git을 협업에 제대로 활용하기 위한 연습

Git은 적절히 활용하면 svn보다 더 협업을 잘 도와줄 수 있는 간편한 도구다. 하지만 막상 써보려고 하면 어려운 기능은 기피하고 결국 svn과 똑같이 쓰게 된다. 따라서 Git도 공부가 필요하다.

특히 branch를 관리하는 것은 생각보다 복잡하다. 본 repo에서는 commit, merge, rebase 등 git의 여러 기능을 써보고 어떤식으로 repository를 관리하는지 연습해볼 것이다.

또한 여러 Git의 용어들과 Git-Flow를 설명하고 Git-Flow를 연습해볼 것이다.

1. [Git 기본 개념](https://github.com/Semaj2010/SsgGitPractice/blob/master/Git%20%EA%B0%9C%EB%85%90%20%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0.md)

2. [Git Flow란](https://github.com/Semaj2010/SsgGitPractice/blob/master/Git%20Flow%EB%9E%80.md)

## 실습 방법 

#### 일단 Remote 저장소 만들지 않고 하는 방법

1. `develop` branch를 checkout한다.

2. 새로운 feature branch를 만든다.

   ###### feature branch는 {username}/feature- {기능} 식으로 만든다.

   ex) `donghyun/feature-print`

   > 당분간은 그냥 자기 이니셜만 붙여도 된다.

3. 작업 수행 후 커밋한다. *squash에 대해선 나중에 추가할 예정*

4. 작업한 feature branch에 develop 브랜치를 merge into 하거나 반대로 feature를 develop 브랜치에 rebase onto 한다. (rebase와 merge의 차이를 비교해볼 수 있다.)

5. feature branch를 develop branch에 merge하는 pull request를 생성한다.
