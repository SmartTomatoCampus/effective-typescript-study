# 아이템 2 타입스크립트 설정 이해하기

먼저 아래 코드를 보자

```tsx
// program.ts

function add(a, b) {
	return a + b
}
add(10, null)
```

위의 코드는 과연 에러를 띄울까, 안 띄울까?

⇒ 정답은 설정에 따라 다르다!!

타입스크립트 컴파일러에 대한 설정은 100개에 이른다. 이 설정들은 커맨드 라인에서도 사용할 수 있다.

```bash
tsc --noImplicitAny program.ts
```

또는 `tsconfig.json` 설정 파일을 통해서도 설정이 가능하다!

```tsx
{
	"compilerOptions: {
		"noImplicitAny: true
	}
}
```

가급적이면 이처럼 설정파일을 사용하는 것이 좋다. `tsc --init` 만 실행하면 간단히 생성된다.

타입스크립트는 어떻게 설정하느냐에 따라 전혀 다른 언어가 될 수도 있다. 
설정을 제대로 사용하려면, `noImplicitAny` 옵션과 `strictNullChecks` 를 이해해야 한다.

## noImplicitAny 옵션

---

```tsx
function add(a, b) {
	return a + b
}
```

위의 코드에 마우스를 올려보면, 타입스크립트가 추론한 함수의 타입을 알 수 있다.

```tsx
function add(a: any, b: any): any
```

any 타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해진다.
any 는 유용하지만 매우 주의해서 사용해야 한다.

이때 설정에서 `noImplicitAny` 를 true 로 설정하면 위의 add 함수는 오류가 발생한다.
암묵적으로 any 가 할당되는 문제가 발생했기 때문!!

```tsx
function add(a: number, b: number) {
	return a + b
}
```

이때 매개변수의 타입을 정확히 지정해줌으로써 명확히 타입 체크를 해줄 수 있기 때문에 `noImplicitAny` 를 설정해주는 것이 좋다.

## strictNullChecks 옵션

---

`strictNullChecks` 옵션은 null 과 undefined 가 모든 타입에서 허용되는지 확인하는 설정이다.

```tsx
const x: number = null
```

`strictNullChecks` 해제되었을 때 위의 코드는 정상 작동한다.
반면 `strictNullChecks` 옵션이 설정되면 위의 코드는 에러가 발생한다. (undefined 를 할당해도 마찬가지)