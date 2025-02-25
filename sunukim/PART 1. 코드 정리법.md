## CHAPTER 1. 보호 구문

```js
if (조건 부정) return
if (다른 조건) return
```

- 실제로 많이 쓰는 양식인듯.
- 우리 서비스에서 특정 조건인 경우 되지 않게 하는게 많은데 조건으로 묶어서 처리하는 경우로 많이 사용했다.

```js
const checkLevels = (user) => {
  if (user.level < 3) {
    return false
  }

  if (user.id !== 'admin') {
    return false
  }

  return true
}
```

## CHAPTER 2. 안 쓰는 코드

- 안쓰는 코드는 지우자.
- 지우는게 맞다. 결국 쓰이지 않을 것이고 쓰이게 된다 하더라고 깃으로 찾아서 쓰면 된다.

## CHAPTER 3. 대칭으로 맞추기

```js

// 두 가지 패턴이 공존하는 코드
const checkLevel = (user) => {
  if (user.level < 3) {
    return false
  }

  return user.id !== 'admin' ? false : true
}

```

- 코드를 쓸 때 한가지 패턴으로 맞추자
- 두 가지 이상의 패턴을 섞어쓰면 혼란스럽다.


## CHAPTER 4. 새로운 인터페이스로 기존 루틴 부르기

- 기존 루틴을 부르는 인터페이스를 새로 만들자.
- 기존 인터페이스가 어렵거나, 복잡하거나, 혼란스럽거나, 지루해지곤 할 때 새로 만들자
- 거꾸로 코딩하기, 테스트 우선 코딩, 도우미 설계도 비슷한 느낌을 받는다.

### CHAPTER 5. 읽는 순서

- 읽기 좋은 순서로 정렬하면, 그 순서대로 코드를 만날 수 있다.
- 코드를 읽을 때는 독자의 입장이 되어보자.

## CHAPTER 6. 응집도를 높이는 배치

- 변경해야 할 동작을 찾았더니 여러 곳에 흩어져 있다.
- 코드의 순서를 바꿔서 변경할 요소들을 가까이 두면 된다.
- 두 파일에 결합도 있으면 같은 디렉토리에 넣자.
- 결합도를 제거할 수 있다면 제거 하는게 좋다.

```
결합도 제거 비용 + 변경 비용 < 결합도에 따른 비용 + 변경 비용
```

- 물론 결합도 제거가 어려운 경우도 있다.
- 코드가 구멍 난 듯 흩어진 채로 두고 동작변경만 하지 말고 응집도를 높이는 순서로 정리하자.
- 자연스레 결합도 제거가 될 수도 있다.

## CHAPTER 7. 선언과 초기화를 함께 옮기기

- 선언과 그 변수를 쓰는 로직을 가까이에 두자.
```js

const foo = () => {
    const level = user.level
    if (level < 3) {
        ....
    }

    const id = user.id
    if (id !== 'admin') {
        ....
    }
}
```

## CHAPTER 8. 설명하는 변수

- 복잡한 코드를 이해했다면, 일부 표현식을 추출한 후 표현식이 드러나도록 변수 이름을 만들어 할당하자.

```js
const foo = () => {
    if (user.level < 3 && user.id !== 'admin') {
        ....
    }

    if (user.createDate < '2024-01-01' && user.level > 5) {
        ....
    }
}
```

```js
const foo = () => {
    const isAdmin = user.level < 3 && user.id !== 'admin'
    if (isAdmin) {
        ....
    }

    const isNewUser = user.createDate < '2024-01-01' && user.level > 5
    if (isNewUser) {
        ....
    }
}
```

- 다음번에 수정할 때 하나만 고치면 된다.
- 언제나 그렇듯이, 코드 정리에 대한 커밋과 동작 변경에 대한 커밋은 분리해야 한다.

## CHAPTER 9. 설명하는 상수

- 상징적인 상수를 만들자. 단순 숫자나 문자로 표현된 것을 상징적인 상수로 바꾸자.

```js
if (response.status === 200) {
    ....
}
```

```js
const STATUS_OK = 200
const STATUS_NOT_FOUND = 404
```

## CHAPTER 10. 명시적인 매개변수

- 읽기 어려운 매개변수를 만들지 말자.
- 매개변수를 명시적으로 작성하여 코드를 읽기 쉽게 하자.

```js
const params = {
    a : 1,
    b : 2,
    c : 3,
}

const foo = (params) => {
    if (params.a === 1) {
        ....
    }
}

---------------------------

const foo = (a, b, c) => {
    if (a === 1) {
        ....
    }
}
```

- 근데 javascript는 구조분해할당이 있기 때문에, 오히려 객제로 묶는게 편하다.

```js
const params = {
    a : 1,
    b : 2,
    c : 3,
}

const foo = ({a, b, c}) => {
    if (a === 1) {
        ....
    }
}

```

## CHAPTER 11. 비슷한 코드끼리

- 모든 코드 정리 중에 가장 단순한 정리법이다.
- 코드가 구분이 될 때는 두 부분 사이에 빈 줄을 넣어 분리하자.

## CHAPTER 12. 도우미 추출

- 코드를 보다가 루틴 속 코드 중에서, 목적이 분명하고 나머지 코드와는 상호작용이 적은 코드블록을 만날 때가 있다.
- 이런 코드블록을 추려내고 도우미 함수로 만들자.
- 나는 util 함수로 이해했다.

```js
const foo = () => {
    ...그대로 두는 코드
    ...바꾸려는 코드
    ...그대로 두는 코드
}
```

```js
const helper = () => {
    ...바꾸려는 코드
}

const foo = () => {
    ...그대로 두는 코드
    helper()
    ...그대로 두는 코드
}
```

## CHAPTER 13. 하나의 더미

- 흩어진 코드를 전체적으로 이해하기 어렵다.
- 필요한 만큼의 코드를 하나의 더미처럼 느껴질 때까지 모으자.
- 그리고 나서 깔끔하게 정리하자.

- 그 뒤 어려운 부분을 추출하자.
- 이해가 되면 순서가 보이고 머리에 정리가 되고 코드 정리가 된다.

## CHAPTER 14. 설명하는 주석

- 코드를 읽다가 어떻게 돌아가는지 이해했다면, 주석을 작성하자.
- 내가 이해하는 데 시간이 걸렸다면 다른 사람도 그렇게 될 가능성이 높다.
- 설명하는 주석을 작성하여 다른 사람들이 고생하지 않도록 하자.
- 결국엔 정리를 해야겠지만, 정리 하기 어려울 때가 있기 때문에 주석을 작성하지 않는 것보다 낫다.

## CHAPTER 15. 불필요한 주석 지우기

- 코드만으로 내용을 모두 이해할 수 있다면 주석을 삭제하자.
- 주석이 코드와 완전한 중복이라면 삭제하자.
