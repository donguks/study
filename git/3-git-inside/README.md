Git 내부 및 사용하면 좋은 명령어들

## 1. git은 커밋을 어떻게 관리할까?
브랜치에 대해서 알아보기 전에, 먼저 커밋이 어떻게 작동하는지를 이해하면 좋다.

### 1-1. 커밋을 하면 어떤일이?

아무 파일이나 만들어서 첫커밋을 해보자. 그러면 .git/objects에 아래와 같이 알 수 없는 폴더가 3개 생성된다.

```
objects
├── 2f
│   └── 781156939ad540b2434d012446154321e41e03 [32B]
├── 83
│   └── 207f0274383b4a79ff6d6c297e95204ba961bc [60B]
├── cf
│   └── 1fdb215f948795523cde2987107944d1374777 [135B]
├── info
└── pack

```
위에 생성되는 폴더와 파일들은 git이 관리 하는 object들이다. 커밋을 하면 3가지의 object가 만들어진다.

### 1-2. 커밋을 하면 만들어지는 3가지 object

커밋을 한다 = `commit object`, `tree object`, `blob object` 를 만든다.

### 1-3. 커밋 정보를 담고 있는 commit object
1) 파일 구조를 가지고 있는 tree object의 링크인 40글자 문자열

3) 이전 커밋을 가르키는 40글자의 문자열

2) 메타데이터(저자정보, 메시지)

commit object는 `git cat-file -p COMMIT_HASH`로 확인 할 수있다. 
```
tree 29e32fbed0e3524fab172aef6cd4e65dda6e56a5 
parent 9ef6949d8b52b0ad4143fb7181abdd533836a7e8 (이전 커밋에 대한 정보)
author 서동욱 <seodonguk@seodong-ug-ui-MacBookPro.local> 1571142926 +0900
committer 서동욱 <seodonguk@seodong-ug-ui-MacBookPro.local> 1571142926 +0900

aaa
```

### 1-4. 해당 커밋의 파일 트리 스냅샷 정보를 가지고 있는 tree object
tree object는`git cat-file -p TREE_HASH`로 확인할 수 있다.

(TREE_HASH는 commit object에서 구할 수 있다.)

```
100644 blob 78981922613b2afb6025042ff6bd878ac1994e85	a.a
100644 blob a82bd1659e5a6b420c4edfbdca82fed358397416	hello.rbi
```


### 1-5. 파일의 스냅샷인 blob object

blob object는 `git cat-file -p BLOB_HASH`로 확인할 수 있다.

```
파일 내용이 등장한다.
```

