버전관리란 파일변화를 시간에 따라 기록하고, 나중에 언제든지 그 시간으로 파일을 되돌릴 수 있는 시스템 

## 1. 버전관리의 역사
로컬 버전 관리 -> 중앙 집중 버전 관리 -> 분산 버전 관리

### 로컬 버전 관리
개인 컴퓨터에 버전에 대한 데이터베이스 + 데이터베이스에서 특정 시점의 파일 버전을 불러온다. 
![](https://git-scm.com/book/en/v2/images/local.png)

### 중앙 집중 버전관리
여러 사람들과 버전관리를 사용하가 위한 시스템. 버전 데이터 베이스가 중앙 서버에 존재하고, 각 유저는 중앙 데이터베이스에서 특정 버전을 불러와서 작업. 로컬에는 특정 버전의 파일만 갖게 됨.
![](https://git-scm.com/book/en/v2/images/centralized.png)

### 분산 버전 관리
 버전 데이터베이스가 중앙 서버, 각 로컬에 존재하도록 구현. 서버와 연결되어있지 않더라도, 언제든 다른 버전으로 체크아웃할 수 있고, 중앙 서버의 데이터가 손실되더라도 언제든 복구할 수 있다.
![](https://git-scm.com/book/en/v2/images/distributed.png)

[Git - 버전 관리란?](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%EB%9E%80%3F)

## 2. Git의 역사
### 이름의 기원
> Git - 멍청한 버전 관리기
> 
> git은 기분에 따라 무엇이나 될 수 있다.
> 
> 그냥 의미 없는 알파벳 3가지. 유닉스 명령어 중에 git이 없어서 정함. 발음 하기 좋음. get으로 하려다 오타남.
> 
> 멍청하고, 한심하고, 들떨어진 의미. 아무거나 고르면 됨
> 
> 기분이 좋다면, Global Information Tracker
> 
> 기분이 좋지 않다면, Goddam idiotic truckload of sh*t (욕)

[Initial revision of “git”, the information manager from hell · git/git@e83c516 · GitHub](https://github.com/git/git/commit/e83c5163316f89bfbde7d9ab23ca2e25604af290#diff-c47c7c7383225ab55ff591cb59c41e6bR2-R13)

### 리눅스 토발즈가 git을 개발하게 된 이유
![](https://www.bitkeeper.org/man/BitKeeper_SN_SVC_Blue.png)

초기 분산 버전 관리 상용 도구 빗키퍼

> *빗키퍼 제작자 맥보이* : 오픈소스 개발자들! 우리 버전 관리 서비스 사용해보셈 짱짱임. 그치만 우린 상업용이니까 빗키퍼 클라이언트를 사용해야만해
>
> *리눅스 토발즈* : 빗키퍼는 상업용이지만, 버전 관리툴을 사용할 수 밖에 없으니 사용합시다!
>
> *리눅스 개발자 중 한명인 트리젤* : 오픈소스 툴로 사용하려먼, 빗키퍼의 클라이언트로만 사용하는 것은 불편하겠는데? 쓰기 편하도록 바꿔보겠음! (빗키퍼의 프로토콜을 참조하여 오픈소스 툴로 사용하기 좋도록 변경함)
>
> *맥보이* : 빗키퍼를 리버스 엔지니어링해?? 무료 사용 취소할꺼야! 
> 
> *트리젤* : 리버스 엔지니어링 아님! 빗키퍼 클라이언트의 네트워크에서 사용하는 값을 가져와서 편하게 사용하는 것 뿐임!
> 
> *토발즈* : 자자 그만싸우고, 내가 새로 개발하지 뭐!

[Linus Torvalds’ BitKeeper blunder | InfoWorld](https://www.infoworld.com/article/2670360/linus-torvalds--bitkeeper-blunder.html)) 을 각색했습니다.

### 빗키퍼를 사용하면서 리눅스 토발즈가 세운 깃의 목표

* 빠른 속도
* 단순한 구조
* 비선형적인 개발(수천 개의 동시 다발적인 브랜치)
* 완벽한 분산
* Linux 커널 같은 대형 프로젝트에도 유용할 것(속도나 데이터 크기 면에서)

[Git - 짧게 보는 Git의 역사](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EC%A7%A7%EA%B2%8C-%EB%B3%B4%EB%8A%94-Git%EC%9D%98-%EC%97%AD%EC%82%AC)

## 3. Git의 특징
Git은 기존 버전 관리 시스템과 미묘하게 다른 부분들이 있다. (인터페이스는 비슷할 지 몰라도 정보를 취급하는 방식이 다름)

### 1. 차이가 아니라 스냅샷
버전 관리 시스템들 대부분이 파일들의 목록을 관리하고, 파일들의 변화를 시간순으로 관리한다. (이를 델타 기반 버전 관리라고 함 )
![](https://git-scm.com/book/en/v2/images/deltas.png)

위의 그림 처럼, 각 버전마다 어떤 파일의 어떤부분이 변경되었는 가를 기록

Git은 위의 델타 기반 버전이 아니라 스냅샷으로 저장함. 스냅샷은 파일 그자체로 생각하면 됨. Git은 프로젝트의 상태를 저장(커밋)할 때 파일이 존재하는 순간을 중여하게 여김. 파일이 달라지지 않으면 저장하지 않고, 단지 이전 상태에 대한 링크를 저장함. 아래의 그림 처럼 각 버전에는 변경값이 아니라 파일의 링크를 저장.

![](https://git-scm.com/book/en/v2/images/snapshots.png)


### 2. 거의 대부분의 명령어를 로컬에서 실행
모든 버전의 파일들이 로컬에 있고, 이 또한 로컬에서 관리하기 때문에 명령어를 로컬에서만 실행할 수 있음. 네트워크에 접속하고 있지 않아도 버전관리를 이용할 수 있다.

### 3. 무결성 (데이터의 정보가 변경되거나 오염되지 않도록 하는 원칙)
데이터를 저장하기전 체크섬을 구하고, 이 체크섬을 이용해 데이터를 관리함.

*체크섬*? git은 SHA-1 방법을 이용해 파일의 내용과, 디렉토리 구조를 이용해서 아래의 긴 문자열(체크섬, 커밋 해쉬)을 만든다.
 `24b9da6552252987aa493b52f8696cd6d3b00373` 
 
이 체크섬은 git을 이용해야만 파일이 어떤 상태인지, 어떤 데이터가 있는지를 확인 할 수 있다. 

*SHA-1 방법*? : 안전한 해시 알고리즘이라는 의미. 숫자는 1, 224, 256 등등이 있으며 1은 보통 낮은 단계의 해시 알고리즘

*해시 함수*? : 임의의 길이의 데이터를 고정된 길이의 데이터로 매핑하는 함수

*해시 알고리즘* ? : 해시를 만들어내는 알고리즘. 

SHA-1 방식은 분명 다른 데이터를 가지고 같은 해쉬 값을 만들 위험이 있지만, 확률이 아주 희박하다. 또한 주변의 다른 체크섬(커밋)을 참고하면 바뀐 파일과 데이터가 무엇이 었는지 확정할 수 있다. 

git은 체크섬을 이용하여 무결성을 보장한다.

### 4. 데이터를 추가할 뿐.
git으로 무엇을 하든 git의 database에 기록하게된다. 스냅샷을 커밋하고 나면 데이터를 잃어버리는 것이 더 어렵다. 커밋 삭제도 git에 기록으로 남기기 때문에 복원이 가능하다. 

### 5. git의 3가지 상태
*Committed* : 데이터가 local db에 안전하게 저장됨

*Modified* : 수정한 파일들. 아직 db에 저장하지 않음

*Staged* : 수정한 파일을 곧 커밋할 것이라 표시함

![](https://git-scm.com/book/en/v2/images/areas.png)
파일의 상태 변경 흐름
* Working Directory 에서 파일 수정 (modified)
* Staging Area에 파일을 stage해서 커밋할 스냅샷을 만듬(working -> staging, Staged)
* Staging Area에 있는 파일들을 커밋해서 git 디렉토리에 영구적으로 스냅샷 저장(staging -> git directory, commited)

[Git - Git 기초](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-Git-%EA%B8%B0%EC%B4%88)을 참고해서 작성했습니다.

## 4. Git 사용하기 
### git config : 최초 설정
Git config 명령어를 이용한다. --system, —global, —local 3가지 존재함. Git config 명령어는 git/gitconfig 파일을 수정하는 명령어임.
`.git/config (local)` `~/.gitconfig (global)` `/etc/gitconfig (system)`순으로 config 파일의 우선순위가 적용한다.

### git help <명령어>
git의 명려어에 어떤 옵션이 있고, 어떤 일을 할 수 있는지 설명.

## 5. Git의 기초
### 1. git 저장소를 만드는 방법 두가지
* git 을 사용하지 않는 디렉토리에 적용하기 git init
* 다른 어딘가에서 git 저장소를 클론하기 git clone

### 2. 수정하고 저장소에 저장하기
#### Working Directory에서 파일의 상태 두가지
* git이 파일을 관리하고 있는 상태인 Tracked, 
* 관리하고 있지 않은 Untracked

#### Tracked 파일에는 3가지 상태가 존재
* 수정하지 않은 Unmodified
* 수정됨 Modified
* 커밋 준비 Staged

#### Git 에서의 파일의 라이프 사이클
![](https://git-scm.com/book/en/v2/images/lifecycle.png)

### 3. 기본 명령어를 배워 봅시다.
[Git - 수정하고 저장소에 저장하기](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%88%98%EC%A0%95%ED%95%98%EA%B3%A0-%EC%A0%80%EC%9E%A5%EC%86%8C%EC%97%90-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)

#### `git status`
파일의 상태는 git status 명령어로 확인 가능하다. 

#### `git add` (untracked or modified -> staged)
프로젝트에 파일을 추가하는게 아니라, 새로 만들 커밋에 추가한다는 의미다. 
Git add 후에 파일을 또 수정하면 해당 파일은 staged, modifed를 둘다 갖게 된다. 

#### gitignore 파일
git으로 관리하지 않는 파일을 명시함. 디렉토리 별로 존재할 수 있고, 하위 디렉토리까지 영향을 준다. linux에는 206여개의 gitignore가 있다. 유명한 라이브러리들은 대부분 자주 쓰이는 gitignore가 있기 때문에 참고하거나 그대로 쓰면 좋습니다.

https://www.gitignore.io/

#### `git diff`
파일에 어디가 바뀌었는 지를 확인할 수 있다. Staged가 아닌 modified, untracked 파일에 대해서만 확인이 가능함. —staged 옵션을 주면 저장소의 커밋과 staging area를 비교할 수 있다.

#### `git commit -a` 
tracked 파일을 자동으로 staging에 넣고 커밋하는 명령어. (Git add와 git commit을 한번에 할 수 있다)

#### `git log`
[Git - 커밋 히스토리 조회하기](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%BB%A4%EB%B0%8B-%ED%9E%88%EC%8A%A4%ED%86%A0%EB%A6%AC-%EC%A1%B0%ED%9A%8C%ED%95%98%EA%B8%B0)
시간상 가장 최근인 커밋 히스토리 조회하기
저자는 코드 작업을 수행한 원작자, 커미터는 마지막으로 이 작업을 저장소에 적용한 사람을 의미한다.

Git log의 다양한 옵션
```
-p 옵션 : 각 커밋의 diff 결과 값을 보여줌
—stat : 몇줄이 변경되었는지 통계자료 포함
—pretty=online : log를 이쁘게 한줄로 보여줌 (online, short, full)
—pretty=format:”%h : %s“ : 포맷을 사용할 수 있다.
—graph : 브랜치, 머지정보를 포함하는 그래프를 그려준다.
—since=2.weeks : 최근 2주간의 로그만 확인
```


### 4. 되돌리는 명령어
[Git - 되돌리기](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EB%90%98%EB%8F%8C%EB%A6%AC%EA%B8%B0)
실수는 대부분 복구할 수 있지만, 되돌린것을 복구할 수는 없다.

#### 1. 가장 최근에 완료한 커밋 수정하기
`git commit --amend`

앗차 빠진 파일!, 이전 커밋 오타 수정! 정말 마지막 수정. 최종최종최종 과 같은 커밋을 만들지 않겠다는 의도.

앗차 빠진 파일!, 이전 커밋 오타 수정! 정말 마지막 수정. 최종최종최종 과 같은 커밋을 만들지 않겠다는 의도.
push를 한 커밋이라면, ammend를 사용할 경우 `git push -f`를 사용해야하므로 새로운 커밋을 만드는게 더 낫다. (Force는 데이터 손실이 있을 수 있기 때문)

#### 2. 파일을 unstaged로 만들기
`git reset HEAD <file>`

#### 3. modified 파일을 되돌리기 (tracked 파일만 가능)
`git checkout -- <file>` : 수정된 내용을 모두 지우기 때문에 조심히 사용!

### 5. 리모트 저장소와 통신하는 명령어

#### `git remote -v` : 모든 리모트 저장소 

#### `git remote add <단축이름 like origin> <url>` : 리모트 추가하기

#### `git fetch <remote 주소 또는 단축이름>` : 로컬에는 없느나, 리모트에 있는 데이터를 모두 가져옴 ( git pull 은 fecth + merge이므로 조심히 사용)

#### `git push <리모트저장소> <브랜치이름>`
한번이라도 브랜치를 리모트에 푸시했으면, 해당 브랜치는 리모트저장소, 브랜치이름을 기록하고 있기 때문에 `git push` 명령어만으로 푸시할 수 있다. 


### 6. Git alias 이용해서 명령어 커스터마이징하기!
`git config --global alias.co checkout` : 체크아웃 대신 git co를 사용할 수 있다.
`git config --global alias.br branch`
`git config --global alias.ci commit`
`git config --global alias.st status`
`git config --global alias.cp cherry-pick`

## 6. 실습 및 과제
1. 본인의 로컬에 git alias를 모두 적용하기
2. Untracked, modified, staged를 모두 가진 `git status`를 만들기(스크린샷)
3. git commit —amend로 이전 커밋에 새로운 파일 추가해보기(사용하기전 커밋, 사용 후 커밋을  `git log --stat`을 이용해서 비교 스크린샷)
4. 오늘 배운 깃 명령어 요약해서 작성하기.
5. Git book 한글 버전 읽기 [Git - Book](https://git-scm.com/book/ko/v2)
	1. 시작하기
	2. git의 기초

## 7. 앞으로 남은 세미나
브랜치, 머지 그리고 리베이스

알면 좋은 git 도구들과 git 내부
