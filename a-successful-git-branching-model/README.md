<!-- @format -->

원문 : [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

# 성공적인 Git 브랜치 모델

#### Vincent Driessen(2010년 1월 5일 화요일)
---
### Note of reflection(2020년 3월 5일)
이 모델은 git이 나오고 얼마 지나지 않은 10년도 더 된 2010년에 확립되었다. 10년간, git flow는 많은 소프트웨어 팀에서 표준처럼 여기게 될 정도로 유명해졌다. 하지만 git flow는 신념도 아니고 만병통치약도 아니다.

10년간, git은 세계를 집어삼켰고, git과 함께한 대부분의 유명한 소프트웨어들은 웹앱으로 바뀌었다. 웹앱은 보통 지속적으로 배포(CD, continuous Delivery)되고 롤백 되지 않는다. 또한 여러 개의 버전을 유지할 필요도 없다.

내가 10년 전 이 포스트를 쓸 때 이런 소프트웨어는 존재하지 않았다. 만약 당신의 팀이 CD를 하고 있다면 git flow같은 구시대의 유물보다는 [GitHub flow](https://guides.github.com/introduction/flow/) 처럼 단순한 워크플로우를 채택할 것을 추천한다.

하지만 혹시 당신의 팀이 명확히 버전 표기가 되어야하거나 소프트웨어에 대해 다수의 버전을 유지해야한다면 git flow는 여전히 좋은 워크플로우다. 이 경우에는 글을 계속 읽어도 좋다.

결론적으로 만병통치약은 없다. 우선 스스로의 상황을 생각해보고, 무조건 남이 좋다고 해서 편향적으로 워크 플로우를 선택하지마라.

---

이 번 포스트에서는 작년 즈음 프로젝트(일적이든 ,개인적이든)에 도입한 성공적인 개발 모델에 대해 소개하려고 한다. 오랫동안 이 주제에 대해 글을 쓰려고 했으나 그간 시간이 없었다. 프로젝트의 상세에 대해서는 말하지 않고 브랜치 전략과 릴리즈 관리에 대해서만 이야기하겠다.

<img style="margin-top : 3rem; margin-left: auto; margin-right : auto; width : 400px; display : block" src="images/git-model@2x.png"/>

## 왜 git인가?

중앙 소스 코드 컨트롤 시스템과 git에 대한 찬반론은 [여기](https://git.wiki.kernel.org/index.php/GitSvnComparsion)에서 살펴보가 바란다. 저 곳에서 이미 뜨거운 논쟁이 벌어지고 있다(2021년은 git의 압승). 나는 개발자로서 git을 다른 툴보다 선호한다. git은 개발자가 머지와 브랜치에 대해 생각하는 방식을 바꾸었다. 고전적인 CVS/SVN에서 머지와 브랜치는 늘 조금은 무서운 것이었다("머지 충돌!").

하지만 git에서는 이러한 동작들이 부담 없고도 단순하게 이루어진다. 머지/브랜치는 심지어 git에서 굉장히 일상적인 동작이다. 가령 CVS/SVN을 다룬 책을 보면 브랜치와 머지는 마지막 챕터에서 처음 다뤄진다. 반면에 git을 다룬 책에서는 해당 내용이 3번쨰 챕터 정도에서 다루어진다.

git에서 브랜치와 머지는 단순하고 반복적이기 때문에 두려운 것들이 아니다.

툴에 대해서는 이 정도로 이야기하고 개발 모델에 대해 이야기 해보자. 여기서 이야기하려는 모델은 모든 팀 인원이 잘 관리된 소프트웨어 개발 프로세스를 위해 따라야할 절차적인 것들이다.

## 분산, 하지만 중앙

중앙(Centralized) 저장소는 소개할 브랜치 모델과 잘 어울린다. 이런 저장소는 논리적으로만 중앙 집중적일 뿐이다(git은 애초에 분산 시스템이기 때문에 기술적인 레벨에서 중앙 저장소를 지원하지 않는다). 이 저장소를 git 사용자들이 익숙한 origin이라고 부르겠다.

<img style="margin-top : 3rem; margin-left: auto; margin-right : auto; width : 400px; display : block" src="images/centr-decentr@2x.png"/>

각 개발자들은 origin으로부터 풀/푸쉬를 수행한다. 하지만, 이와 별개로 개발자들은 서로 간 풀/푸쉬를 수행할 수 있다. 즉, 서로가 하나의 작은 팀을 이루는 것이다. 이런 시나리에서는 두 명 이상의 개발자가 하나의 새롭고 큰 기능을 함께 개발하게 된다.

그림에서는 Alice와 Bob의 팀, Alice와 David의 팀, 그리고 Clair와 David의 팀을 확인할 수 있다.

실질적으로는 Alice가 그저 Bob의 저장소를 가리키는 bob이라는 이름의 원격 저장소를 정의한 것에 불과하다(git remote add bob).

## 메인 브랜치

git flow는 기본적으로 원래 개발 모델로부터 영감을 받았다. 중앙 저장소(origin)는 보통 아래 두 개의 메인브랜치를 게속 유지한다.

-   master
-   develop

<img style="margin-top : 3rem; margin-left: auto; margin-right : auto; width : 400px; display : block" src="images/main-branches@2x.png"/>

origin의 master 브랜치는 모든 git 사용자들에게 친숙하다(git init으로 처음 만들어지는 기본 브랜치 이름이니까). master 브랜치와는 별개로 develop 브랜치가 존재한다.

origin/master는 메인 브랜치로 HEAD(최신커밋)은 항상 최신 릴리즈 제품을 의미한다.
origin/develop 또한 메인 브랜치로 HEAD는 항상 개발중인 최신 버전을 의미한다. develop 브랜치는 일종의 "통합 브랜치"로서의 역할도 한다. 흔히 말하는 nightly 빌드(소스 작업 결과물로 나오는 임시 빌드 버전)라고 생각해도 된다.

develop 브랜치의 소스코드가 안정적인 상태에 진입하고 릴리즈 준비가 되었을 때 모든 변화는 다시 master로 머지된다. 그리고 해당 커밋에는 릴리즈 넘버로 태그가 붙는다. 이런 일련의 동작이 어떻게 행해지는지는 추후에 살펴볼 것이다.

그러므로 변화의 결과물(develop)이 다시 master로 머지 될 때 이 커밋은 새로운 릴리즈 제품이 된다. 즉, master의 커밋들은 하나하나가 릴리즈가 된다. 어떤 프로젝트에서는 master로 머지가 발생할 때마다 git 훅(일정 동작 중간에 새로운 동작을 넣는 등)을 이용해 자동으로 빌드/배포를 진행한다.

## 서포트 브랜치

이런 메인 브랜치 외에도 우리의 개발 모델은 추가적인 서포트 브랜치들을 사용한다. 서포트 브랜치를 사용하면 팀 내에 구성원들이 서로 무관하게 병렬적으로 작업할 수 있다. 이외에도 이슈 트래킹, 릴리즈, 또는 핫픽스 등의 작업에서 큰 도움을 받을 수 있다. 메인 브랜치들과 다르게 서포트 브랜치들은 항상 제한된 수를 유지해야한다. 결국에는 머지되어 사라질 브랜치들이기 때문이다.

대표적으로는 이런 브랜치들이 있다(팀의 필요에 따라 다양한 브랜치를 생각할 수 있을 것이다).

-   feature 브랜치
-   release 블내치
-   hotfix 브랜치
    각 브랜치들은 특별한 목적을 가지며 저마다 엄격한 규칙을 준수한다. 이 규칙으로 스스로가 어떤 브랜치에 종속되고 어떤 브랜치로 합쳐지는지 등이 결정된다.

서포트 브랜치라고 해서 기술적으로 특별한 브랜치는 아니다. 그냥 git branch를 통해 만들어지는 평범한 git의 브랜치이다.

### feature 브랜치

> 어디서 브랜치 되는가? - develop

> 어디로 머지 되는가? - develop

> 이름 규칙 - master, develop, release-*, 또는 hotfix-*외에 전부

<img style="margin-top : 3rem; margin-left: auto; margin-right : auto; width : 300px; display : block" src="images/fb@2x.png"/>

feature 브랜치(혹은 Topic)는 단기나 장기적인 새 기능 개발을 위해 사용된다. feature 브랜치는 develop에 종속되고 develop으로 합쳐진다. 즉, master 브랜치와는 완전히 분리 되어있어 릴리즈와 상관 없이 기능 개발을 진행할 수 있다. 중요한 점은 해당 브랜치가 기능 개발 중에만 유지되고, 기능에 대한 결정(추가하거나 버리거나)이 났을 때는 사라진다는 점이다. 추가한다면 develop 브랜치로 합쳐질 것이고 버려진다면 그대로 브랜치가 제거 될 것이다.

feature 브랜치는 일반적으로 개발자의 로컬 저장소에만 존재한다. origin에는 존재하지 않는다.

```bash
$git checkout -b myfeature develop #develop으로부터 분기
#개발 진행
$git checkout develop #개발 완료된 기능을 머지하기 위해 develop으로
$git merge --no-ff myfeature
$git branch -d myfeature #합쳐졌으므로 브랜치 제거
$git push origin develop
```

--no-ff 플래그는 패스트 포워드 여부를 무시하고 머지가 항상 새 커밋을 만들게 끔 하는 옵션이다. 이를 통해 소스 작업의 모든 이력을 보존할 수 있다. 패스트 포워드의 겨우 머지 이전 브랜치들의 이력을 볼 수가 없고 수동으로 로그 메시지를 읽어야만 확인할 수 있다. 커밋을 되돌리는 작업등을 할 때에도 복잡해진다. 반드시 머지시에는 --no-ff 옵션을 붙이자.

쓸데없이 빈 커밋을 만든다고 여길지도 모르지만 득이 실보다 크다.

<img style="margin-top : 3rem; margin-left: auto; margin-right : auto; width : 400px; display : block" src="images/merge-without-ff@2x.png"/>

### release 브랜치

> 어디서 브랜치 되는가? - develop

> 어디로 머지 되는가? - develop과 master

> 이름 규칙 - release-\*

release 브랜치는 새 제품 릴리즈 준비를 돕는다. 릴리즈 직전 마무리에 활용된다. 이를 통해 마이너 버그 픽스와 릴리즈를 위한 메타데이터(버전 넘버등) 갱신과 같은 작업을 진행할 수 있다. 이 모든 걸 release 브랜치에서 진행함으로써 develop 브랜치를 가능한한 깔끔히 유지할 수 있다.

release 브랜치가 만들어지는 순간은 develop이 다음 릴리즈를 할 준비가 되었을 때다. 계획된 릴리즈에 따른 기능들이 전부 개발 되었을 때 release 브랜치가 만들어지는 것이다. 먼 미래의 릴리즈를 위한 기능들은 포함되지 않는다.

처음 브랜치를 하고 난 뒤 반드시 버전 넘버 같은 릴리즈 메타데이터를 갱신한다. 이 때 브랜치나 업데이트 데이터에 "next releae"처럼 대충 기록하는 게 아니라 릴리즈의 정확한 메타데이터를 입력한다.

release 브랜치는 develop 브랜치로부터 만들어진다. 버전 1.1.5가 현재 제품 릴리즈고 다음 릴리즈를 배포할 것이라고 가정해보자. develop은 다음 릴리즈를 위해 모든 준비를 마쳤다. 우리가 해야할 일은 버전 넘버를 결정하는 것이다. 1.1.6이 될 수도 있고 2.0이 될 수도 있다. 그리고 브랜치를 만든다. 그 릴리즈 브랜치의 이름에는 버전 넘버를 준다.

```bash
$git checkout -b release-1.2 develop
$./update_version.sh 1.2 #릴리즈 메타데이터(버전 정보) 업데이트 스크립트
$git commit -a -m "updated version numberto 1.2" #반드시 커밋으로 이력을 남긴다.
```

여기서 update_version.sh는 버전 정보를 업데이트하는 가상의 스크립트이다.

해당 브랜치는 다음 릴리즈가 출시 될 때까지 존재한다. 그동안 마이너한 버그 픽스가 이 브랜치에 적용될 수도 있다. 중요한 건 이런 버그 픽스를 develop에 하지 않는 것이다. 병렬로 기능 개발이 진행중인 feature 브랜치를 병합해 새 기능을 추가하는 것도 절대 금지다. 새 기능 추가는 다음 릴리즈에서 진행한다.

release 브랜치가 준비가 되면 첫번째로 해당 브랜치를 master로 머지한다. master의 모든 커밋들은 언급했듯이 실제 릴리즈 그 자체이기 때문이다. 그 다음 master를 커밋하고 버전 tag를 달아준다. 마지막으로 develop에 release 브랜치에서 진행한 모든 작업 내용을 반영하기 위해 develop 브랜치로 머지해준다. 그래야 릴리즈 메타데이터와 버그 픽스를 develop에도 반영할 수 있다.

```bash
$git checkout master
$git merge --no-ff release-1.2
$git tag -a 1.2
$git checkout develop
$git merge --no-ff release-1.2
$git branch -d release-1.2
```

### hotfix 브랜치

> 어디서 브랜치 되는가? - master

> 어디로 머지 되는가? - develop과 master

> 이름 규칙 - hotfix-\*

<img style="margin-top : 3rem; margin-left: auto; margin-right : auto; width : 400px; display : block" src="images/hotfix-branches@2x.png"/>

hotfix 브랜치는 릴리즈 브랜치와 거의 유사하다. 다른 점은 계획된 작업이 아니라는 점이다. hotfix 브랜치는 이미 릴리즈된 버전의 제품에서 즉각적인 픽스를 요하는 문제가 벌어졌을 때를 대비해 존재한다. hotfix 브랜치는 상응하는 tag의 master 커밋으로부터 분기된다.

가장 중요한 건 hotfix가 진행되는 도중에도 develop 브랜치에서 나머지 팀 인원들이 병렬로 일을 이어나갈 수 있게끔 해야한다는 것이다.

hotfix 브랜치는 master 브랜치로부터 생성된다. 가령, 1.2 버전의 릴리즈가 출시 되었고 심각한 버그가 발생했다고 생각해보자. 하지만 그렇다고 "통합 브랜치인" develop 브랜치를 변경하기에는 부담이 너무 크다. 무엇보다 다음 릴리즈에 쓰일 새로운 기능을 개발 중인 팀 인원들이 영향을 받을 수 있다.

```bash
$git checkout -b hotfix-1.2.1 master #최신 master의 커밋은 1.2버전의 릴리즈
$./update_version.sh 1.2.1
$git commit -a -m "updated version numberto 1.2.1" #반드시 커밋으로 이력을 '바로' 남긴다.
```

반드시 버전 정보를 먼저 업데이트하는 것을 잊지말자.

```bash
#결함 수정
$git commit -m "Fixed severe production problem"
```

위처럼 모든 작업이 끝난 뒤 바로 master로 hotfix 브랜치를 머지한다. develop에도 해당 내용을 반영해야하므로 develop으로도 머지한다.

```bash
$git checkout master
$git merge --no-ff hotfix-1.2.1
$git tag -a 1.2.1 #릴리즈이므로 태그를 잊지말 것
$git checkout develop
$git merge --no-ff hotfix-1.2.1
$git branch -d hotfix-1.2.1
```

> 만약 이 때 다음 릴리즈를 위한 release 브랜치가 존재한다면 해당 내용을 develop이 아니라 release로 머지해야한다. 어차피 release는 곧 develop으로 머지될 예정이기 때문이다.
