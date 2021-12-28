## 가상 DOM은 순전히 오버헤드다

가상 DOM은 빠르다`라는 미신을 끌어내려 봅시다

---

https://svelte.dev/blog/virtual-dom-is-pure-overhead

[Rich Harris](https://twitter.com/Rich_Harris), 2018년 12월 27일

지난 몇 년간 js 프레임워크를 사용해왔다면 `가상 DOM은 빠르다`라는 문구를 들어봤을 겁니다. 흔히 가상 DOM이 실제 DOM보다 빠르다고 말해지는 그런 것들 말이죠. 정말이지 놀랍도록 끈질긴 밈이라고 할 수 있습니다. 사람들이 어떻게 Svelte가 가상 DOM을 쓰지 않고 빠를 수 있냐고 묻는 걸 보면 말이죠.

자세히 살펴봅시다.

### 가상 DOM이 뭐죠?

많은 프레임워크에서 render() 함수를 만들어냄으로써 앱을 만듭니다. [React](https://reactjs.org/)가 그렇죠.

```js
function HelloMessage(props) {
  return <div className="greeting">Hello {props.name}</div>;
}
```

JSX 없이도 똑같은 일을 할 수 있습니다.

```js
function HelloMessage(props) {
  return React.createElement(
    "div",
    { className: "greeting" },
    "Hello ",
    props.name
  );
}
```

...하지만 결과는 같죠. 페이지의 모습을 표현하는 오브젝트가 있습니다. 그 오브젝트가 가상 DOM이죠. 앱의 상태가 변할 때마다(가령, `name` prop이 바뀔 때) 새로운 가상 DOM이 만들어집니다. 프레임워크 일은 새로운 가상 DOM과 오래된 가상 DOM을 조화시키는(reconcile) 겁니다. 실제 DOM에 어떤 변화가 필요하고 적용해야하는지 이해하기 위한 과정입니다.

### 어떻게 이런 밈이 퍼졌을까?

가상 DOM의 성능에 대한 몰이해는 React의 시작으로 거슬러 올라갑니다. 이전 React 코어 팀 일원이었던 Pete Hunt가 2013년 세미나에서 발표한 [최적의 사례를 다시 생각하기](https://www.youtube.com/watch?v=x7cQ3mrcKaY&ab_channel=JSConf)에서 다음을 알 수 있습니다.

> 굉장히 빠릅니다, 일반적으로 대부분 DOM 연산이 느리기 때문이죠. DOM에 대해 많은 퍼포먼스 작업들이 있었지만 대부분 DOM 연산은 프레임을 떨어트립니다.

![Rethinking Best Practices](https://svelte.dev/media/rethinking-best-practices.jpg)

2013년 JSConfEU의 [최적의 사례를 다시 생각하기](https://www.youtube.com/watch?v=x7cQ3mrcKaY&ab_channel=JSConf) 스크린샷

그런데 잠깐만요! 가상 DOM 연산이란 건 사실 실제 DOM에 연산이 추가된 것입니다. 가상 DOM이 더 빠를 수 있는 방법은 덜 효율적인 프레임워크(2013년에도 많은 게 있었습니다)와 비교하거나 허수아비랑 논쟁하는 방법 밖에 없습니다. 대안은 누구도 하지 않는 짓을 하는 겁니다.

```js
onEveryStateChange(() => {
  document.body.innerHTML = renderMyApp();
});
```

Pete은 후에 이 부분에 대해 명확히 말했습니다.

> React는 마법이 아닙니다. C를 어셈블러와 사용해 C 컴파일러를 이길 수 있는 것처럼 원한다면 원래 DOM 연산과 DOM API 호출을 사용해 React를 이길 수 있습니다. 하지만 C, Java, 또는 js를 씀으로써 성능을 크게 향상시킬 수 있습니다. 왜냐구요? 어떤 플랫폼을 쓰는지 걱정할 필요가 없기 때문이죠. React를 쓰면 퍼포먼스에 대해 생각할 필요 없이 앱을 만들 수 있고 기본적으로 빠릅니다.

하지만 문제가 되는 부분은 아닙니다.

### 그래서... 가상 DOM이 느린가요?

정확히는 아닙니다. `가상 DOM이 보통 충분히 빠르다` 정도지요. 하지만 주의해야할 점이 있습니다.

원래 React가 약속한 건 퍼포먼스 걱정 없이 단일 상태가 변할 때마다 전체 앱이 다시 렌더링 될 수 있다는 것이었죠. 실제로, 그렇지 않다고 나는 생각합니다. 그랬다면 `shouldComponentUpdate`(안전하게 React에게 컴포넌트를 업데이트 하지 않게 하는 방법)같은 최적화가 있을 필요가 없겠죠.

`shouldComponentUpdate`를 쓰더라도 전체 앱의 가상 DOM을 업데이트하는 건 많은 일을 요구합니다. 잠시 돌아가보면 React 팀은 React Fiber라는 걸 도입했습니다. 이 건 업데이트를 작은 부분으로 나누게끔 해주는 것이었죠. 이말은 즉, 업데이트는 긴 시간동안 메인 스레드를 막지 말아야한다는 것입니다. 그런데 이런 동작이 업데이트에 걸리는 전체 시간을 줄여주진 않습니다.

### 이런 오버헤드가 왜 발생하나요?

거의 명확하게 [비교(diffing)은 공짜가 아닙니다](https://twitter.com/pcwalton/status/1015694528857047040). 이전 스냅샷과 처음 비교할 새 가상 DOM이 없이 실제 DOM에 변화를 적용할 수 없습니다. 앞서 보여준 `HelloMessage`예시에서 `name` prop이 'world'에서 'everybody'로 바뀐다고 해봅시다.

1. 단일 엘레멘트를 포함한 두 스냅샷이 존재한다. 두 스냅샷은 `<div>`이고 같은 DOM 노드를 유지한다는 걸 의미한다.
2. 오래된 `<div>`와 새 `<div>`의 모든 어트리뷰트를 순회하며 바뀌거나 더해지거나, 또는 사라진 게 있는지 찾는다. 이 경우, 예시처럼 `"greeting"`이라는 값을 지닌 `classname`이라는 하나의 어트리뷰트를 가지는 걸 알 수 있다.
3. 자식 엘레멘트로 내려간다. 텍스트가 변했음을 알 수 있다. 그러니 실제 DOM을 업데이트 해야한다.

이 세 단계에서 실제로는 세 번째 단계만 가치가 있습니다. 대부분의 경우처럼 앱의 기본 구조가 변하지 않았기 때문이죠. 만약 다 건너뛰고 바로 세번째 단계로 갔으면 더 효율적일 겁니다.

```js
if (changed.name) {
  text.data = name;
}
```

(이 코드는 Svelte가 생성하는 코드와 거의 동일합니다. 전통적인 UI 프레임워크와 다르게 Svelte는 컴파일러로써 앱이 어떻게 변하지는 빌드 타임에 알고 있습니다. 런타임까지 기다리지 않습니다.)

### 비교(diffing)뿐만이 아니다

React나 다른 가상 DOM 프레임워크의 비교 알고리즘은 빠릅니다. 자신있게 말하건대 컴포넌트 그 자체가 더 큰 오버헤드입니다. 이런 코드는 작성하지 않았을 겁니다...

```js
function StrawManComponent(props) {
  const value = expensivelyCalculateValue(props.foo);

  return <p>the value is {value}</p>;
}
```

`props.foo`가 변하든말든 매 업데이트에서 생각 없이 `value`를 재계산하고 있기때문이죠. 이렇게 여러모로 불필요한 계산과 할당을 하는 건 더 정상적인 것처럼 보이는 상황에서 흔합니다.

```js
function MoreRealisticComponent(props) {
  const [selected, setSelected] = useState(null);

  return (
    <div>
      <p>Selected {selected ? selected.name : "nothing"}</p>

      <ul>
        {props.items.map((item) => (
          <li>
            <button onClick={() => setSelected(item)}>{item.name}</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

여기에서는 `props.items`가 변하든말든 모든 상태가 변할 때마다 각자가 인라인 이벤트 핸들러를 가진 가상 `<li>` 엘레멘트의 새 배열을 만들어내고 있습니다. 퍼포먼스에 광적으로 집착하지 않는다면 굳이 최적화하지 않을 겁니다. 문제는 없습니다. 충분히 빠르니까요. 그런데 뭐가 더 빠를지 아시나요? 그러지 마세요.

이렇게 그냥 불필요한 일(사소한 것이라도)을 하는 것에 대한 위험성은 앱을 실제로 점차적으로 죽이는 겁니다. 최적화를 해야할 시간이 오면 명확한 병목점도 잡지 못하게 됩니다.

Svelte는 이런 상황을 막기 위해 고안 되었습니다.

> [React Hook](https://reactjs.org/docs/hooks-intro.html)은 [예측 가능한 결과](https://twitter.com/thekitze/status/1078582382201131008)로 더욱더 불필요한 작업을 하게 합니다

### 그럼 왜 가상 DOM을 써야하나요?

가상 DOM이 기능이 아니란 걸 이해하는 게 중요합니다. 결국 수단에 불과하단 거죠. 결국엔 선언적이고, 상태에 의해 주도 되는 UI 개발을 위한 수단에 불과합니다. 가상 DOM은 충분히 좋은 퍼포먼스로 상태에 대한 생각 없이 앱을 만들게끔 하기 때문에 가치가 있습니다. 그덕에 버그가 덜한 코드를 만들 수 있는 거고 지루한 작업 대신 더 창조적인 일에 힘 쓸 수 있는 거죠.

하지만 가상 DOM 없이 동일한 프로그래밍 모델을 사용할 수 있습니다. 거기에 Svelte가 있는 거죠.
