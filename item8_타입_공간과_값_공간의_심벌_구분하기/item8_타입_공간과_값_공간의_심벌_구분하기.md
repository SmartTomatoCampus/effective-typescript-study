# 아이템 8 타입 공간과 값 공간의 심벌 구분하기

**타입스크립트의 심벌(symbol)은 타입 공간이나 값 공간 중의 한 곳에 존재한다.**
심벌은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있기 때문에 혼란스러울 수 있다.

```tsx
interface Cylinder {
	radius: number
	height: number
}

const Cylinder = (radius: number, height: number) => ({radius, height})
```

interface Cylinder 에서 Cylinder 는 타입으로 쓰인다.
반면 const Cylinder 에서 Cylinder 와 이름은 같지만 값으로 쓰이며, 서로 아무런 관련이 없다.
상황에 따라 타입 또는 값으로 쓰일 수 있다. 때문에 가끔 오류를 야기하기도 한다.

```tsx
function calculateVolume(shape: unknown) {
	if (shape instanceof Cylinder) {
		shape.radius
		// ~~~~~ '{}' 형식에 'radius' 속성이 없습니다.
	}
}
```

위의 instanceof 를 이용해 shape 가 Cylinder 타입인지 체크하려고 했으나, 위의 Cylinder 는 값으로 인식한다.
따라서 `instanceof Cylinder` 는 Cylinder 함수를 참조한다.

한 심벌이 타입인지 값인지는 언뜻 봐서는 알 수 없다.
어떤 형태로 쓰이는지 문맥을 살펴 알아내야 한다.

타입 선언 `:` 또는 단언문 `as` 뒤에 나오는 심벌은 타입이지만, `=` 다음에 나오는 모든 것은 값이다.

```tsx
class Cylinder {
	radius = 1
	height = 1
}
function calculateVolume(shape: unknown) {
	if (shape instanceof Cylinder) {
		shape        // 정상, 타입은 Cylinder
		shape.radius // 정상, 타입은 number
	}
}
```

**클래스가 타입으로 쓰일 때는 형태(속성과 메서드)가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용된다.**

한편, 연산자 중에서도 타입에서 쓰일 떄와 값에서 쓰일 때 다른 기능을 하는 것이 있다. 대표적으로 `typeof` 가 있다.

```tsx
type T1 = typeof p     // 타입은 Person 
type T2 = typeof email // 타입은 (p: Person, subject: string, body: string) => Response

const v1 = typeof p      // 값은 'object'
const v2 = typeof email  // 값은 'function'
```

- 타입의 관점에서 `typeof` 는 값을 읽어 타입스크립트 타입을 반환한다.
타입 공간의 `typeof` 는 보다 큰 타입의 일부분으로 사용할 수 있고, type 구문으로 일므을 붙이는 용도로도 사용할 수 있다.
- 값의 관점에서 `typeof` 는 자바스크립트 런타임의 `typeof`연산자가 된다.
값 공간의 `typeof` 는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환하며, 타입스크립트의 타입과는 별개다.

```tsx
const v = typeof Cylinder // 값이 'function' 
type T = typeof Cylinder  // 타입이 typeof Cylinder
```

클래스가 자바스크립트에서는 실제 함수로 구현되기 때문에 첫 번째 줄의 값은 function 이 나온다.
두번째 줄의 타입은 new 키워드를 사용할 떄 볼수 있는 생성자 함수 Cylinder 의 타입 Cylinder 가 나온다.
(Cylinder 가 인스턴스의 타입이 아니란 점에 주의해야 한다)

```tsx
declare let fn: T
const c = new fn() // 타입이 Cylinder
```

다음 코드처럼 `InstanceType` 제너릭을 사용해 생성자 타입과 인스턴스 타입을 전환할 수 있다.

```tsx
type C = InstanceType<typeof Cylinder> // 타입이 Cylinder
```

**프로퍼티 대괄호 접근법 `[]` 는 타입으로도 쓰일 때에도 동일하게 동작한다.**

```tsx
const first: Person['first'] = p['first'] // 또는 p.first
```

p.first 와 p[’first’]는 프로퍼티에 접근하는 점표기법과 대괄호표기법의 차이다.
**하지만 타입스크립트 타입에서 프로퍼티 타입에 접근하기 위해서는 대괄호 표기법으로 `Person['first']` 처럼 접근해야 한다!!**

```tsx
type PersonEl = Person['first' | 'last'] // 타입은 string
type Tuple = [string, number, Date]
type TupleEl = Tuple[number] // 타입은 string | number | Date
```

### 그 외의 값과 타입에서의 차이

---

**`this`**

- 값으로 쓰이는 this 는 자바스크립트의 this 다.
- 타입으로 쓰이는 this 는 일명 ‘다형성(polymorphic) this’라고 불리는 this 의 타입스크립트 타입이다.
서브클래스의 메서드 체인을 구현할 때 유용하다

`**&` 와 `|`**

- & : 값에서는 ‘AND’, 타입에서는 인터섹션이다.
- | : 값에서는 ‘OR’, 타입에서는 유니온이다.

`**const**`

- const 는 새 변수를 선언하지만, `as const` 는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꾼다.(아이템 21)

`**extends**`

- extends 는 서브클래스(`class A extends B`) 또는 서브타입(`interface A extends B`) 또는 제너릭 타입의 한정자(`Generic<T extends number>`)를 정의한다.

### 요약

---

- 타입스크립트 코드를 읽을 때 타입인지 값인지 구분해야 한다.
- 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.
- class 나 enum 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
- “foo” 는 문자열 리터럴이거나, 문자열 리터럴 타입일 수 있다.
- typeof, this 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.