# Merge

우선 새로운 branch를 만든다.

![브랜치 포인터를 새로 만듦](https://git-scm.com/book/en/v2/images/basic-branching-2.png)

```bash
$ git branch iss53
$ git checkout iss53
```

위 두 명령어를 합치면 아래와 같이 된다.

```bash
$ git checkout -b iss53
Switched to a new branch "iss53"
```



`iss53` 브랜치를 Checkout 했기 때문에(즉, `HEAD` 는 `iss53` 브랜치를 가리킨다) 뭔가 일을 하고 커밋하면 `iss53` 브랜치가 앞으로 나아간다.

```bash
$ vim index.html
$ git commit -a -m 'added a new footer [issue 53]'
```

![진행 중인 `iss53` 브랜치](https://git-scm.com/book/en/v2/images/basic-branching-3.png)



## Merge의 기초

#### fast-forward는 쉬우니까 non fast-forward에 대해서만 설명한다.

![master와 별개로 진행하는 iss53 브랜치](https://git-scm.com/book/en/v2/images/basic-branching-6.png)

master도 이미 진행되고 있고, iss53이 별개로 진행되고 있다.

이 상황에서 53번 이슈를 다 구현하고 master branch에 merge하는 과정을 살펴보자.

```bash
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```

현재 브랜치가 가리키는 커밋이 Merge할 브랜치의 조상이 아니므로 Git은 'Fast-forward'로 merge 하지 않는다. 이 경우에는 Git은 각 브랜치가 가리키는 커밋 두개와 공통 조상 하나를 사용하여 3-way Merge를 한다.

![커밋 3개를 Merge](https://git-scm.com/book/en/v2/images/basic-merging-1.png)

단순히 브랜치 포인터를 최신 커밋으로 옮기는게 아니라 3-way merge의 결과를 별도의 커밋으로 만들고 나서 해당 브랜치가 그 커밋을 가리키도록 이동시킨다. 그래서 이런 커밋은 부모가 여러 개고 Merge 커밋이라고 부른다.

![Merge 커밋](https://git-scm.com/book/en/v2/images/basic-merging-2.png)



### Conflict Merge

IDE에서 제공하는 merge tool로 merge conflict 해결.

터미널로 conflict 상태를 보려면 `$git status` 로 확인 가능

