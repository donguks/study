브랜치때문에 git을 사용한다고 해도 과언이 아니다. 

브랜치를 이용하면 여러 이슈를 동시에 여려명이 독립적으로 진행할 수 있고, merge와 rebase를 이용해서 작업을 합칠 수 있다.

브랜치가 무엇이고, git이 브랜치를 어떻게 관리하는지, 더나아가 merge와 rebase에 대해서 알아보자.

## 1. 커밋이란 무엇인가?
브랜치에 대해서 알아보기 전에, 먼저 커밋이 어떻게 작동하는지를 이해하면 좋다.

### 1-1. 커밋을 하면 무슨일이?
git에서 커밋을 하면,  커밋에 대한 정보를 가지고 있는 commit object, 그 순간의 디렉토리를 기억하는 tree 오브젝트,  그 순간의 파일을 저장하는 blob object 가 생성된다. 각 object는 다음의 순서로 생성된다.

스테이지 상태의 모든 파일을 새로운 블랍 오브젝트로 만든다. 바뀌지 않은 파일은 예전에 저장해둔 blob object의 주소로, 업데이트된 파일들은 새로운 blob 주소를 저장하는 tree object 를 생성한다. 그리고 이 tree object의 주소와 커밋 메시지 등의 메타 데이터를 저장하는 commit object를 생성한다.  (git이 이것을 어떻게 구체적으로 관리하는지는 다음 세미나!) 

아래의 그림을 보면 한 커밋이 가지고 있는 object들을 한눈에 볼 수 있다.
![](https://git-scm.com/book/en/v2/images/commit-and-tree.png)


### 1-2 커밋은 이전 커밋은 가르킨다.
맨 처음 작성하는 커밋은 당연히 이전 커밋이 없지만,

두번째 커밋 부터는 항상 이전의 커밋이 무엇인지를 가르킨다. 

따라서 git은 차이값이 아니라 스냅샷을 저장하더라도 이전 커밋과 비교해서 차이를 구할 수 있다.

머지한 커밋의 경우 이전 커밋은 여러개다.

![](https://git-scm.com/book/en/v2/images/commits-and-parents.png)

## 2. 브랜치란 무엇인가?
git의 브랜치는 커밋사이를 쉽게 이동할 수 있는 어떤 링크다. 

(어떤 한 커밋을 가르키는 40글자의 문자열을 가진 파일일 뿐)

### 1. 새로 커밋을 할 경우 브랜치에 무슨일이?
master 브랜치에서 새로운 커밋을 작성하자. 

master 브랜치에 새로운 커밋 해쉬를 저장하게 된다.

### 2. 새로 브랜치를 만들면 어떻게 되지?
master 브랜치에서 test 브랜치를 만들어보자. `git branch test`

test 브랜치는 master의 최근 커밋 해쉬값을 가지게 된다.

![](https://git-scm.com/book/en/v2/images/two-branches.png)

### 3. 현재 작업중인 브랜치는 어떻게 가르키지?
git은 `HEAD` 라는 포인터로 작업중인 로컬 브랜치를 가르킨다. 

git log만 입력하더라도 HEAD가 어디 커밋을 가르키고 있는지 확인 가능하다.

git은 HEAD가 가르키는 커밋을 기준으로 `Working directory`를 만든다.

### 4. 브랜치를 이동하는 방법 = HEAD를 이동하는 방법 = git checkout
`git checkout test`를 입력하면 현재 브랜치는 test가 된다. (git checkout 은 HEAD를 이동하는 방법이고, test는 특정 커밋을 가르키고 있기 때문에 가능한 명령어)

![](https://git-scm.com/book/en/v2/images/head-to-master.png)
![](https://git-scm.com/book/en/v2/images/head-to-testing.png)

### 5. 새로운 브랜치에서 커밋을 하게 되면?
test 브랜치에서 아무거나 커밋을 해보자.

그러면 test 브랜치는 새로운 커밋 해쉬를 갖게되고, HEAD 또한 최근 커밋을 가르킨다. master는 여전히 이전 커밋을 유지한다.

![](https://git-scm.com/book/en/v2/images/advance-testing.png)

### 6. 결국 모두 주소를 가진 파일들이다.
위의 예제에서 HEAD, master 브랜치, test 브랜치는 각각 .git 폴더에서 HEAD, refs/heads/master, refs/heads/test 에 저장된 문자열이다.

### 7. 이쯤에서 퀴즈!
git이 관리하는 파일 : HEAD, refs/heads/master, refs/heads/test

*퀴즈 1)* 위의 예제에서 git checkout master를 할 경우 어떻게 되는지 그림을 그리고, 업데이트 되는 파일과 그렇지 않은 파일을 구분하기.

*퀴즈 2)* 퀴즈 1)의 상태에서 master의 브랜치에서 커밋을 두번 더 했을 경우 어떻게 되는지 그림을 그리고, 업데이트 되는 파일과 그렇지 않은 파일을 구분하기

퀴즈를 풀고 각자 `git log --oneline --decorate --graph --all`을 이용해서 확인해보자(스크린샷 남기기)

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


