```
- 함수형 프로그래밍을 하려면 부수 효과를 처리하는 방법을 알아야 한다.
- 부수 효과가 있는 함수인 액션과 부수 효과가 없는 함수인 계산을 잘 분리하는 방법을 설명한다. 
- 불변형 데이터 구조를 만들어 전역변수와 같은 부수 효과를 없애는 방법을 설명하고, 입출력과 같은 부수 효과를 순수 함수와 분리하는 방법을 배울 수 있다. 
- 또 필요한 부수 효과와 그렇지 않은 함수를 설계의 관점에서 어떻게 구성하는 것이 좋을지도 설명하고, 동시성에 대한 문제도 다룬다.
```

함수형 코딩은 코드를 **액션과 계산, 데이터**로 구분한다. <br/>
함수형 프로그래밍은 분산 시스템에 잘 어울리는 **프로그래밍 패러다임**이다.

<br/>

# Action

- 실행 시점이나 횟수 또는 둘 다에 의존한다.
    - 언제 실행되는지 - 순서
    - 얼마나 실행되는지 - 반복
- Side Effect가 일어나는 함수
- 함수형 프로그래밍에서 액션을 다루는 방법
    - 가능한 액션을 적게 사용한다.
    - 액션은 가능한 작게 만든다.
    - 액션이 외부 세계와 상호작용하는 것을 제한할 수 있다.
    내부에 계산과 데이터만 있고 가장 바깥쪽에 액션이 있는 구조가 이상적 (캡슐화)
    - 액션이 호출 시점에 의존하는 것을 제한한다.

# Calculation

- 같은 입력값을 넣을 시, 항상 같은 결괏값(출력)이 나온다.
- 테스트하기에 용이하다.
- 순수 함수, 수학 함수
- 계산은 일반적으로 함수형 프로그래밍 외부에 있는 액션을 통해 수행된다.
- 계산을 사용해야 하는 이유
    - 테스트에 용이 : 외부에 의존적이지 않기 때문에 원하는 만큼 테스트 실행이 가능하다.
    - 기계적인 분석에 용이
    - 조합하기 쉬움
- 계산의 특징
    - 동시에 실행되는 것을 신경쓰지 않아도 된다.
    - 실행 횟수가 무의미하다.

# Data

- 이벤트에 대해 기록한 사실
- 구현하는 방법
    - 기본 데이터 타입 - 숫자, 문자, 배열, 객체 (JS)
- 불변성
    - 카피-온-라이트
    - 방어적 복사
- 실행하지 않아도 데이터 자체로 유의미하다.
    - 직렬화
    - 동일성 비교
    - 자유로운 해석

설계를 할 때(문제 파악 시), 코딩 할 때, 코드를 읽을 때 모두 이 세가지를 **구분**하려는 노력 필요

# 계층형 설계 (Startified Design)

테스트, 재사용, 유지보수

상위 계층 (자주 바뀌는 것) → 하위 계층 (자주 바뀌지 않는 것)

비즈니스 규칙 - 도메인 규칙 - 기술 스택

- 각 계층은 그 아래에 있는 계층을 기반으로 만들어진다.
- 상위 계층 코드는 의존성이 적기 때문에 쉽게 변경 가능하며, 하위 계층에 있는 코드는 의존성이 많아 바꾸기 어렵지만 자주 바뀌지 않는다.

# 타임라인 다이어그램

- 분산 시스템 / 시간에 따라 변하는 액션을 타임라인으로 시각화하기
- 타임라인을 서로 맞추지 않은 분산 시스템은 예측 불가능한 순서로 실행된다.
- 액션이 실행되는 시간은 중요하지 않다.
- 타이밍이 어긋나는 경우가 존재한다.

## 타임라인 커팅

여러 타임라인이 동시에 진행될 때 서로 순서를 맞추는 방법이다.

타임라인 커팅은 고차 동작으로 구현된다.

# 불변성

불변 데이터를 유지함으로써 해당 변수가 시점과 상관없이 같은 값을 지니게끔 해야한다.

자바스크립트에서 원시 타입의 경우에는 value context이지만, 객체의 경우 identifier context를 따른다. 즉, 원시 타입은 const를 사용하면 불변성이 보장되지만, 객체 타입은 const만으로 불변성이 보장되지 않는다.

## Copy-on-write

얕은 복사(shallow copy)

카피온라이트는 중첩된 객체까지 메모리 주소 공유를 분리시키지는 못한다. 따라서 이를 구조적 공유라고 부르기도 한다.

함수형 프로그래밍에서 함수 인자로 객체를 받을 때 데이터의 불변성을 유지하기 위해 반드시 카피-온-라이트를 해야 한다.

## Defensive-copy

깊은 복사

신뢰할 수 없는 코드로부터 받은 데이터를 사용할 때 쓰인다. 중요한 것은 받을 때와 내보낼 때 모두 deep copy가 이루어져야 한다.
