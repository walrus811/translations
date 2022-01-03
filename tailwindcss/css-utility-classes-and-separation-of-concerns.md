# CSS 유틸리티 클래스와 "관심사 분리(Separation of Concerns)"

https://adamwathan.me/css-utility-classes-and-separation-of-concerns/

TailWindCSS 개발자, Adam Wathan

2017년 8월 7일

지난 수년간 내가 CSS를 작성하는 방식은 "시맨틱"한 방식에서 "기능적인 CSS"라고 불리는 방식으로 바뀌어갔다.

이런 방식은 많은 개발자로부터 감정적인 반응을 이끌어냈다. 그래서 이 자리를 빌어 어떻게 그간 이런 관점을 지니게 되었는지와 몇 가지 교훈과 인사이트를 공유하려고 한다.

### 페이즈 1: "시맨틱" CSS

어떻게 좋은 CSS를 작성하는지 배우려고 할 때 좋은 사례중 하나로 듣게 되는 게 하나 있는데 바로 "관심사 분리(SOC)"이다.

이는 곧, HTML은 컨텐트에 관한 정보만을 포함하고 모든 스타일 결정은 CSS에서 이루어져야한다는 것이다.

이 HTML을 살펴보자.

```HTML
<p class="text-center">
    Hello there!
</p>
```

`.text-center` 클래스가 보이는가? 텍스트를 중앙에 놓는 디자인 결정이다. 즉, 이 코드는 "관심사 분리"를 위반했다. 스타일 정보를 HTML에 넣었기 때문이다.

대신 권장되는 방법은 컨텐트에 기반한 이름의 클래스를 엘레멘트에게 부여하는 것이다. 그리고 마크업을 스타일링하기 위해 이 클래스를 CSS에서 연결고리으로 사용한다.

```HTML
<style>
.greeting {
    text-align: center;
}
</style>

<p class="greeting">
    Hello there!
</p>
```

