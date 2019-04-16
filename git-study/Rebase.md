# Rebase

## Rebase 활용

Rebase는 단순히 브랜치를 합치는 것만 아니라 다른 용도로도 사용가능. 

```console
$ git rebase --onto master server clinet
```
이 명령은 `master` 브랜치부터 `server` 브랜치와 `client` 브랜치의 공통 조상까지의 커밋을 `client` 브랜치에서 없애고 싶을 때 사용.

이제 master branch로 돌아가서 fast-forward 시킬 수 있다.
```console
$ git checkout master
$ git merge client
```
server 브랜치의 일이 다 끝나면 `git rebase <basebranch> <topicbranch>` 라는 명령으로 Checkout 하지 않고 바로 `server` 브랜치를 `master` 브랜치로 rebase 할 수 있다. 이 명령은 토픽(server) 브랜치를 checkout하고 베이스(master) 브랜치에 rebase 한다.
```console
$ git rebase master server
```