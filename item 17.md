## item 17 변경 관련된 오류 방지를 위해 readonly 사용하기

readonly 매개변수를 사용하여 인터페이스를 명확히하고, 매개변수가 변경되는 것을 방지할 수 있다.
readonly는 얕게 동작하지만 제너릭을 만들면 깊은 readonly 타입을 사용할 수 있는데, 제너릭은 만들기 까다롭기 때문에 라이브러리르 사용하는게 낫다.
ex. ts-essentials의 DeepReadonly 제너릭 사용
