## 정리를 어떻게 할지 모르겠으면 응집도를 높이는 순서대로 정리하라

> 응집도를 높이는 순서로 정리하면, 코드를 더 쉽게 변경할 수 있습니다. <br/> 응집도가 좋아지면 결합도 역시 덩달아 좋아집니다.

결합도와 응집도에 대한 개념을 명확히 정리하고, React 컴포넌트 설계에 적용하는 사례를 아래와 같이 기술한다.

---

### 결합도

- **결합도 (Coupling):**  
  서로 다른 모듈이나 컴포넌트 간의 의존성 정도를 나타냄.
  - **낮은 결합도:** 모듈 간 최소한의 인터페이스만 공유하며 독립적으로 동작함.
  - **높은 결합도:** 모듈들이 구체적인 구현에 의존해 긴밀하게 연결됨.

---

### 응집도

- **응집도 (Cohesion):**  
  모듈이나 컴포넌트 내부의 구성 요소들이 단일 역할이나 기능에 집중된 정도를 나타냄.
  - **높은 응집도:** 내부 요소들이 밀접하게 관련되어 한 가지 책임에 집중함.
  - **낮은 응집도:** 여러 역할이나 기능이 혼재되어 내부 로직이 분산됨.

---

### 결합도 낮추고 응집도 높이기. 그래서 그거 어떻게 하는건데?

React 컴포넌트 설계에서 결합도는 낮고 응집도는 높은 전형적인 패턴이 있다. 아래 예시는 그중 하나인 `Headless` 패턴을 적용하여 상태 관리와 UI 렌더링을 분리함으로써 결합도를 낮추고 응집도를 높이는 예제이다.

```jsx
// UI가 없는, 동작만 담당하는 headless 컴포넌트.
function HeadlessDropdown({ children }) {
  const [open, setOpen] = useState(false);
  const toggleDropdown = () => setOpen((prev) => !prev);
  return children({ open, toggleDropdown });
}
```

```jsx
// 드랍다운 UI를 렌더링하는 컴포넌트.
// 드랍다운 상태와 제어 인터페이스를 외부에서 주입받음.
function MyDropdownComponent({ open, toggleDropdown }) {
  return (
    <div>
      <button onClick={toggleDropdown}>Toggle Dropdown</button>
      {open && (
        <ul>
          <li>Option 1</li>
          <li>Option 2</li>
          <li>Option 3</li>
        </ul>
      )}
    </div>
  );
}
```

```jsx
function App() {
  return (
    <>
      <p>아래를 눌러서 드랍다운 발동시키기.</p>
      <HeadlessDropdown>
        {({ open, toggleDropdown }) => (
          <MyDropdownComponent open={open} toggleDropdown={toggleDropdown} />
        )}
      </HeadlessDropdown>
    </>
  );
}
```

#### 그냥 MyDropdownComponet에 다 때려박으면 되는거 아냐?

- 새로운 Dropdown 컴포넌트를 도입할 때마다 내부 로직이 함께 포함되어 재구성이 필요해지므로, 유지보수 및 확장성이 저해된다.

#### 그냥 headless부분을 훅으로 뺴면 되는거 아니야?

- 훅만 사용하는 경우:
  - 상태 관리 및 토글 기능을 별도의 훅(useDropdown)으로 분리하여 재사용성을 향상시킬 수 있음.
  - 단, 해당 훅은 UI 컴포넌트에 직접 결합되므로 UI와 일정 부분 의존성을 가지게 됨.
  - 코드를 가까이 둘 수 없음.

* Headless UI 패턴 사용 시:
  - 내부 로직(상태 관리 등)은 HeadlessDropdown과 같이 별도 컴포넌트나 렌더 프로퍼티 패턴으로 캡슐화되고,
  - UI 렌더링은 MyDropdownComponent에서 독립적으로 정의되어, 로직과 UI가 명확히 분리됨.

## 결합도 제거의 환상에 관하여

> 제 믿음 중에서, 증명하거나 적절하게 설명할 수 없는 것이 하나 있습니다. <br/> 한 종류의 코드 변경에 대한 결합도를 줄일수록 다른 종류의 코드 변경에 대한 결합도가 커진다는 것입니다.<br/>이것이 의미하는 실질적인 의미는 (여러분의 직관과 일치한다면) 모든 결합을 다 색풀하듯 없애려고 애쓰지 말아야 한다는 것입니다.<br/>그렇게 해서 만들어진 결합도는 그만한 가치가 없습니다.

모든 코드를 무조건 분리하고 인터페이스를 “암흑영역”처럼 구성하지 말라는 소리로 이해했다.

### 분리시킨다고 무조건 좋을까?

다음 코드는 내부에서 여러 `useEffect`와 상태 관리 로직이 혼재되어 있는 예시다:

```jsx
const Component = (props) => {
  const [awesomeState, setAwesomeState] = useState(null);

  useEffect(() => {
    console.log("props.some changed:", someProp);
  }, [someProp]);

  useEffect(() => {
    console.log("awesomeState changed:", awesomeState);
  }, [awesomeState]);

  const dependencies = awesomeState;
  useEffect(() => {
    console.log("dependencies changed:", dependencies);
  }, [dependencies]);

  const something = someProp;
  useEffect(() => {
    console.log("something changed:", something);
  }, [something]);

  return { awesomeState, setAwesomeState };

  const handleClick = useCallback(() => {
    // do something cool
  }, [setAwesomeState]);

  return <button onClick={handleClick}>do something awesome</button>;
};
```

음.. `useEffect`랑 `useState` 꼴보기 싫으니까 아래처럼 바궈보자.

```jsx
const Component = (props) => {
  const { onClick } = useAwesomeHook(props);

  return <button onClick={onClick}>do something awesome</button>;
};
```

이렇게 한다고해서 이게 꼭 결합도가 낮고 유지보수가 좋은 코드라고 할 수 있을까?

- `Component` 자체는 간결해졌지만 `useAwesomeHook` 안에 무시무시한 폭탄을 안게 되었다.
  - 실제 동작과 상태 관리, 사이드 이펙트는 모두 useAwesomeHook 내부에 숨겨져 있어서 오히려 각 `useEffect`들 간의 결합을 한눈에 파악하기 힘들어졌다.
- 문제는 책임 분리의 불명확성:
  - 단순히 UI 코드와 로직을 분리하는 것만으로는 결합도가 낮고 유지보수가 좋은 구조라고 할 수 없다.
  - 핵심은 내부 로직을 명확하게 모듈화하고, 각 기능별 책임을 명시적으로 분리하여, 개별 변경이 전체에 미치는 영향을 최소화 하도록 해야한다.