이런 방식의 전형적인 예는 [CSS Zen Garden](http://www.csszengarden.com/)이다. 이 사이트는 "관심사 분리"를 한다면 스타일 시트를 바꾸는 것만으로 디자인을 새롭게 할 수 있다는 것을 보여주기 위해 디자인 되었다.

내 워크플로우는 다음과 같았다.

1. 새 UI에 필요한 마크업을 작성한다(예시로는 저자를 소개하는 카드를 사용했다).

```HTML
<div>
  <img src="https://cdn-images-1.medium.com/max/1600/0*o3c1g40EXj65Fq9k." alt="">
  <div>
    <h2>Adam Wathan</h2>
    <p>
      Adam is a rad dude who likes TDD, Active Record, and garlic bread with cheese. He also hosts a decent podcast and has never had a really great haircut.
    </p>
  </div>
</div>
```

2. 컨텐츠에 기반해 설명적인 클래스를 추가한다.

```HTML
<div class="author-bio">
    <img src="https://cdn-images-1.medium.com/max/1600/0*o3c1g40EXj65Fq9k." alt="">
    <div>
      <h2>Adam Wathan</h2>
      <p>
        Adam is a rad dude who likes TDD, Active Record, and garlic bread with cheese. He also hosts a decent podcast and has never had a really great haircut.
      </p>
    </div>
  </div>
```

3. 마크업을 스타일링하기 위해 CSS/Less/Sass 내에 해당 클래스를 연결고리로써 사용한다.

```CSS
.author-bio {
  background-color: white;
  border: 1px solid hsl(0,0%,85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
  > img {
    display: block;
    width: 100%;
    height: auto;
  }
  > div {
    padding: 1rem;
    > h2 {
      font-size: 1.25rem;
      color: rgba(0,0,0,0.8);
    }
    > p {
      font-size: 1rem;
      color: rgba(0,0,0,0.75);
      line-height: 1.5;
    }
  }
}
```

결과는 다음과 같다.

![soc1](./images/soc1.png)

이 방식이 직관적으로는 이치에 맞는 것 같았고 한동안 나는 HTML과 CSS를 이렇게 작성했다.

하지만 결국 무언가 이상하다고 느꼈다.

"관심사 분리"를 했지만, 여전히 CSS와 HTML 간에는 명확한 커플링이 존재했다. 대부분의 경우 CSS는 마크업을 미러링하는 것처럼 보였다. 중첩된 CSS 셀렉터가 내 HTML 구조를 그대로 반영하고 있었기 때문이다.

**내 마크업은 스타일 결정과 연관이 없었지만, 내 CSS는 마크업 구조와 굉장히 밀접한 연관을 가지고 있었다.**

내 관심사는 전혀 분리 되지 않았던 것일지도 모른다.

### 페이즈 2: 구조로부터 스타일을 분리하기

이런 커플링 문제에 대한 여러 솔루션을 살펴보니 마크업에 더 많은 클래스를 추가하라는 권유들이 많았다. 이 클래스들은 마크업에 직접적으로 쓰여진다. 셀렉터의 특정성(specificity)를 낮게 유지하고 CSS가 특정한 DOM 구조에 덜 의존하게끔 말이다.

이런 생각 중 하나로 가장 잘 알려진 방식은 [BEM(Block Element Modifier)](http://getbem.com/introduction/)이었다.

BEM 같은 방식으로 작성하면 위의 저자 소개는 다음과 같을 것이다.

```HTML
<div class="author-bio">
  <img class="author-bio__image" src="https://cdn-images-1.medium.com/max/1600/0*o3c1g40EXj65Fq9k." alt="">
  <div class="author-bio__content">
    <h2 class="author-bio__name">Adam Wathan</h2>
    <p class="author-bio__body">
      Adam is a rad dude who likes TDD, Active Record, and garlic bread with cheese. He also hosts a decent podcast and has never had a really great haircut.
    </p>
  </div>
</div>
```

그리고 CSS는 다음과 같을 것이다.

```CSS
.author-bio {
  background-color: white;
  border: 1px solid hsl(0,0%,85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
}
.author-bio__image {
  display: block;
  width: 100%;
  height: auto;
}
.author-bio__content {
  padding: 1rem;
}
.author-bio__name {
  font-size: 1.25rem;
  color: rgba(0,0,0,0.8);
}
.author-bio__body {
  font-size: 1rem;
  color: rgba(0,0,0,0.75);
  line-height: 1.5;
}
```

[CodePen](https://codepen.io/adamwathan/pen/ZJepYj)

이런 방식이 크나 큰 진보라고 생각했다. 내 마크업은 여전히 "시맨틱"하고 그 어떤 스타일 결정도 포함하지 않는다. 또한 이제 CSS가 내 마크업 구조에서 떨어져 나갔다는 생각도 들었다. 그리고 보너스로 불확신한 셀렉터의 특정성 문제도 피할 수 있었다.

하지만 나는 곧 딜레마에 빠졌다.

### 비슷한 컴포넌트 다루기

사이트에 새로운 기능을 추가할 것이라고 가정해보자. 카드 레이아웃 내에서 글의 프리뷰를 보여줄 것이다.

이 프리뷰 카드에는 최상단에 꽉 찬 이미지가 있고 아래에는 패딩이 들어간 컨텐트 영역이 존재한다. 이 영역에서 타이틀은 볼드처리가 되어있고 본문 텍스트는 좀 더 작은 크기다.

이 프리뷰 카드가 앞서 만든 저자 소개와 거의 비슷할 것이라고 해보자.

![soc2](./images/soc2.png)

여전히 관심사 분리가 되어있는 상황에서 이 문제를 다루기 위한 가장 좋은 방법이 뭘까?

`.author-bio`를 프리뷰 카드에 적용할 수는 없다. 그렇게 하면 시맨틱하지 못하다. 결국, 컴포넌트만의 `.article-preview`를 만들게 된다.

아래와 같을 것이다.

```HTML
<div class="article-preview">
  <img class="article-preview__image" src="https://i.vimeocdn.com/video/585037904_1280x720.webp" alt="">
  <div class="article-preview__content">
    <h2 class="article-preview__title">Stubbing Eloquent Relations for Faster Tests</h2>
    <p class="article-preview__body">
      In this quick blog post and screencast, I share a trick I use to speed up tests that use Eloquent relationships but don't really depend on database functionality.
    </p>
  </div>
</div>
```

그치만 CSS는 어떻게 해야한단 말인가?

#### 옵션 1: 중복 스타일

단순한 접근법 중 하나는 `.author-bio` 스타일을 복사하고 그 클래스 이름을 바꾸는 것이다.

```CSS
.article-preview {
  background-color: white;
  border: 1px solid hsl(0,0%,85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
}
.article-preview__image {
  display: block;
  width: 100%;
  height: auto;
}
.article-preview__content {
  padding: 1rem;
}
.article-preview__title {
  font-size: 1.25rem;
  color: rgba(0,0,0,0.8);
}
.article-preview__body {
  font-size: 1rem;
  color: rgba(0,0,0,0.75);
  line-height: 1.5;
}
```

동작하지만 전혀 _DRY_ 하지 않다. 또한 두 컴포넌트가 미세하게 달라보이게 하는 문제를 만들어내기 너무 쉽다(패딩이 다르다든지, 폰트 색이 다르다든지). 결국 이런 문제는 디자인 간의 불일치를 유발한다.

#### 옵션 2: 저자 소개 컴포넌트를 @extend

다른 방법은 선택한 전처리기의 @extend 기능을 사용하는 것이다. 이미 `.author-bio`에 정의된 스타일을 재활용할 수 있다.

```CSS
.article-preview {
  @extend .author-bio;
}
.article-preview__image {
  @extend .author-bio__image;
}
.article-preview__content {
  @extend .author-bio__content;
}
.article-preview__title {
  @extend .author-bio__name;
}
.article-preview__body {
  @extend .author-bio__body;
}
```

[CodePen](https://codepen.io/adamwathan/pen/ZJepLq)

`@extend`를 사용하는 건 [보통 권장하지 않는다](https://csswizardry.com/2014/11/when-to-use-extend-when-to-use-a-mixin/). 그래도 일단은 문제를 제대로 해결한 것 같지 않은가?

CSS에서 중복을 제거하고 마크업은 여전히 스타일 결정에서 자유롭다.

하지만 또 하나의 옵션을 살펴보자.

#### 옵션 3: 컨텐트와 무관한 컴포넌트 작성

`.author-bio`와 `.article-preview` 컴포넌트는 "시맨틱"의 관점에서 아무 공통점이 없다. 그냥 따로따로다.

하지만 디자인적인 관점에서는 굉장히 많은 공통점이 존재하는 걸 보았다.

그러니 공통적으로 "행하는" 것을 이름으로 하여 새 컴포넌트를 만들 수도 있다. 그리고 새 컴포넌트를 두 타입의 컨텐트에 재사용한다.

`.media-card`라고 하자.

CSS는 다음과 같다.

```CSS
.media-card {
  background-color: white;
  border: 1px solid hsl(0,0%,85%);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  overflow: hidden;
}
.media-card__image {
  display: block;
  width: 100%;
  height: auto;
}
.media-card__content {
  padding: 1rem;
}
.media-card__title {
  font-size: 1.25rem;
  color: rgba(0,0,0,0.8);
}
.media-card__body {
  font-size: 1rem;
  color: rgba(0,0,0,0.75);
  line-height: 1.5;
}
```

저자 소개를 위한 마크업은 다음과 같다.

```HTML
<div class="media-card">
  <img class="media-card__image" src="https://cdn-images-1.medium.com/max/1600/0*o3c1g40EXj65Fq9k." alt="">
  <div class="media-card__content">
    <h2 class="media-card__title">Adam Wathan</h2>
    <p class="media-card__body">
      Adam is a rad dude who likes TDD, Active Record, and garlic bread with cheese. He also hosts a decent podcast and has never had a really great haircut.
    </p>
  </div>
</div>
```

글 프리뷰는 다음과 같다.

```HTML
<div class="media-card">
  <img class="media-card__image" src="https://i.vimeocdn.com/video/585037904_1280x720.webp" alt="">
  <div class="media-card__content">
    <h2 class="media-card__title">Stubbing Eloquent Relations for Faster Tests</h2>
    <p class="media-card__body">
      In this quick blog post and screencast, I share a trick I use to speed up tests that use Eloquent relationships but don't really depend on database functionality.
    </p>
  </div>
</div>
```

이 방식은 CSS의 중복을 없애준다. 하지만, "관심사를 혼합" 시키고 있지는 않은가?

마크업은 우리가 두 개의 컨텐트를 미디어 카드로 스타일링하길 원한다는 사실을 알아버렸다. 프리뷰 카드의 외형을 바꾸지 않고 저자 소개 카드의 외형만 바꾸고 싶다면 어떻게 할까?

이전에는 그냥 스타일 시트를 열고 두 컴포넌트에 새 스타일을 고르는 것으로 족했다. 하지만 이제는 HTML을 수정해야 한다! 맙소사!

잠시동안 다른 면을 생각해보자.

#### 같은 스타일이 필요한 새 컨텐트를 추가할 거라면 어떡해야하지?

"시맨틱"한 접근법으로 새 HTML을 작성한다, 컨텐트에 특정된 클래스 이름을 스타일 연결고리로 추가한다, 스타일 시트를 연다, 새 컨텐트에 대한 새CSS 컴포넌트를 만든다, 공통 스타일을 적용한다, 복붙하든지, `@extend`를 쓰든지, 믹스인을 쓰든지 말이다.

컨텐트와 무관한 접근법으로 `.media-card` 클래스를 작성한다. 새 HTML만 작성하면 된다. 스타일 시트를 열어볼 필요조차 없다.

우리가 실제로 "관심사를 혼합"한다면 여기저기를 바꿔야하는 것 아닌가?

### "관심사 분리"는 허상이다

"관심사 분리"의 관점에서 HTML과 CSS의 관계를 생각할 때는 흑백 밖에 없다.

관심사를 분리하면 좋은 것이고 아니면 나쁜 것이다.

HTML와 CSS를 생각할 때 옳은 방식이 아니다.

대신에 **의존성 방향을 생각해야한다**.

HTML과 CSS를 작성할 수 있는 두 가지 방식이 있다.

1. ~~ "관심사 분리" ~~

**CSS는 HTML에 의존한다.**

`.author-bio`처럼 컨텐트에 기반하여 클래스의 이름을 짓는 건 CSS가 HTML에 의존한다는 것이다.

HTML은 CSS를 의존하지 않는다. 외형을 어떻게 만들든, 그저 HTML은 자신이 컨트롤하는 `.author-bio`같은 연결고리를 노출하고 있을 뿐이다.

반면 CSS는 HTML을 의존한다. CSS는 HTML이 어떤 클래스를 노출하기로 결정했는지 알 필요가 있다. 그리고 그 클래스로 HTML을 스타일링 해야한다.

이 모델에서 HTML은 리스타일링이 가능하지만 CSS는 재사용이 불가능하다.

2. ~~ "관심사 혼합" ~~

**HTML은 CSS에 의존한다.**

`.media-card`처럼 UI 내에서 패턴이 반복되는 걸 보고 컨텐트에 무관한 방식으로 클래스 이름을 짓는 건 HTML이 CSS에 의존한다는 것이다.

CSS는 HTML을 의존하지 않는다. 컨텐트가 무엇이든 신경 쓰지 않는다. 그냥 마크업에 적용할 수 있는 것들을 제공할 뿐이다.

HTML은 CSS에 의존한다. CSS에 의해 제공되는 클래스들을 사용한다. 또한 원하는 디자인을 달성하기 위해 어떤 클래스가 존재하는지 알아야한다.

이 모델에서 CSS는 재사용이 가능하지만 HTML은 리스타일링이 불가능하다.

CSS Zen Garden은 첫번째 방식을 사용했다. [Bootstrap](https://getbootstrap.com/)과 [Bulma](https://bulma.io/)는 두번째 방식을 사용했다.

두 방법 중 "잘못"된 건 없다. 그저 그 상황에 무엇이 더 중요한지에 따른 결정일 뿐이다.

작업중인 프로젝트에서 리스타일링 가능한 HTML과 재사용 가능한 CSS 중 뭐가 중요한가?

#### 재사용성을 선택

Nicolas Gallagher의 [HTML 시맨틱스와 프론트 엔드 아키텍처에 관해](https://nicolasgallagher.com/about-html-semantics-front-end-architecture/)서가 나의 전환점이었다.

그의 주장을 되풀이할 생각은 없다. 그 블로그 포스트가 내게 재사용이 가능한 CSS에 초점을 맞추는 게 내가 작업하는 프로젝트에 있어 올바른 선택이라는 확신을 주었다.

### 페이즈 3: 컨텐트와 무관한 CSS 컴포넌트

이제 내 목표는 컨텐트 때문에 클래스를 만드는 걸 완전히 피하는 것이다. 대신 모든 클래스 이름을 최대한 재사용할 수 있게 짓는 것이다.

이런 클래스들의 이름은 다음과 같을 것이다.

- `.card`
- `.btn`, `.btn--primary`, `.btn--secondary`
- `.badge`
- `.card-list`, `.card-list-item`
- `.img--round`
- `.modal-form`, `.modal-form-section`

재사용 가능한 클래스를 만드는데 초점을 맞추면서 깨달은 게 있다.

**컴포넌트가 많을수록, 컴포넌트가 더 구체적일수록, 재사용하기 힘들다.**

직관적인 예시를 살펴보자.

폼을 만든다고 가정해보자. 여러 폼 섹션이 있고 아래에는 제출 버튼이 있다.

모든 폼 컨텐트들을 `.stacked-form` 컴포넌트의 일부로써 생각했었다면 아마 제출 버튼의 클래스는 `.stacked-form__button`이 될 것이다.

```HTML
<form class="stacked-form" action="#">
  <div class="stacked-form__section">
    <!-- ... -->
  </div>
  <div class="stacked-form__section">
    <!-- ... -->
  </div>
  <div class="stacked-form__section">
    <button class="stacked-form__button">Submit</button>
  </div>
</form>
```

하지만 아마도 사이트에는 폼의 일부가 아닌 다른 버튼이 존재할 것이다. 그 버튼도 같은 방식으로 스타일링을 해야할 지도 모른다.

그 버튼에 `.stacked-form__button` 클래스를 사용하는 건 이치에 맞지 않는다. stacked-form의 일부가 아니니까.

두 버튼 모두 각 페이지에서 주요한 동작을 담당한다. 그러니까 `.stacked-form__` 접미사를 완전히 떼고, 공통점을 따서 `.btn--primary`라고 부르면 어떨까?

```HTML
  <form class="stacked-form" action="#">
    <!-- ... -->
    <div class="stacked-form__section">
      <button class="btn btn--primary">Submit</button>
    </div>
  </form>
```

이제 이 stacked-form이 플로팅 카드 안에 있는 것처럼 보이길 원한다고 가정해보자.

한가지 방법은 추가적인 클래스(modifier)를 만들고 폼에 적용하는 것이다. 예시에서는 `.stacked-form--card`가 될 것이다.

```HTML
<form class="stacked-form stacked-form--card" action="#">
    <!-- ... -->
  </form>
```

그런데 이미 `.card` 클래스가 있다면 그것을 쓰는 새 UI와 stacked-form을 *구성*하면 된다.

```HTML
<div class="card">
    <form class="stacked-form" action="#">
      <!-- ... -->
    </form>
</div>
```

이 방법으로 이제 `.card`는 불특정한 컨텐트를 담을 수 있고 뭔가를 강제하지도 않는다. `.stacked-form`은 그 어떤 컨테이너 안에도 담길 수 있다.

이렇게 더 재사용성이 좋은 컴포넌트를 만들 수 있고 새 CSS를 더 작성하지 않아도 된다.

### Composition over subcomponents
