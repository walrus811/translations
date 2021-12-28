## Svelte 3 : 반응성(reactivity)에 대해 다시 생각하기

마침내

---

https://svelte.dev/blog/svelte-3-rethinking-reactivity

[Rich Harris](https://twitter.com/Rich_Harris), 2019년 4월 22일

여러달이 지난 후에 마침내 우리는 Svelte 3의 안정화 릴리즈를 발표합니다. 이 거대한 릴리즈는 Svelte 커뮤니티의 수많은 사람들과 그들으 수백 시간의 작업을 의미합니다. 물론 모든 단계에서 설계에 대한 도움이 된 베타 테스터들의 귀중한 피드백도 포함해서 말이죠.

정말 좋아할 거라고 생각합니다.

### Svelte가 뭐죠?

Svelte는 React나 Vue 같은 컴포넌트 프레임워크지만 중요한 차이점이 있습니다. 전통적인 프레임워크들에선 선언적이고 상태에 의해 변하는 코드를 작성할 수 있습니다. 하지만 문제가 있죠, 브라우저가 그 선언적인 구조를 DOM 연산으로 바꾸기 위해 별도의 작업을 해야한다는 겁니다. [가상 DOM 비교](https://svelte.dev/blog/virtual-dom-is-pure-overhead)(번역 - [가상 DOM 비교](./virtual-dom-is-pure-overhead.md))
