### 생각해보기

<aside>
‼️ `useState` 에 할당된 값은 액션인가 데이터인가?

</aside>

- 나의 답 : `액션`이다

**답의** **근거**

- `useState`는 리액트 훅이다. 리액트에서는 `setState`가 컴포넌트의 `state` 객체에 대한 업데이트를 실행한다고 한다. ([리액트 공식문서 보러가기](https://ko.legacy.reactjs.org/docs/faq-state.html))
    
    업데이트를 실행한다는 것은 실행 시점에 따라 값이 변한 다는 것이고 이것은 `state`가 **실행 시점에 의존**하는 것이기 때문에 `액션`이다.
    
- 책 p.57에서 액션의 다양한 형태를 소개한다. **상태 값 할당 부분**에서 공유하기 위해 값을 할당 했고 변경 가능한 변수라면 다른 코드에 영향을 주기 때문에 `액션`이라고 정의한다.
    
    여기서 주목해야 할 문장은 ‘**변경 가능한** 변수라면 다른 코드에 영향을 주기 때문에 액션이라고 정의한다’ 이다. `useState`의 `state` 도 **변경 가능한** 객체이며 다른 코드에 영향을 주기 때문에 `액션`이라고 생각한다. 

    <br/>
<br/>

# Chapter 4.

- 테스트 하기 쉽고 재사용성이 좋은 코드를 만들기 위한 함수형 기술
- 액션에서 계산을 빼내는 방법

<br/>
<br/>
<br/>

## MegaMart.com 의 비즈니스 규칙
- 구매 합계가 20 달러 이상이면 무료 배송
    - 장바구니에 넣었을 때 합계가 20달러가 넘는 제품의 구매 버튼 옆에 무료 배송 아이콘 표시
- 장바구니의 금액 합계가 바뀔 때마다 세금을 다시 계산해야 한다.

→ 책에서는 절차적인 방법으로 구현

<br/>
<br/>

### 지금의 코드는 비즈니스 규칙을 테스트하기 어렵다.

- 코드가 바뀔 때 마다 조지는 아래와 같은 테스트를 만들어야 한다.
    1. 브라우저 설정하기
    2. 페이지 로드하기
    3. 장바구니에 제품 담기 버튼 클릭
    4. DOM이 업데이트 될 때까지 기다리기
    5. DOM에서 값 가져오기
    6. 가져온 문자열 값을 숫자로 바꾸기
    7. 예상하는 값과 비교하기


### 테스트 개선을 위한 조지의 제안

- DOM 업데이트와 비즈니스 규칙은 분리되어야 한다!
- 전역변수가 없어야 한다!

→ 조지의 제안은 함수형 프로그래밍과 잘 맞는다.
<br/>
<br/>

## 재사용하기 쉽게 만들기
### 결제팀과 배송팀이 우리 코드를 사용하려고 할때

- 현재의 코드는 다음과 같은 이유로 사용할 수 없다.
    - 장바구니 정보는 전역변수에서 읽어오고 있지만, 결제팀과 배송팀은 데이터베이스에서 장바구니 정보를 읽어 와야 한다.
    - 결과를 보여주기 위해 DOM을 직접 바꾸고 있지만, 결제팀은 영수증을, 배송팀은 운송장을 출력해야 한다.

### 개발팀 제나의 제안

- 전역변수에 의존하지 않아야 한다.
- DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안된다.
- 함수가 결괏값을 리턴해야 한다.

→ 제나가 제안한 내용은 함수형 프로그래밍과 잘 맞는다.

<br/>
<br/>

## 액션과 계산, 데이터를 구분하기

- 테스트 개선을 위해 가장먼저 각 함수가 액션과 계산, 데이터 중 어떤 것인지 구분하는 일을 해야한다. → 그래야 어떻게 개선할 지 알 수 있다.

<br/>
<br/>

## 함수에는 입력과 출력이 있다

- **입력**은 함수가 계산을 하기 위한 외부 정보
- **출력**은 함수 밖으로 나오는 정보나 어떤 동작
- 함수를 호출하는 이유는 결과가 필요하기 때문 → 원하는 결과를 얻으려면 입력이 필요

### 입력과 출력은 명시적이거나 암묵적일 수 있다.

- **인자**는 **명시적인 입력**
- `return` 값은 **명시적인 출력**

### 함수에 암묵적 입력과 출력이 있으면 액션이 된다.

- 함수에서 암묵적 입력과 출력을 없애면 `계산`이 된다.
- 암묵적 입력 → 함수의 인자로, 암묵적 출력 → 함수의 `return` 값으로

<aside>
‼️ 함수형 프로그래머는 암묵적 입력과 출력을 **부수효과**라고 부릅니다. 부수효과는 함수가 하려고 하는 주요 기능(리턴 값을 계산하는 일)이 아니다.

</aside>

<br/>
<br/>


## 테스트와 재사용성은 입출력과 관련 있다

### 1. DOM 업데이트와 비즈니스 규칙은 분리되어야 한다.

- DOM을 업데이트 하는 일 → 함수에서 어떤 정보가 나오는 것 →출력
    - 하지만 `return` 값이 아니기 때문에 **암묵적 출력**
    - 그래서 암묵적인 출력인 DOM 업데이트와 비즈니스 규칙을 분리하여 계산을 만들어서 테스트와 재사용성을 높인다.

### 2. 전역변수가 없어야 한다.

- 전역변수를 읽는 것은 암묵적 입력이고 바꾸는 것은 암묵적 출력이다. → 암묵적 입력과 출력을 없애자!
- 암묵적 입력은 인자로 바꾸고 암묵적 출력은 리턴값으로 바꾸어 계산을 만들고, 테스트와 재사용성을 높인다.

### 3. DOM을 사용할 수 있는 곳에서 실행된다고 가정하면 안된다.

- 앞에서 만든 함수들은 DOM에 직접 접근하는 암묵적 출력! → 암묵적 출력은 함수의 리턴 값으로 바꿀 수 있다.

### 4. 함수가 결과값을 리턴해야 한다.

- 결과값 리턴 = 명시적 출력

  <br/>
<br/>


## 액션에서 계산 빼내기

- 계산에 해당하는 코드 분리
- 입력값은 인자로, 출력값은 리턴으로 바꾼다.

→ **서브루틴** 추출하기

### 암묵적 출력을 명시적 출력으로 바꾸기

- 새롭게 만든 함수(`calc_total()`)는 여전히 액션 → 입력과 출력을 파악하기
- 전역변수 대신 지역변수를 사용하도록 바꾸고 지역변수값을 리턴하도록 고친다.
- 그리고 `shapping_cart_total()` 함수는 `calc_total()`함수의 리턴값을 받아 전역변수에 할당한다.

### 암묵적 입력을 명시적 입력으로 바꾸기

- `calc_total()` 함수에 전역변수 대신 `cart` 인자를 받아 사용하도록 바꾼다.
- `calc_total()` 함수에 `shopping_cart`를 인자로 전달한다.

<br/>
<br/>

## 계산 추출을 단계별로 알아보기

1. 계산 코드를 찾아 빼낸다.
2. 새 함수에 암묵적 입력과 출력을 찾는다.
3. 암묵적 입력은 인자로, 암묵적 출력은 리턴값으로 바꾼다.

<br/>
<br/>

## 요점 정리

- 액션은 암묵적인 입력 또는 출력을 가지고 있다.
- 계산의 정의에 따르면 계산은 암묵적인 입력이나 출력이 없어야 한다.
- 공유변수(전역변수와 같은)는 일반적으로 암묵적 입력 또는 출력이 된다.
- 암묵적 입력은 인자로 바꿀 수 있다.
- 암묵적 출력은 리턴값으로 바꿀 수 있다.
- 함수형 원칙을 적용하면 액션은 줄어들고 계산은 늘어난다.

<br/>
<br/>

# Chapter 5.

- 암묵적 입력과 출력을 제거해서 재사용하기 좋은 코드를 만드는 방법
- 복잡하게 엉킨 코드를 풀어 더 좋은 구조로 만드는 법

<br/>
<br/>
<br/>


## 비즈니스 요구 사항과 설계를 맞추기

### 요구 사항에 맞춰 더 나은 추상화 단계 선택하기

- 액션에서 계산으로 리팩토링 하는 과정은 단순하고 기계적임 → 항상 최선의 구조를 만드는 것은 아니다.
    
    ```jsx
    function gets_free_shipping(total, item_price){
    	 return item_price + total >=20;
    }
    
    function calc_total(cart) {
    	const total = 0;
    	for(var i = 0; i < cart.length; i++){
    		const item = cart[i];
    		total += item.price;
    	}
    	return total;
    }
    ```
    
- `gets_free_shipping()` 함수는 비즈니스 요구 사항으로 봤을때 맞지 않는 부분이 있다.
    - 요구사항: 장바구니에 담긴 제품을 주문할 때 무료배송인지 확인하는 것
    - 실제 로직: 제품의 합계와 가격으로 확인
- 중복된 코드 존재 → **코드의 냄새**
    - 장바구니 합계를 계산하는 코드(`item + total`)가 중복
- `gets_free_shipping(total, item_price)` 함수를 아래와 같이 바꾼다. 그리고 `calc_total()`함수를 재사용하여 중복을 없앤다.
    
    ```jsx
    function get_free_shipping(cart) {
    	return calc_total(cart) >= 20;
    }
    ```
    
    - 바꾼 함수는 장바구니 데이터를 사용한다.
    - 전자상거래에서 많이 사용하는 entity 타입이기 대문에 비즈니스 요구 사항과 잘 맞는다.

<br/>
<br/>

## 원칙: 암묵적 입력과 출력은 적을수록 좋다.

- 인자가 아닌 모든 입력은 암묵적 입력이고 리턴값이 아닌 모든 출력은 암묵적 출력이다.
- 암묵적 입력과 출력이 없는 함수는 **계산**이다.
- 어떤 함수에 암묵적입력과 출력이 있다면 다른 컴포넌트와 강하게 연결된 컴포넌트라고 할 수 있다. → 다른 곳에서 사용할 수 없기 때문에 모듈이 아니다.
- 암묵적 입력과 출력을 명시적으로 바꿔 모듈화된 컴포넌트로 만들 수 있다.
- 암묵적 입력과 출력이 있는 함수는 아무 때나 실행할 수 없기 때문에 테스트하기 어렵다.
- 반면에 **계산**은 암묵적 입력과 출력이 없기 때문에 테스트하기 쉽다.

<br/>
<br/>

## 계산 분류하기
### 의미있는 계층에 대해 알아보기 위해 계산을 분류

- 어떤 함수가 장바구니 구조를 알아야 하면 C, 제품에 대한 구조를 알아야 하면 I, 비즈니스 규칙에 대한 함수라면 B라고 표시해 보자.

<br/>
<br/>

## 원칙: 설계는 엉켜있는 코드를 푸는 것이다

- 함수를 사용하면 관심사를 자연스럽게 분리할 수 있다. → 함수를 인자로 넘기는 값과 그 값을 사용하는 방법으로 분리
- 크고 복잡한 것이 잘 만들어진 것 같다고 느끼지만 분리된 것은 언제든 쉽게 조립할 수 있다. → 잘 분리하는 방법을 찾기가 더 어려움
- 함수는 작으면 작을 수록 재사용하기 쉽다.
- 작은 함수는 쉽게 이해할 수 있고 유지보수 하기 쉽다. 코드가 작기 때문에 올바른지 아닌지 명확하게 알 수 있다.
- 작은함수는 테스트하기 좋다. 한 가지 일만 하기 때문에 한 가지만 테스트하면 된다.
- 함수에 특별한 문제가 없어도 꺼낼 것이 있다면 분리하는 것이 좋다. 그러면 더 좋은 설계가 된다.

<br/>
<br/>

## add_Item() 을 분리해 더 좋은 설계 만들기

- add_item() 함수는 네 부분으로 하는 일을 나눌 수 있다.
    - `item`구조만 알고있는 함수와 `cart` 구조만 알고 있는 함수로 나눔
    - 이렇게 분리하면 `cart`와 `item` 을 독립적으로 확장할 수 있다.
    - 1, 3, 4번은 값을 바꿀 때 복사하는 **카피-온-라이트**를 구현한 부분이기 때문에 함께 두는 것이 좋다.
    - `add_item()` 함수는 이제 장바구니와 제품에만 쓸 수 있는 함수가 아닌 어떤 배열이나 항목에도 쓸 수 있다. → 함수 이름 변경 (`add_element_last(cart, item)`)
    - 이 함수는 이제 재사용 할 수 있는 **유틸리티** 함수라고 할 수 있다.
    

<br/>
<br/>

## 요점 정리

- 일반적으로 암묵적 입력과 출력은 인자와 리턴값으로 바꿔 없애는 것이 좋다.
- 설계는 엉켜있는 것을 푸는 것. 풀려있는 것은 언제든 다시 합칠 수 있다.
- 엉켜있는 것을 풀어 각 함수가 하나의 일만 하도록 하면, 개념을 중심으로 쉽게 구성할 수 있다.

  <br/>
<br/>


## 연습문제

```jsx
function update_shipping_icons(cart){
	// button에 대한 동작
	const buy_buttons = get_buy_buttons_dom();

	// Item, button
	for(let i = 0; i < buy_buttons.length; i++) {
		const item = button.item;
		const new_cart = add_item(cart, item);
		
		// 비즈니스 로직
		if(gets_free_shipping(new_cart))
			button.show_free_shipping_icon();
		else 
			button.hide_free_shipping_icon();
	}

}
```

**함수가 하는 일**

1. 모든 버튼을 가져오기
2. 버튼을 가지고 반복하기
3. 버튼에 관련된 제품을 가져오기
4. 가져온 제품을 가지고 새 장바구니 만들기
5. 장바구니가 무료배송이 필요한지 확인하기
6. 아이콘 표시하거나 감추기

**계산 분류해보기(나의 답)**

```jsx

function show_or_hide_shipping_icon(button, new_cart) {
	if(gets_free_shipping(new_cart)) 
		return button.show_free_shipping_icon();
	else 
		return button.hide_free_shipping_icon();
	}
}

function gets_new_cart_with_item(cart, item){
	const new_cart = add_item(cart, item);
	return new_cart;
}

function update_shipping_icons(cart){
	const buy_buttons = get_buy_buttons_dom();

	for(let i = 0; i < buy_buttons.length; i++) {
		const button = buy_buttons[i];
		const item = button.item;

		show_or_hide_shipping_icon(button, get_new_cart_with_item(cart, item))
	}

}
```
