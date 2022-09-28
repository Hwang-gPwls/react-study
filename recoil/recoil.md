# Recoil

## 작고 React 스러운, 데이터 흐름 그래프, 교차하는 앱 관찰

## Recoil은 어떻게 탄생하게 되었을까?

Recoil 공식문서에서는 이렇게 답하고 있다.
"호횐성 및 단순함을 이유로 외부의 글로벌 상태 관리 라이브러리 보다는 React 자체에 내장된 상태 관리 기능을 사용하는 것이 좋다. 그러나 React는 다음과 같은 한계가 있다."

- 컴포넌트의 상태는 공통된 상위요소까지 끌어올려야만 공유될 수 있으며, 이 과정에서 거대한 트리가 다시 렌더링되는 효과를 야기하기도 한다.

  => 여러 하위 컴포넌트들에서 같은 상태를 사용하기 위해서는 하나의 루트 컴포넌트가 필요하게 되는데, 이 루트 컴포넌트가 렌더링 하게 되면 하위 컴포넌트가 렌더링 되는 비효율성이 발생한다.

- context는 단일 값만 저장할 수 있으며, 자체 소비자(consumer)를 가지는 여러 값들의 집합을 담을 수는 없다.

- 이 두 가지 특성이 트리의 최상단(state가 존재하는 곳)부터 트리의 말단(state가 사용되는 곳)까지의 코드 분할을 어렵게 한다.

## Recoil의 주요 개념

Recoil을 사용하면 atoms(공유 상태)에서 selectors(순수 함수)를 거쳐 React 컴포넌트로 내려가는 data-flow graph를 만들 수 있다. Atoms는 컴포넌트가 구독할 수 있는 상태의 단위다. Selectors는 atoms 상태값을 동기 또는 비동기 방식을 통해 변환한다.

=> global state를 어플리케이션의 분리된 공간에서 관리하는 것

=> 오직 state가 필요한 component만 그 state를 가진다.

### Atoms

- Atoms는 상태의 단위이며, 업데이트와 구독이 가능하다.
- <b>atom이 업데이트되면 각각의 구독된 컴포넌트는 새로운 값을 반영하여 다시 렌더링 된다.</b>
- Atoms는 런타임에서 생성될 수도 있다.
- Atoms는 React의 로컬 컴포넌트의 상태 대신 사용할 수 있다.
- 동일한 atom이 여러 컴포넌트에서 사용되는 경우 모든 컴포넌트는 상태를 공유한다.

생성

```javascript
const fontSizeState = atom({
  key: "fontSizeState", //고유 key는 전역적으로 고유하도록 한다.
  default: 14,
});
```

사용

버튼을 클릭하면 글꼴 크기 1증가.

fontSizeState atom을 사용하는 다른 컴포넌트의 글꼴 크기도 같이 변경.

```javascript
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return (
    <button
      onClick={() => setFontSize((size) => size + 1)}
      style={{ fontSize }}
    >
      Click to Enlarge
    </button>
  );
}

function Text() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  return <p style={{ fontSize }}>This text will increase in size too.</p>;
}
```

### Selectors

- Selector는 atoms나 다른 selectors를 입력으로 받아들이는 순수 함수(pure function)다.

> 순수함수란
>
> - 동일한 인자가 들어갈 경우 항상 같은 값이 나와야 한다.
> - side effect가 일어나면 안된다. (함수에서 외부 변수의 값을 변경하거나 들어온 인자의 값을 변화시켜선 안된다.)

- 상위의 atoms 또는 selectors가 업데이트되면 하위의 selector 함수도 다시 실행된다.
- 컴포넌트들은 selectors를 atoms처럼 구독할 수 있으며 selectors가 변경되면 컴포넌트들도 다시 렌더링된다.
- <b>상태를 기반으로 하는 파생 데이터를 계산하는 데 사용된다. 최소한의 상태 집합만 atoms에 저장하고 다른 모든 파생되는 데이터는 selectors에 명시한 함수를 통해 효율적으로 계산함으로써 쓸모없는 상태의 보존을 방지한다.</b>
- 어떤 컴포넌트가 자신을 필요로하는지, 또 자신은 어떤 상태에 의존하는지를 추적하기 때문에 함수적인 접근방식을 매우 효율적으로 만든다.
- 컴포넌트의 관점에서 보면 selectors와 atoms는 동일한 인터페이스를 가지므로 서로 대체할 수 있다.

정의

fontSizeState를 입력으로 사용하고 형식화된 글꼴 크기 레이블을 출력으로 반환하는 순수 함수처럼 동작한다.

```javascript
const fontSizeLabelState = selector({
  key: "fontSizeLabelState",
  get: ({ get }) => {
    const fontSize = get(fontSizeState);
    const unit = "px";

    return `${fontSize}${unit}`;
  },
}); // get 속성은 계산될 함수다. 전달되는 get 인자를 통해 atoms와 다른 selectors에 접근할 수 있다.
```

다른 atoms나 selectors에 접근하면 자동으로 종속 관계가 생성되므로, 참조했던 다른 atoms나 selectors가 업데이트되면 이 함수도 다시 실행된다.

사용

```javascript
function FontButton() {
  const [fontSize, setFontSize] = useRecoilState(fontSizeState);
  const fontSizeLabel = useRecoilValue(fontSizeLabelState); // useRecoilValue()를 사용해 selector를 읽을 수 있다.

  return (
    <>
      <div>Current font size: ${fontSizeLabel}</div>

      <button onClick={setFontSize(fontSize + 1)} style={{ fontSize }}>
        Click to Enlarge
      </button>
    </>
  );
}
```

useRecoilValue()는 하나의 atom이나 selector를 인자로 받아 대응하는 값을 반환한다.
