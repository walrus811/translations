## Svelte 3 : 반응성(reactivity)에 대해 다시 생각하기

마침내

---

https://svelte.dev/blog/svelte-3-rethinking-reactivity

[Rich Harris](https://twitter.com/Rich_Harris), 2019년 4월 22일

여러달이 지난 후에 마침내 우리는 Svelte 3의 안정화 릴리즈를 발표합니다. 이 거대한 릴리즈는 Svelte 커뮤니티의 수많은 사람들과 그들으 수백 시간의 작업을 의미합니다. 물론 모든 단계에서 설계에 대한 도움이 된 베타 테스터들의 귀중한 피드백도 포함해서 말이죠.

정말 좋아할 거라고 생각합니다.

### Svelte가 뭐죠?

Svelte는 React나 Vue 같은 컴포넌트 프레임워크지만 중요한 차이점이 있습니다. 전통적인 프레임워크들에선 선언적이고 상태에 의해 변하는 코드를 작성할 수 있습니다. 하지만 문제가 있죠, 브라우저가 그 선언적인 구조를 DOM 연산으로 바꾸기 위해 별도의 작업을 해야한다는 겁니다. [가상 DOM 비교](https://svelte.dev/blog/virtual-dom-is-pure-overhead)([번역](./virtual-dom-is-pure-overhead.md)) 같은 기술을 사용해 프레임에 부담을 주고 가비지 컬렉터를 무겁게 만듭니다.

대신 Svelte는 빌드 시간에 동작합니다. 컴포넌트를 고가용성의 선언적 코드로 만들어줍니다. 거기서는 DOM이 외과적으로(surgically) 업데이트됩니다. 즉,  훌륭한 성능을 지니는 야심찬 어플리케이션을 작성할 수 있죠.

Svelte의 첫 버전은 [가설에 대한 실험](https://svelte.dev/blog/frameworks-without-the-framework)([번역](./frameworks-without-the-framework.md))에 대한 것이었습니다. 이 때 만들어진 컴파일러는 훌륭한UX를 제공하는 견고한 코드를 만들 수 있었습니다. 두 번째는 약간의 정돈만한 작은 업그레이드였습니다.

버전 3는 중요한 오버홀입니다. 지난 5~6달간 우리는 몹시 훌륭한 개발 경험을 만드는데 집중했습니다. 이제 당신이 본 그 어떤 것보다 [엄청나게 적은 보일러플레이트](https://svelte.dev/blog/write-less-code)([번역](./frameworks-without-the-framework.md))로 컴포넌트를 작성할 수 있습니다. 완전히 새로운 [튜토리얼](https://svelte.dev/tutorial/basics)을 체험해보고 우리가 뜻하는 바를 확인해 보세요. 다른 프레임워크에 익숙하다면 놀랍도록 기뻐할 것입니다.

이를 위해 우리는 우선 현대 UI 프레임워크의 중점이 무엇인지 다시 생각해보았습니다. 바로 반응성(reactivity)였습니다.

https://youtu.be/AdNJ3fydeao
