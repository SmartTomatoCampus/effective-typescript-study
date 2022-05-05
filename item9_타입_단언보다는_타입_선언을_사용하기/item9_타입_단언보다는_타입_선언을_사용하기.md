# 아이템 9 타입 단언보다는 타입 선언을 사용하기

## 타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 두 가지 방법

---

```tsx
interface Person { name: string }

const alice: Person = { name: 'KMin' }  // 첫번째 방법
const KMB = { name: 'KMBok' } as Person // 두번째 방법
```

첫번째 방법은 ‘타입 선언'이고, 두번째 방법은 ‘타입 단언'이다.
**타입 단언보다는 타입 선언을 사용하는 게 낫다.**

```tsx
const KMin: Person {}     // 에러 발생 O
const KMB = {} as Person  // 에러 발생 X
```

타입 선언은 할당된 값이 해당 타입을 충족하는지 확인하지만, 타입 단언은 강제로 타입을 단언했기 때문에 타입 체커에게 오류를 무시하라고 강제한다.

### 타입 선언이 정확한 경우

---

아래 코드 예시를 보자

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({ name })
// Person[] 을 원했지만 결과는 { name: string }[] 이 된다.
```

ex) 타입 단언 예시 ❌

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({ name } as Person)
// Person[] 타입이 된다.
```

{ name } 에 타입 단언을 쓰면 문제가 해결되는 것처럼 보일 수 있다.
하지만 타입 단언을 사용하면 의도치않게 런타임에서 에러가 날 수 있다.

ex) 타입 선언 예시 ⭕️

```tsx
const people = ['alice', 'bob', 'jan'].map(name => {
	const person: Person = { name }
	return person
}) // Person[] 타입이 된다.
```

이를 더 간결하게 표현할 수 있다.

```tsx
const people = ['alice', 'bob', 'jan'].map((name): Person => ({ name }))
```

위의 코드에서 `(name): Person => ...` 는 이 화살표 함수가 반환하는 값의 타입이 Person 타입임을 말한다.
`(name: Person) ⇒ ...` 와는 다르다!

### 타입 단언이 정확한 경우

---

타입스크립트가 항상 정확하지만은 않다.

```tsx
document.querySelector('#myButton').addEventListener('click', e => {
	e.currentTarget  // 타입은 EventTarget
	const button = e.currentTarget as HTMLButtonElement
	button // 타입은 HTMLButtonElement
})
```

타입스크립트는 DOM 에 접근할 수 없기 때문에 #myButton 이 버튼 엘리먼트인지 알지 못한다.
그리고 이벤트의 currentTarget 이 같은 버튼이어야 하는 것도 알지 못 한다.
이 경우 코드를 작성하는 개발자는 e.currentTarget 이 버튼 엘리먼트라는 것을 한치의 의심없이 확신하기 때문에 HTMLButtonElement 라고 타입 단언을 해준다.

```tsx
const elNull = document.getElementById('foo') // 타입은 HTMLElement | null
const el = document.getElementById('foo')!    // 타입은 HTMLElement
```

이 경우도 아이디가 foo 인 엘리먼트가 있다고 확신한다면 `!` 를 붙여서 null 이 아님을 단언해준다.

<aside>
💻 **타입 단언에 주의할 점!**

---

타입 단언문으로 임의의 타입 간에 변환을 할 수는 없다.
타입 변환을 하는 경우는 단언하려는 타입이 단언 이전의 타입의 부분집합이어야 한다.

HTMLElement | null 일 때 HTMLElement 로 단언할 수 있다.
HTMLButtonElement 는 EventTarget 의 서브타입이기 때문에 역시 동작한다.
또한 Person 은 {} 의 서브타입이므로 동작한다

그러나 Person 과 HTMLElement 는 서로의 서브타입이 아니기 때문에 변환이 불가능하다

```tsx
interface Person { name: string }
const body = document.body
const el = body as Person 
// HTMLElement 형식을 Person 형식으로 변환하는 것은 안된다.
// 의도적인 경우 먼저 식을 'unknown' 으로 변환해야 한다.
```

이 오류를 해결하려면 `unknown` 타입을 사용한다.
**모든 타입은 unknown 타입의 서브타입이기 때문에 unknown 이 포함된 단언문은 항상 동작한다.**
unknown 단언은 임의의 타입 간에 변환을 가능케 하지만, 그만큼 무언가 위험한 동작을 하고 있다는 것을 인지해야 한다.

</aside>

### 요약

---

- 타입 단언(`as Type`)보다 타입 선언(`: Type`)을 사용해야 한다.
- 화살표 함수의 반환 타입을 명시하는 방법을 터득해야 한다.
- 타입스크립트보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null 아님 단언문을 사용하면 된다.