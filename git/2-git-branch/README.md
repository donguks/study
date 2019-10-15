브랜치때문에 git을 사용한다고 해도 과언이 아니다. 브랜치를 이용하면 여러 이슈를 동시에 여려명이 독립적으로 진행할 수 있고, merge와 rebase를 이용해서 작업을 합칠 수 있다.
커밋이 무엇인지 깊게 확인하고, 브랜치가 무엇이고, git이 브랜치를 어떻게 관리하는지, 더나아가 merge와 rebase에 대해서 알아보자.

## 1. 커밋이란 무엇인가?
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

### 1-6. git commit의 큰그림
커밋을 하면 아래의 큰그림을 그리게된다.

![](https://git-scm.com/book/en/v2/images/commit-and-tree.png)


### 1-7 커밋은 이전 커밋은 가르킨다.
맨 처음 작성하는 커밋은 당연히 이전 커밋이 없지만,

두번째 커밋 부터는 항상 이전의 커밋이 무엇인지를 가르킨다. 

(따라서 git은 차이값이 아니라 스냅샷을 저장하더라도 이전 커밋과 비교해서 차이를 구할 수 있다.)

![](https://git-scm.com/book/en/v2/images/commits-and-parents.png)

## 2. 브랜치는 무엇인가?
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

![]([https://git-scm.com/book/en/v2/images/two-branches.png)

### 3. 현재 작업중인 브랜치는 어떻게 가르키지?
git은 `HEAD` 라는 포인터로 작업중인 로컬 브랜치를 가르킨다. 

git log만 입력하더라도 HEAD가 어디 커밋을 가르키고 있는지 확인 가능하다.

git은 HEAD가 가르키는 커밋을 기준으로 `Working directory`를 만든다.

HEAD는`.git/HEAD`에 저장한다.
```
ref: refs/heads/test
```

### 4. 브랜치를 이동하는 방법 = HEAD를 이동하는 방법 = git checkout
`git checkout test`를 입력하면 현재 브랜치는 test가 된다.

이때 .git/HEAD 파일에 test branch가 저장되고 working directory는 이에 맞게 변경된다.(지금은 변경내역이 없어서 아무것도 바뀌지 않음)

![](https://git-scm.com/book/en/v2/images/head-to-master.png)
![](https://git-scm.com/book/en/v2/images/head-to-testing.png)

### 5. 새로운 브랜치에서 커밋을 하게 되면?
test 브랜치에서 아무거나 커밋을 해보자.

그러면 test 브랜치는 새로운 커밋 해쉬를 갖게되고, HEAD 또한 최근 커밋을 가르킨다. master는 여전히 이전 커밋을 유지한다.

![](https://git-scm.com/book/en/v2/images/advance-testing.png)

### 이쯤에서 퀴즈!
파일 리스트 = HEAD, refs/heads/master, refs/heads/test

*퀴즈 1)* 위의 예제에서 git checkout master를 할 경우 어떻게 되는지 그림을 그리고, 업데이트 되는 파일과 그렇지 않은 파일을 구분하기.

*퀴즈 2)* 퀴즈 1)의 상태에서 master의 브랜치에서 커밋을 두번 더 했을 경우 어떻게 되는지 그림을 그리고, 업데이트 되는 파일과 그렇지 않은 파일을 구분하기

퀴즈를 풀고 각가 `git log --oneline --decorate --graph --all`을 이용해서 확인해보자(스크린샷 남기기)

## 3. 머지란 무엇인가?
위의 퀴즈2)에서 master와 test의 브랜치는 갈라졌다. 이것을 하나의 브랜치로 합치는 과정을 머지라고 한다.

## 4. 이쯤와서 정리하는 branch 명령어들

## 5. 리베이스란 무엇인가?
rebase 실제로 해보는 과제 만들기

## 6. 대충 브랜치, 머지, 리베이스를 동적으로 확인 할 수 있는 사이트


### References
1) https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80
2) https://matthew-brett.github.io/curious-git/git_object_types.html
3)


