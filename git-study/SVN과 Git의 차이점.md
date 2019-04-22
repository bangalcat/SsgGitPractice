# SVN과 Git의 차이점

##### SVN은 중앙 집중식 버전 관리 시스템

![img](https://t1.daumcdn.net/cfile/tistory/27096736594B1C5B0A)

CVCS는 중앙 서버가 잘못 되면 모든 것이 잘못된다.

이러한 문제를 해결하기 위해 개발된 버전관리 시스템이

##### 분산 버전 관리 시스템

대표적으로 Git, Mecurial, Bazaar, Darcs 등이 있다.

DVCS에서는 클라이언트가 파일들의 마지막 스냅샷을 가져오는 대신 저장소(Repository)를 통째로 복제. 체크아웃시마다 전체 백업이 일어나는 셈이다.

![img](https://t1.daumcdn.net/cfile/tistory/213CC73C594B1FDC11)



svn으로 작업할 때에는 소스를 중앙 저장소에 commit 하기 전에 대부분의 기능을 완성해놓고 commit하는 경우가 많다.