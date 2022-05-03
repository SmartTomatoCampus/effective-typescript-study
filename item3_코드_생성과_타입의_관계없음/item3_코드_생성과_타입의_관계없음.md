# 아이템 3 코드 생성과 타입이 관계 없음을 이해하기

# 타입스크립트 컴파일러의 두 가지 역할

1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transpile)
2. 코드의 타입 오류를 체크

→ 이 두가지는 완전히 독립적으로 동작함

### 타입 오류가 있는 코드도 컴파일이 가능하다

---

**컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능하다**

<aside>
💻 **컴파일과 타입 체크**

---

코드에 오류가 있을 때 “컴파일에 문제가 있다"고 말하는 경우가 있지만 사실은 기술적으로 틀린 말.
엄밀히 말하면 코드 생성만이 ‘컴파일'이기 때문

따라서 타입스크립트가 유효한 자바스크립트라면 타입스크립트 컴파일러는 컴파일을 해냄.
따라서 코드에 오류가 있을 때 “타입 체크에 문제가 있다"고 말하는 것이 더 정확한 표현!

</aside>

만약 오류가 있을 때 컴파일하지 않으려면 `noEmitOnError` 를 true 로 설정하거나 빌드 도구에 동일하게 적용하면 된다

### 런타임에는 타입 체크가 불가능하다

---

```tsx
interface square {
	width: number
}
interface Rectangle extends Square {
	height: number
}
type Shape = Square | Rectangle

function claculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
		// 'Rectangle'은 형식만 참조하지만, 여기서는 값으로 사용되고 있습니다
		return shape.width * shape.height
		// 'Shape' 형식에 'height' 속성이 없습니다.
	} else {
		return shape.width * shape.width
	}
}
```

위의 코드에서 instanceof 체크는 런타임에 일어나지만, Rectangle 은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다.
타입스크립트의 타입은 ‘제거 가능(erasable)’하다.
실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.

위 코드에서 다루고 있는 shape 타입을 명확하게 하려면, 런타임에 타입 정보를 유지하는 방법이 필요함

1. 하나의 방법은 height 속성이 존재하는지 체크해보는 것
    
    ```tsx
    function claculateArea(shpae: Shape) {
    	if ('height' in shape) {
    		shape; // 타입이 Rectangle
    		return shape.width * shape.height
    	} else {
    		shape; // 타입이 Square
    		return shape.width * shape.width
    	}
    }
    ```
    
    속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시도 shape 의 타입을 Rectangle 로 보정해주기 때문에 오류가 사라진다
    
2. 두번째 방법은 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 ‘태그’ 기법
    
    ```tsx
    interface Square {
    	kind: 'square'
    	width: number
    }
    interface Rectangle {
    	kind: 'rectangle'
    	height: number
    	width: number
    }
    type Shape = Square | Rectangle
    
    function claculateArea(shape: Shape) {
    	if (shape.kind === 'rectangle') {
    		shape // 타입이 Rectangle
    		return shape.width * shape.height
    	} else {
    		shape // 타입이 Square
    		return shape.width * shape.width
    	}
    }
    ```
    
    여기서 Shape 타입은 ‘태그된 유니온(tagged union)’의 한 예시
    이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있음
    
3. 세번째 방법은 타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법
    
    **타입을 클래스로 만듦**
    
    ```tsx
    class Square {
    	constructor(public width: number) {}
    }
    class Rectangle extends Square {
    	constructor(public width: number, public height: number) {
    		super(width)
    	}
    }
    type Shape = Square | Rectangle
    
    function calculateArea(shape: Shape) {
    	if (shape instanceof Rectangle) {
    		shape // 타입이 Rectangle
    		return shape.width * shape.height
    	} else {
    		shape // 타입이 Square
    		return shape.width * shape.width
    	}
    }
    ```
    
    인터페이스는 타입으로만 사용이 가능하지만, Rectangle 을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없다.
    

### 타입 연산은 런타임에 영향을 주지 않는다

---

string 또는 number 타입인 값을 항상 number 로 정제하는 경우를 가정해보자
다음 코드는 타입 체커를 통과하지만 잘못된 방법을 쓴 코드다

```tsx
function asNumber(val: number | string): number {
	return val as number
}
```

이 코드는 자바스크립트로 변환되면 어떠한 정제 작용도 거치지 못한다. 
그 이유는 `as number` 구문은 타입 연산이고, 런타임 동작에는 아무런 영향을 끼치지 않기 때문
값을 정제하기 위서는 런타임의 타입을 체크해야 하고, 자바스크립트 연산을 통해 변환을 수행해야 한다.

```tsx
function asNumber(val: number | string): number {
	return typeof(val) === 'string' ? Number(val) : val
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다

---

```tsx
function setLightSwitch(value: boolean) {
	switch (value) {
		case true:
			turnLightOn()
			break
		case false: 
			turnLightOff()
			break
		default: 
			console.log(`실행되지 않을 지도..?`)
	}
}
```

타입스크립트는 실행되지 않는 죽은 코드를 찾아내지만, 여기서는 찾아내지 못한다.
`: boolean` 은 타입선언문이기 때문에 런타임에서 제거되고, 자바스크립트였다면 매개변수로 boolean 타입 외에 다른 값이 들어올 가능성이 있기 때문!

### 타입스크립트 타입으로는 함수를 오버로드할 수 없다

---

C++ 같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용한다. 이를 ‘함수 오버로딩'이라 한다.
**그러나 타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩이 불가능하다.**

```tsx
function add(a: number, b: number) { return a + b }
function add(a: string, b: string) { return a + b }
// 중복된 함수 구현!
```

타입스크립트가 함수 오버로딩을 지원하기는 하지만, 온전히 타입 수준에서만 동작한다.
하나의 함수에 대해 여러 개의 선언문을 작성할 수는 있지만, 구현체(implementation)는 오직 하나 뿐이다.

```tsx
function add(a: number, b: number): number
function add(a: string, b: string): string

function add(a, b) {
	return a + b
}

const three = add(1, 2) // 타입이 number
const twelve = add('1', '2') // 타입이 string
```

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다

---

타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에 런타임의 성능에 아무런 영향을 주지 않는다.