### 1-6. 그림으로 정리하는 git object
![](https://git-scm.com/book/en/v2/images/data-model-3.png)

## 2. git은 브랜치를 어떻게 관리할까?
git의 브랜치는 커밋사이를 쉽게 이동할 수 있는 어떤 링크다. 

(어떤 한 커밋을 가르키는 40글자의 문자열을 가진 파일일 뿐)

.git/refs/heads/master를 열어보면 특정 커밋 해쉬를 가지고 있다.

```
 91370f9dcd0207dcba2e5de7966f3366b5abee17
```

### 1. 새로 커밋을 할 경우 브랜치에 무슨일이?
master 브랜치에서 새로운 커밋을 작성하자. 

master 브랜치에는 새로운 커밋 해쉬를 가지게 된다.

### 2. 새로 브랜치를 만들면 어떻게 되지?
master 브랜치에서 test 브랜치를 만들어보자. `git branch test`

test 브랜치는 master의 최근 커밋 해쉬값을 가지게 된다.(.git/refs/heads/test 파일을 만들고 이 값을 저장한다.)

### 3. 현재 작업중인 브랜치는 어떻게 가르키지?
git은 `HEAD` 라는 포인터로 작업중인 로컬 브랜치를 가르킨다. 

git log만 입력하더라도 HEAD가 어디 커밋을 가르키고 있는지 확인 가능하다.

git은 HEAD가 가르키는 커밋을 기준으로 `Working directory`를 만든다.

HEAD는`.git/HEAD`에 저장한다.
```
ref: refs/heads/test
```

## 3. Git의 두가지 명령어
플럼빙(Plumbing) : git init, branch 등 git을 쉽게 쓰기 위한 명령어
포셀린(Procelain) : plumbing 명령어를 구성하고 있는 git의 저수준 명령어
위에서 쓰인 cat-file과 같은 명령어가 저수준 명령어다.

커밋을 한다 = git hash_object 명령어로 blob, tree, commit object를 만들고, git update-ref 명령어로 브랜치가 가르키는 커밋 주소를 업데이트한다.

## 4. Git이 파일 데이터 크기를 줄이는 방법
### 1. zlip을 이용해 압축한다
git의 대부분의 오브젝트 tree, commit, blob는 압축해서 저장한다.

### 2. 리모트 서버로 푸시하기전에 여러 버전이 존재하는 파일의 경우 차이값만 저장한다.
git-merge-with-conflict repo가 압축하기전 가지고 있는 오브젝트는 아래와 같다.
```console
find .git/objects -type f # 현재 git object를 모두 확인
.git/objects/0c/462352f050f90069324b1a2490ad5c862504c0
.git/objects/03/08951fcc6f7174e8f1a351e607ce3e46914215
.git/objects/56/0a8d745ff33664ba0815d2ca43b1d1c51c245d
.git/objects/bb/b0d6a5a6db0c54cfca7df52f9065aeab0431a3
.git/objects/bb/f08953b89f12205330498a0ef399466789237d
.git/objects/e2/cf5e790565bc0bf428eb2709e0bdf08cf9ca3e
.git/objects/18/a961e382123bb4bc7cef162f1a97ef03e5ebe6
.git/objects/10/f4701ffb0019daa15823833298d45b123c9bd4
.git/objects/d4/8d4c9085b9bb2d5a151acb15f3444f62d07c00
.git/objects/a9/e8115890c28ce6c5750a835f3a3b27d1407f87
.git/objects/a8/8fe08151d8f5385f314634a8614efe28f9d3c2
.git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391
.git/objects/e7/84803975de93681699432774305964e17f054b
.git/objects/cb/9d89e0ec6ab695953043a2073d5dca8ac00c5f
.git/objects/1b/275bb88e2778437fda73dc4a71332824758e78
.git/objects/48/6a3a8021c705071d01cbc0255c2cc7ba96ce44
.git/objects/14/7d8cd29d5f7967669c457ab6be8bedc55341f8

```

git gc를 이용해 압축을 해보자!
```
git gc # 다양한 버전을 가진 파일을 차이값만 저장하도록 변경
오브젝트 나열하는 중: 16, 완료.
오브젝트 개수 세는 중: 100% (16/16), 완료.
Delta compression using up to 4 threads
오브젝트 압축하는 중: 100% (6/6), 완료.
오브젝트 쓰는 중: 100% (16/16), 완료.
Total 16 (delta 1), reused 0 (delta 0)
```
git gc 명령어를 입력하면, git은 이름이나 크기가 비슷한 파일을 찾고 두 파일을 비교해서 한 파일은 다른 부분만 저장하도록 변경한다. 이러한 내용은 packfile에 저장한다. git gc 명령어를 입력 후 다시 오브젝트를 확인하면 오브젝트의 개수가 크게 줄어든 것을 확인 할 수 있다.

```console
find .git/objects -type f # 현재 git object를 모두 확인 
git/objects/bb/b0d6a5a6db0c54cfca7df52f9065aeab0431a3
.git/objects/pack/pack-f221d785e9ff4ea78a86086a9c8f6e22aa347f5e.pack
.git/objects/pack/pack-f221d785e9ff4ea78a86086a9c8f6e22aa347f5e.idx
.git/objects/info/packs
```
git은 버전1, 버전2의 파일이 있을 때 원본파일로 저장하는 것은 버전1이 아닌 버전2이다. 리모트에서 받아오는 것은 항상 최신버전이기 때문에 최근버전을 원본파일로 저장하는것이 성능상에서 가장 이득이 크기 때문이다.


## 5. Git 도구들

### 1. Reflog
git 은 자동으로 브랜치와 HEAD가 지난 몇달동안에 가르킨것을 모두 기록하고 있다. 이를 확인 할 수 있는 명령어는 git reflog다. reflog는 로컬 기록만 남기기 때문에 다른 사람의 로컬의 reflog는 확인 할 수 없다.

### 2. Show
git show는 특정 커밋의 커밋내용을 확인 할 수 있는 명령어다. 
git show HEAD (HEAD가 가르키는 브랜치의 최근 커밋을 보여줘!)

`^숫자`를 이용하면 좀 더 쉽게 다른 커밋으로 이동할 수 있다.
git show HEAD^1 # HEAD이 이전 커밋 확인
git show branch_name^3  # branch_name의 3번째 조상의 커밋

### 3. stashing과 cleaning
Stash를 이용하면 워킹디렉토리의 수정한 파일들만 저장함. Modified이면서 Tracked 상태인 파일과 Staging Area에 있는 파일을 저장한다. Untracked 파일인 경우에는 git add를 통해 staging에 추가한 후 stash를 이용해야한다.

`git stash list`를 이용하면 stash 명령어를 입력해 저장한 데이터의 리스트를 확인 할 수 있다.
`git stash apply stash@{1}`을 입력하면 stash@{1}의 데이터를 현재 워킹디렉토리에 적용해주며, --index 옵션을 주면 스테이징 상태까지 만들어준다.
`git stash drop` 은 stash에 있는 데이터를 제거해준다.
`git stash pop`은 stash를 적용하고 drop을 이용해 해당 stash data를 제거한다.

`git clean`을 이용하면 워킹디렉토리의 추적중이지 않은 파일을 모두 제거하는 명령어다.  `-f -d` 옵션을 주면 추적중이지 않은 하위 디렉토리의 변경사항까지 모두 제거한다. `-n` 옵션을 주면 명령어를 실행하지 않고 어떤 일을 하는지 미리 확인 할 수 있다. (다른명령어에서도 -n 옵션을 줄 수 있다.) 지워진 파일은 복구할 수 없기 때문에 git stash --all`을 이용하는것도 좋은 방법이다.

### 4. 커밋 메시지 수정하기
`git commit --amend`를 이용하면 가장 최근 커밋 내용을 수정할 수 있다.

`git rebase -i HEAD~3`을 이용하면 HEAD부터 3개의 커밋을 수정할 수 있다.
명령어를 입력하면 에디터창이 열리는데 이때 수정하고 싶은 커밋은 pick에서 edit으로 변경한 후 저장하자. 그러면 edit 하려는 커밋으로 이동하고, `git commit --amend`를 이용해 커밋 메시지를 수정할 수 있다. 그 후 `git rebase --continue`를 입력하면 다른 커밋을 수정하거나 수정 완료 할 수 있다.  pick에서 squash로 바꾸면 이전 커밋과 합쳐지도록 할 수 있다.

### 5. Reset
#### Git Reset 정확히 이해하기
git reset은 git checkout과 많이 비슷하기 때문에 헷갈리기 쉬운 명령어다.
둘의 차이를 구분하기 위해서는 3개의 트리 HEAD, index, 워킹디렉토리의 관점에서 이해해야한다.

HEAD : 마지막 커밋, 다음 커밋의 부모 커밋
index : 다음 커밋에 커밋할 스냅샷 (Staging Area)
워킹디렉토리 : 수정중이거나 새로 추가한 파일들

git init을 입력하고 file.txt를 하나 만들면 아래와 같다.
![reset ex1](https://git-scm.com/book/en/v2/images/reset-ex1.png)

그 후 file.txt를 스테이징에 추가하면 working directory에 파일을 인덱스에 추가하게 된다.
![reset ex2](https://git-scm.com/book/en/v2/images/reset-ex2.png)

스테이징에 있는 내역을 커밋을 하면 아래와 같이 HEAD는 새로운 커밋을 가르키고 그 커밋은 file.txt를 가르킨다.
![reset ex3](https://git-scm.com/book/en/v2/images/reset-ex3.png)

위 상태에서 git status를 입력하면 Index와 Working Directory의 tree의 차이값이 없기 때문에 아무런 내용이 나오지 않는다.

file.txt를 수정해서 스테이징에 추가하고 커밋을 완료하고, 이 과정을 한번 더 반복해서 총 3개의 커밋을 만들면 아래와 같다.
![reset start](https://git-scm.com/book/en/v2/images/reset-start.png)

reset 명령어는 3단계로 위 3개의 트리를 조작하는 명령어다.

1.  HEAD가 가리키는 브랜치를 옮긴다. 
2.  Index를 HEAD가 가리키는 상태로 만든다.  
3.  워킹 디렉토리를 Index의 상태로 만든다.

`git reset --soft HEAD~` 는 HEAD가 가르키는 브랜치 커밋만 이전 커밋(두번째 커밋)으로 이동시킨다. 따라서 아래와 같이 3번째 커밋의 커밋하기 직전으로 되돌리게 된다. (3번째 커밋의 변경내역이 모두 스테이징에 들어있음)
![reset soft](https://git-scm.com/book/en/v2/images/reset-soft.png)

`git reset --mixed HEAD~`는 HEAD가 가르키는 브랜치의 커밋과 인덱스를 이전커밋 트리로 변경한다. 즉 3 번째 커밋의 변경이력은 staging에는 들어있지 않지만 working directory에는 들어있어 언제든지 git add를 이용해서 staging에 추가할 수 있다.
![reset mixed](https://git-scm.com/book/en/v2/images/reset-mixed.png)

`git reset --hard HEAD~`는 worksing directory까지 변경시키며 세번째 커밋의 내용이 모두 사라진다.
![reset hard](https://git-scm.com/book/en/v2/images/reset-hard.png)

1.  HEAD가 가리키는 브랜치를 옮긴다.  _(`--soft`  옵션이 붙으면 여기까지)_
2.  Index를 HEAD가 가리키는 상태로 만든다.  _(`--hard`  옵션이 붙지 않았으면 여기까지)_
3.  워킹 디렉토리를 Index의 상태로 만든다.

git reset 명령어를 사용할 때 파일의 경로를 주면 1단계를 건너띄고 2~3단계를 적용한 파일을 가져오게된다. --hard와 같은 옵션을 주지않으면 --mixed이며 2단계만 적용하며 index를 업데이트한다.

#### git reset을 이용하여 특정 파일을 특정 버전으로 되돌리기

`git reset HEAD file.txt` 라는 명령어는 file.txt를 헤더의 파일로 스테이징에 올려달라는 의미로 보통 스테이징 취소에 사용한다. (HEAD는 생략가능)
`git reset commit file.txt` 명령어는 file.txt를 commit의 버전으로 가져와서 스테이징에 올려달라는 의미 --hard를 주면 스테이징과 워킹디렉토리에 적용하게 된다.
![reset path1](https://git-scm.com/book/en/v2/images/reset-path1.png)

#### git checkout branch?
git checkout은 HEAD의 브랜치가 가르키는 커밋을 변경하는 것이 아니라 HEAD가 가르키는 브랜치를 변경한다.  또한 working directory를 잘 보존한 상태에서 작업이 이루어진다. 아래는 checkout과 reset의 차이를 설명해준다.
![reset checkout](https://git-scm.com/book/en/v2/images/reset-checkout.png)


## 6. 과제
1. https://github.com/sdu6342/git-merge-with-no-conflict.git의 레포를 클론받아  `git rebase -i`를 이용하여 master 브랜치의 모든 커밋 메시지에 new를 추가하기. 
2.  1의 과제에서 reset을 이용해 master 브랜치의 최근 2개 커밋을 하나로 합치기. (git reset --soft HEAD~3)
3. 2과제 완료후 `git rebase -i HEAD~2`와 squash 옵션을 이용하여 2개의 커밋을 하나로 합치기
위 3개 과제 모두 `git log --pretty=oneline`스크린샷 남겨주세요!

### References
1) https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80
2) https://matthew-brett.github.io/curious-git/git_object_types.html
