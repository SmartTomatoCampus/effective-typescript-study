# 아이템 10 객체 래퍼 타입 피하기

자바스크립트에는 객체 이외에도 기본형 값들에 대한 일곱 가지 타입(string, number, boolean, null, undefined, symbol, bigint)가 있다.
symbol 은 ES6 에서 추가되었고, bigint 는 최종 확정 단계에 있다.

기본형들은 불변(immutable)이며 메서드를 가지지 않는다는 점에서 객체와 구분된다.
반면 기본형인 string 의 경우 메서드를 가지고 있는 것처럼 보인다.

```tsx
'primitive'.charAt(3)
```

하지만 사실 charAt 은 string 의 메서드가 아니며, string 을 사용할 때 자바스크립트 내부적으로 많은 동작이 일어난다.
**string ‘기본형'에는 메서드가 없지만, 자바스크립트에는 메서드를 가지는 String ‘객체' 타입이 정의되어 있고, 기본형과 객체 타입을 서로 자유롭게 변환한다.**

string 기본형에 charAt 같은 메서드를 사용할 때, 자바스크립트는 기본형을 String 객체로 래핑(wrapping)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버린다.

가장 중요한 것은 string 기본형과 String 래퍼객체를 혼동해선 안 된단 것이다.

```tsx
"hello" === new String('hello") // false
new String("hello") === new String("hello") // false
```

기본형으로만 비교한다면 “hello” 와 “hello” 는 같겠지만,
String 객체는 오직 자기자신과 동일하기 때문에 이러한 결과가 나타난다.

```tsx
x = "hello"
x.language = 'English' // 'English'
x.language // undefined
```

x 는 string 기본형이지만, language 프로퍼티를 할당하는 과정에서 String 래퍼객체로 변환됐고, 이후 래퍼객체를 다시 버리면서 x 는 다시 string 기본형으로 돌아오고, 이로 인해 x.language 는 당연히 undefined 가 뜬다.

이런 래퍼 타입들 덕분에 기본형 값에 메서드를 사용할 수 있다.
**타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링한다.**

- string 과 String
- number 와 Number
- boolean 과 Boolean
- symbol 과 Symbol
- bigint 와 Bigint

**주의할 점은 string 은 String 에 할당할 수 있지만, String 은 string 에 할당할 수 없다는 점이다.**

당연히 런타임의 값은 객체가 아니고 기본형이므로, 웬만해서 타입을 선언할 때 기본형 타입을 사용ㅎ아는 것이 좋다.

### 요약

---

- 기본형 값에 메서드를 제공하기 위해 객체 래퍼 타입이 어떻게 쓰이는지 이해해야 한다.
직접 사용하거나 인스턴스를 생성하는 것은 피해야 한다.
- 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야 한다.