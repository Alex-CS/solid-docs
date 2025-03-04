---
title: 자주하는 질문
description: 커뮤니티에 자주 올라오는 질문.
sort: 6
---

# 자주하는 질문

### 가상 DOM 을 사용하지 않는 JSX라구요? 베이퍼웨어인가요? 저명한 인사가 이건 불가능하다고 말하는 것을 들었습니다.

React의 업데이트 모델이 없으면 가능합니다. JSX는 Svelte나 Vue에 있는 것과 같은 템플릿 언어로, 어떤 면에서는 더 유연합니다. 임의의 자바스크립트를 삽입하는게 어려울 때도 있지만, 스프레드 연산자를 지원하는것과 별반 다르지 않습니다. 대답은 아니오: 이것은 베이퍼웨어가 아니라 가장 성능이 좋은것으로 입증된 접근 방식입니다.

진정한 이점은 확장 가능성에 있습니다. 컴파일러는 최적의 네이티브 DOM 업데이트를 제공하면서도 React와 같은 라이브러리의 모든 자유도를 가지고 있습니다. [render props](https://ko.reactjs.org/docs/render-props.html)나 고차 컴포넌트<sub>High order component</sub> 같은 기술을 사용하는 컴포넌트를 리액티브 "Hook"과 함께 쓸 수 있습니다. Solid의 제어 흐름 작동 방식이 마음에 들지 않는다면 직접 만들어 사용할 수 있습니다.

### Solid는 얼마나 성능이 좋은가요?

뭔가 하나를 꼽을 수 있다면 좋겠지만, 실제로는 몇 가지 중요한 디자인상의 결정의 조합입니다:

1. 명시적인 Reactivity는 리액티브해야하는 항목만 추적합니다.
2. 초기 생성을 고려하여 컴파일합니다. Solid는 휴리스틱 기술을 사용하고 올바른 표현식을 결합하여 계산 횟수를 줄이면서도, 중요한 업데이트를 세분화하여 성능을 유지합니다.
3. 리액티브 표현식은 단순한 함수일 뿐입니다. 이렇게 하면 불필요한 래퍼와 동기화 오버헤드를 제거하면서 지연 props 평가를 사용해 "사라지는 컴포넌트<sub>vanishing components</sub>"를 가능하게 합니다.

이것은 현재 Solid가 경쟁에서 우위를 점할 수 있도록 하는 고유한 기술 조합입니다.

### React와 호환성이 있나요? 아니면 React로 작성한 라이브러리를 Solid에서 사용할 수 있나요?

아니오. 그리고 앞으로도 불가능할 것입니다. API는 비슷하고, 컴포넌트는 약간의 편집으로 마이그레이션이 가능할 수 있지만, 업데이트 모델은 근본적으로 다릅니다. React 컴포넌트는 계속 렌더링되므로, Hook 외부의 코드는 아주 다르게 동작합니다. 클로저와 Hook 규칙은 Solid에서 불필요할 뿐만 아니라 여기에서는 작동하지 않는 코드를 규정할 수 있습니다.

반면에 현재 구현 계획은 없지만 Vue 호환은 가능합니다.

### 왜 템플릿에서 `map`을 사용할 수 없나요? `<For>`와 `<Index>`의 차이점은 뭔가요?

정적 배열의 경우 map을 사용하는데 아무 문제가 없습니다. 하지만 Signal이나 리액티브 프로퍼티에 대해 루프를 도는 경우, `map`은 충분하지 않습니다: 배열이 어떤 이유로 변경된다면 _전체 map_ 은 다시 실행되며, 모든 노드가 다시 생성됩니다.

`<For>` 와 `<Index>`는 둘 다 이보다 더 똑똑한 루프 솔루션을 제공합니다. 각각의 렌더링된 노드는 배열의 엘리먼트에 매핑됩니다. 배열의 엘리먼트가 변경되면 매핑된 노드는 다시 렌더링됩니다.

`<Index>` 는 이 작업을 _인덱스를 사용해_ 수행하며, 각 노드는 배열의 인덱스에 연결됩니다; `<For>` 는 이 작업을 _값을 사용해_ 수행하며, 각 노드는 배열의 데이터 조각에 연결됩니다. 이 때문에 콜백에서 `<Index>`가 항목에 대한 signal을 제공하는 이유입니다. 각 항목의 인덱스는 고정된 것으로 간주되지만 해당 인덱스의 데이터는 변경될 수 있습니다. 반면에 `<For>`는 인덱스에 대한 signal을 제공합니다. 항목의 내용은 고정된 것으로 간주되지만 항목은 배열내에서 이동할 수 있습니다.

예를 들어, 배열의 두 요소가 바뀌면, `<For>`는 연결된 두 DOM 노드의 위치를 변경하고 그 과정에서 `index()` signal을 업데이트합니다. `<Index>`는 DOM 노드의 위치를 변경하지 않지만, 두 DOM 노드에 대한 `item()` signal을 업데이트하고 다시 렌더링하도록 합니다.

차이점에 대한 자세한 설명은 Ryan의 동영상에서 [이 부분](https://www.youtube.com/watch?v=YxroH_MXuhw&t=2164s)을 참고하세요.

### 왜 props나 store에서 디스트럭쳐링을 사용할 수 없나요?

props 및 store 객체를 사용하면 프로퍼티에 접근하는 경우 반응성이 추적됩니다: 리액티브 컨텍스트 내에서 `props.whatever`를 호출하면, Solid가 이 컨텍스트를 추적하고 props가 변경되면 업데이트하도록 합니다. 디스트럭쳐링을 사용하게 되면 객체에서 값을 분리하여 해당 시점의 값을 제공하게 되면서 반응성을 잃게 됩니다.

### 왜 `onChange` 이벤트 핸들러가 제 때 호출되지 않는건가요?

일부 프레임워크에서는 입력에 대한 `onChange` 이벤트가 수정되어, 키를 누를 때마다 발생합니다. 하지만 이는 `onChange` 이벤트가 [네이티브하게 동작하는 방식](https://developer.mozilla.org/ko/docs/Web/API/GlobalEventHandlers/onchange)이 아닙니다: `onChange`는 입력창에 _커밋된_ 변경을 반영하기 위해서 사용되며, 일반적으로 입력창이 포커스를 잃으면 발생합니다. 입력 값에 대한 모든 변경 사항을 핸들링하려면 `onInput`을 사용하세요.

### 클래스 컴포넌트에 대한 지원을 추가할 수 있나요? 라이프 사이클이 더 추론하기 쉽다고 생각합니다.

클래스 컴포넌트는 지원할 생각이 없습니다. Solid의 컴포넌트 라이프사이클은 리액티브 시스템의 스케줄링과 관련되어 있으며 인위적입니다. 클래스를 만들 수는 있지만, 사실상 이벤트 핸들러 이외의 코드는 render 함수를 포함해 생성자에서 실행됩니다. 이는 데이터를 덜 세분화하기 위한 변명을 위한 구문일 뿐입니다.

라이프사이클이 아니라 데이터와 동작을 함께 그룹화하세요. 이것은 수십년 동안 작동해온 리액비트의 모범 사례입니다.

### 나는 JSX를 아주 싫어합니다. 다른 템플릿 언어를 지원할 생각이 있나요? 아.. 태그가 지정된 템플릿 리터럴 / 하이퍼스크립트가 있군요. 이걸 사용해 볼까요?

하지 마세요. 우리는 Svelte가 템플릿을 사용하는 방식으로 JSX를 사용하여 최적화된 DOM 명령어를 생성합니다. 태그된 템플릿 리터널 및 하이퍼스크립트 솔루션은 그 자체로 매우 정말 훌륭하지만, 빌드를 하지 않아야하는 것과 같은 현실적인 요구사항이 없는 한 모든 측면에서 더 좋지 않습니다. 번들이 커지고, 성능이 저하되며, 수동으로 값을 래핑해야 합니다.

선택 사항이 있는 것은 좋지만, Solid의 JSX는 정말 최고의 솔루션입니다. 템플릿 DSL도 제한적이긴 하면서도 훌륭하지만, JSX는 많은 것을 무료로 제공합니다: 기존의 파서, 구문 강조 표시, prettier, 코드 완성, 그리고 TypeScript가 있습니다.
 
다른 라이브러리에서 이런 기능들에 대한 지원을 추가하고 있지만, 엄청난 노력이 필요하며 여전히 불완전하고 지속적인 유지보수에 어려움을 겪고 있습니다. 우리는 정말 현실적인 스탠스를 취하고 있습니다.

### Signal과 Store는 언제 사용하나요? 어떤 차이가 있나요?

Store는 중첩된 값을 자동으로 래핑므로 심층 데이터 구조와 모델과 같은 것에 적합합니다. 다른 대부분의 경우 Signal은 가볍고 훌륭하게 작업을 수행합니다.

우리가 이를 하나로 묶고 싶어도, 프리미티브를 프록시할 수 없습니다. 함수는 가장 간단한 인터페이스이며 모든 리액티브 표현식(상태 접근 포함)은 전송시 하나로 래핑될 수 있으므로 유니버설 API를 제공합니다. signal이나 상태에 원하는 이름을 지정할 수 있어 최소한의 기능에 머무를 수 있습니다. 최종 사용자에서 `.get()` `.set()` 입력을 강요하거나, 심지어 `.value`를 입력하게 강요하는 것은 절대로 피하고 싶은 것입니다. 적어도 전자는 간결함을 위해 alias를 지정할 수 있지만, 후자는 함수를 호출하는 매우 간결하지 않은 방법입니다.

### 왜 Vue 처럼 Solid Store에 값을 할당할 수 없는건가요? Svelte 나 MobX? 양방향 바인딩은 어떻게 하나요?

반응성은 강력한 도구이지만 동시에 위험한 도구이기도 합니다. MobX 는 이를 알고 업데이트가 발생하는 위치/시기를 제한하는 Strict 모드와 Action을 도입했습니다. 데이터의 전체 컴포넌트 트리를 다루는 Solid에서는 React에서 무언가를 배울 수 있다는 것이 분명해졌습니다. 동일한 계약을 가질 수 있는 방법을 제공한다면 실제로 immutable일 필요는 없습니다.

상태 업데이트 기능을 전달할 수 있는지 여부는 상태를 전달할지 여부를 결정하는 것보다 훨씬 더 중요합니다. 따라서 이를 분리할 수 있는 것이 중요하며, 읽기가 immutable인 경우에만 가능합니다. 또한 세부적으로 업데이트할 수 있다면 immutable 사용 비용을 지불할 필요가 없습니다. 다행히도 ImmutableJS 와 Immer 사이에는 수많은 선행 기술이 있습니다. 아이러니하게도, Solid는 mutable한 내부와 immutable한 인터페이스를 사용해 리버스 Immer로 작동합니다.

### Solid의 반응성을 단독으로 사용할 수 있나요?

물론입니다. 독립된 패키지를 익스포트하지는 않았지만, 컴파일러 없는 Solid를 설치하고 리액티브 프리미티브를 사용하는 것은 간단합니다. 세분화된 반응성의 장점 중 하나는 라이브러리에 의존하지 않는다는 것입니다. 거의 대부분의 리액티브 라이브러리가 이런 식으로 작동합니다. 이것이 [Solid](https://github.com/solidjs/solid)와 그 기초가 되는 [DOM Expressions library](https://github.com/ryansolid/dom-expressions)가 처음부터 리액티브 시스템으로부터 렌더러를 만들도록 영감을 준 것입니다. 

시도해 볼 만한 몇 가지를 나열해 보면 [Solid](https://github.com/solidjs/solid), [MobX](https://github.com/mobxjs/mobx), [Knockout](https://github.com/knockout/knockout), [Svelte](https://github.com/sveltejs/svelte), [S.js](https://github.com/adamhaile/S), [CellX](https://github.com/Riim/cellx), [Derivable](https://github.com/ds300/derivablejs), [Sinuous](https://github.com/luwes/sinuous), [Vue](https://github.com/vuejs/vue) 등이 있습니다. 예를 들어, [lit-html](https://github.com/Polymer/lit-html)처럼 렌더러에 태그를 지정하는 것보다는 리액티브 라이브러리를 만드는데 훨씬 더 많은 시간이 필요하지만 감을 잡기에는 좋은 방법입니다.

### Solid에는 Next.js 나 Material Components 같은 라이브러리가 있나요?

 현재 작업중입니다! 라이브러리 구축에 관심이 있다면, [Discord](https://discord.com/invite/solidjs)를 통해서 현재 에코시스템에 참여하거나 새로 만들 수도 있습니다.
