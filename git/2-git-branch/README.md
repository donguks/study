브랜치때문에 git을 사용한다고 해도 과언이 아니다. 브랜치를 이용하면 여러 이슈를 동시에 여려명이 독립적으로 진행할 수 있고, 머지와 리베이스를 이용해서 작업을 합칠 수 있다. 이를 각각 알아보자~

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

*퀴즈 2)* 퀴즈 1)의 상태에서 master의 브랜치에서 커밋을 한번 더 했을 경우 어떻게 되는지 그림을 그리고, 업데이트 되는 파일과 그렇지 않은 파일을 구분하기

퀴즈를 풀고 각자 `git log --oneline --decorate --graph --all`을 이용해서 확인해보자(스크린샷 남기기)

## 3. 머지란 무엇인가?
퀴즈 2의 정답은 아래와 같다. master와 test의 브랜치는 갈라졌다. 
![](https://git-scm.com/book/en/v2/images/advance-master.png)

이것을 하나의 브랜치로 합치는 과정을 머지라고 한다.

실습을 위해서 아래의 스토리로 커밋과 브랜치를 만들자.

1. 마스터 브랜치에서 커밋 3개를 만든다.
	1. 파일 수정, 스테이지에 추가한 후 `git commit -m "C"`의 반복 ( 메시지는 C0 ~ C2로)
2. 마스터 브랜치에서 iss53 브랜치를 만든 후 커밋을 하나 만든다.
	1. `git checkout -b iss53` (iss53브랜치를 만들고 HEAD를 이동)
	2.  파일 수정, 스테이지에 추가한 후 `git commit -m "C3"`
3.  마스터 브랜치에 급한 이슈가 발생해서 새로운 브랜치 만들어야 한다.
	1. `git checkout master` 마스터 브랜치로 이동
	2. `git checkout -b hotfix` (hotfix 브랜치를 만들고 HEAD 이동)  
	3.  파일 수정, 스테이지에 추가한 후 `git commit -m "C4"`

위 3가지의 과정을 진행하면 아래의 그림과 같다.
![`master` 브랜치에서 갈라져 나온 hotfix 브랜치](https://git-scm.com/book/en/v2/images/basic-branching-4.png)

머지에는 fast forwad (고속 감기), 3 way로 2종류가 있다. 하나씩 알아보자.

### 1. fast forward 머지란 무엇인가?
위의 예제에서 hotfix는 작업을 완료했다 가정하고 master로 merge는 상황을 상상하자.

`git checkout master`

`git merge hotfix` ( HEAD에서 hotfix의 커밋을 가져오겠다는 뜻)

merge 명령어를 입력하면 fast-forward 가 등장한다.
우리는 mater와 hotfix를 머지할때, hotfix는 단순히 master의 최근 커밋으로 부터 몇 커밋 이동했을 뿐이다. 따라서 master의 브랜치가 hotfix branch를 가르키게 만들면 머지가 끝난다. git은 이런 방식의 머지를 fast-forward라 한다. 아래의 그림과 같다.

![Merge 후 `hotfix` 같은 것을 가리키는 `master` 브랜치](https://git-scm.com/book/en/v2/images/basic-branching-5.png)

hotfix는 사용하지 않을 예정이므로 브랜치를 지워주자.
`git branch -d hotfix`

### 2. 3 way 머지란 무엇인가.
iss53의 브랜치에서 커밋을 하나 더 추가하자(C5) 그렇다면 아래의 그림처럼 된다.
![master와 별개로 진행하는 iss53 브랜치](https://git-scm.com/book/en/v2/images/basic-branching-6.png)

 iss53의 작업이 마무리 되었다고 생각하고 iss53을 master로 머지해보자.

`git checkout master`

`git merge iss53`

이번엔 fast-forward와는 메시지가 조금 다르다. 이전과 상황이 다르기 때문이다.

이전에는 C2와 C2에서 시작한 C4 커밋을 비교했지만(부모와 자식),  지금은 C5와 C4(자식과 자식)을 비교해야 한다. 

이 경우 git은 공통 부모인 C2커밋과 각 브랜치가 가르키는 두 커밋인 C5, C4 3개를 비교해서 머지를 한다. 그렇기 때문에 3 way 라고 한다. 아래의 그림은 머지할 때 참고하는 3개의 커밋을 보여준다.
![커밋 3개를 Merge](https://git-scm.com/book/en/v2/images/basic-merging-1.png)

3-way 머지를 진행하면 단순히 브랜치를 이동시키지는 못하고 새로운 커밋을 만들게 된다. 이때 생성되는 커밋을 머지 커밋이라 하며, 부모가 여러개(여기서는 2개)가 된다. 아래의 그림에서 C6는 머지 커밋이다.

![Merge 커밋](https://git-scm.com/book/en/v2/images/basic-merging-2.png)

우리가 github을 이용해서 하나의 브랜치로 작업을 하더라도, 수 많은 머지 커밋이 발생하는 것을 경험을 했을 것이다. 그 이유는 각자의 로컬, 리모트는 모두 다른 브랜치이므로 3-way 머지가 빈번하게 일어나기 때문이다. 리모트 마스터 브랜치가 c0 커밋일 때, 내가 로컬에 c0에서 c1를 만들었고, 누군가 c0에서 c2를 만들어 리모트의 마스터 브랜치로 푸시를 했다고 가정하자. 나의 로컬 마스터(c1)를 리모트 마스터(c2)으로 푸시할 때 3-way 머지를 진행하므로 머지커밋이 생성된다.

### 3. 충돌
fast-forward의 경우 단순히 브랜치를 옮기는 것이므로 충돌이 존재할 수 없다. 3-way 의 경우 충돌이 발생할 수 있다.  (이미 위의 예제를 따라하다 충돌을 경험할 수도 있다.) 

git은 머지할 때 자식 커밋들의 변경 내역에서 무엇이 더 중요한 변경인지 알 수 없기 때문이다.  위의 예제에서 C4와 C5에서 같은 파일내의 같은 줄을 수정했으면, git 을 이를 자동으로 머지 할 수 없기 때문에 충돌을 알리고 사용자가 직접 선택하게 한다. 

머지 중에 충돌이 날 경우 git stauts를 입력하면 both modified 파일을 확인 할 수 있다.
충돌이 난 곳을 확인후 내용을 수정한다. (>>>> HEAD와 같은 내용을 지운다.) 모두 스테이징에 추가한 후 머지 커밋(C6)를 만들 수 있다.

## 4. 이쯤와서 정리하는 branch 명령어들

`git branch` git 브랜치 리스트

`git branch -v` 각 브랜치와 가르키고 있는 커밋

`git branch --merged` 현재 브랜치를 기준으로 머지된 브랜치만 보여줌.

`git branch <name>` 새로운 브랜치 만들기. HEAD는 여전히 현재 브랜치

`git checkout -b <name>` 새로운 브랜치를 만들고, 브랜치로 이동

`git branch -d <name>` 로컬 브랜치 삭제 (머지하지 않았으면 -D로 지워야함)


## 5. 리베이스란 무엇인가?
rebase 실제로 해보는 과제 만들기

## 6. 대충 브랜치, 머지, 리베이스를 동적으로 확인 할 수 있는 사이트


### References
1) https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80
2) https://matthew-brett.github.io/curious-git/git_object_types.html
