# Week 9

# Chapter 18. 반응형 아키텍처와 어니언 아키텍처

💡 반응형 아키텍처로 순차적 액션을 파이프라인으로 만드는 방법을 배운다.
상태 변경을 다루기 위한 기본형을 만든다.
도메인과 현실 세계의 상호작용을 위해 어니언 아키텍처를 만든다.
여러 계층에 어니언 아키텍처를 적용하는 방법을 살펴본다.
전통적인 계층형 아키텍처와 어니언 아키텍처를 비교한다.


반응형 아키텍처는 순차적 액션 단계에 사용하고, 어니언 아키텍처는 서비스의 모든 단계에 사용한다. 두 패턴은 함께 사용할 수 있지만 따로 사용할 수도 있다.

반응형 아키텍처는 코드에 나타난 순차적 액션의 순서를 뒤집는다. 효과와 그 효과에 대한 원인을 분리해서 코드에 복잡하게 꼬인 부분을 풀 수 있다.

어니언 아키텍처는 현실 세계와 상호작용하기 위한 서비스 구조를 만든다. 함수형 사고를 적용한다면 자연스럽게 쓸 수 있는 아키텍처이다.

## 반응형 아키텍처

반응형 아키텍처는 애플리케이션을 구조화하는 방법이다. 반응형 아키텍처의 핵심 원칙은 이벤트에 대한 반응으로 일어날 일을 지정하는 것이다. 반응형 아키텍처는 웹 서비스와 UI에 잘 어울린다. 웹 서비스는 웹 요청 응답에 일어날 일을 지정하고, UI는 버튼 클릭과 같은 이벤트 응답에 일어날 일을 지정하면 된다. 이런 것을 일반적으로 이벤트 핸들러라고 한다.

반응형 아키텍처의 절충점

반응형 아키텍처는 코드에 나타난 순차적 액션의 순서를 뒤집는다. 

- 원인과 효과가 결합한 것을 분리한다.
- 여러 단계를 파이프라인으로 처리한다.
- 타임라인이 유연해진다.

**변경 가능한 값을 일급 함수로 만들기**

```jsx
function ValueCell(initialValue) {
	var currentValue = initialValue;
	return {
		val: function() {
			return currentValue;
		},
		update: function(f) {
			var oldValue = currentValue;
			var newValue = f(oldValue);
			currentValue = newValue;
		}
	};
}
```

**감시자 (watcher)**

- event handler, observer, callback, listener

```jsx
function ValueCell(initialValue) {
	var currentValue = initialValue;
	var watchers = [];
	return {
		val: function() {
			return currentValue;
		},
		update: function(f) {
			var oldValue = currentValue;
			var newValue = f (oldValue);
			if (oldValue !== newValue) {
				currentValue = newValue;
				forEach(watchers, function (watcher) {
					watcher(newValue);
				});
			}
		},
		addWatcher: function(f) {
			watchers.push(f);
		}
	}
} 
```

**의존성이 있는 경우**

```jsx
function FormulaCell(upstreamCell, f) {
	var myCell = valuecell(f(upstreamCell.val());
	upstreamCell.addWatcher(function(newUpstreamalue) {
		myCell.update(function(currentValue) {
			return f(newUpstreamValue);
		}
	});
	return {
		val: myCell.val,
		addWatcher: myCell.addWatcher
	};
}
```

ValueCell과 비슷한 개념

- 클로저, 리액트 상태관리

## 어니언 아키텍처

현실 세계와 상호작용하기 위한 서비스 구조를 만드는 방법

인터랙션 계층 - 바깥세상에 영향을 주거나 받는 액션

도메인 계층 - 비즈니스 규칙을 정의하는 계산

언어 계층 - 언어 유틸리티와 라이브러리

규칙

1. 현실 세계와 상호작용은 인터랙션 계층에서 해야 한다.
2. 계층에서 호출하는 방향은 중심 방향이다.
3. 계층은 외부에 어떤 계층이 있는지 모른다.

계층형 설계

함수 호출 관계를 기반으로 함수를 배치하는 방법이다. 어떤 함수가 재사용성이 좋고 변경하기 쉬운지, 테스트할 가치가 높은 코드가 무엇인지 알 수 있다.

함수형 아키텍처와 **전통적인 계층형 아키텍처의 비교**

전통적인 아키텍처로 웹 API를 만들 때 계층(layer)이라고 하는 개념을 사용한다. 하지만 어니언 아키텍처의 계층과는 다르다.

웹 인터페이스 계층 - 웹 도요청을 도메인으로 바꾸고 도메인을 웹 응답으로 바꾼다.

도메인 계층 - 애플리케이션 핵심 로직으로 도메인 개념에 DB 쿼리나 명령이 들어간다.

데이터베이스 계층 - 시간에 따라 바뀌는 정보를 저장한다.

전통적인 계층형 아키텍처는 데이터베이스를 기반으로 한다. 도메인 계층은 데이터베이스 동작으로 만든다. 그리고 웹 인터페이스는 웹 요청을 도메인 동작으로 변환한다.

데이터베이스 계층이 가장 아래에 있다면 그 위에 있는 모든 것이 액션이 되기 때문에 함수형 스타일이 아니다. 함수형 아키텍처는 계산과 액션에 대한 명확한 규칙이 있어야 한다.

반면 함수형 아키텍처는 도메인 계층이 데이터베이스 계층에 의존적이지 않다. 데이터베이스는 변경 가능하고 접근하는 모든 것을 액션으로 만드는 것이 핵심이다. 따라서 데이터베이스를 도메인과 분리하는 것이 중요하다.

## Summary

- 반응형 아키텍처는 코드에 나타난 순차적 액션의 순서를 뒤집는다.
- 반응형 아키텍처는 액션과 계산을 조합해 파이프라인을 만든다. 파이프라인은 순서대로 발생하는 작은 액션들의 조합이다.
- 읽고 쓰는 동작을 제한해 변경 가능한 일급 상태를 만들 수 있다.
- 어니언 아키텍처는 넓은 범위에서 소프트웨어를 세 개의 계층으로 나눈다. 인터랙션과 도메인, 언어 계층이다.
- 가장 바깥 인터랙션 계층은 액션으로 되어있다. 도메인 계층과 액션을 사용하는 것을 조율한다.
- 도메인 계층은 도메인 로직과 비즈니스 규칙과 같은 소프트웨어의 동작으로 되어 있다. 도메인 계층은 대부분 계산으로 구성되어 있다.
