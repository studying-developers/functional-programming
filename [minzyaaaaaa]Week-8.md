# Chapter 16. 타임라인 사이에 자원 공유하기


💡 자원을 안전하게 공유하기 위해 동시성 기본형(concurrency primitive)이라는 재사용 가능한 코드를 만드는 방법에 대해 알아본다.




<br/>

## 좋은 타임라인의 원칙

1. 타임라인은 적을수록 이해하기 쉽다.
2. 타임라인은 짧을수록 이해하기 쉽다.
3. 공유하는 자원이 적을수록 이해하기 쉽다.
4. 자원을 공유한다면 서로 조율해야 한다.
- 타임라인은 공유 자원을 안전하게 공유할 수 있어야 한다. 안전하게 공유한다는 말은 올바른 순서대로 자원을 쓰고 돌려준다는 말이다. 그리고 타임라인을 조율한다는 것은 실행 가능한 순서를 줄인다는 것을 의미한다.
5. 시간을 일급으로 다룬다.

<br/>

## 기본형 1. 큐(Queue)

- FIFO(First-in, First-out) 데이터 구조
- 큐는 여러 타임라인에 있는 액션 순서를 조율하기 위해 사용할 수 있다.
- 큐는 공유 자원이지만 안전하게 공유된다. 순서대로 작업을 꺼내 쓸 수 있기 때문이다. 그리고 큐에 있는 모든 작업은 같은 타임라인에서 처리되기 때문에 순서가 관리된다.

### 자바스크립트에서 큐 만들기

- 자바스크립트에는 큐 자료 구조가 없기 때문에 만들어서 사용 가능하다.
- 큐는 자료 구조지만 타임라인 조율에 사용한다면 동시성 기본형(concurrency primitive)이라고 부른다.
- 동시성 기본형은 자원을 안전하게 공유할 수 있는 재사용 가능한 코드를 말한다.
- 큐는 동시성 기본형이다. 여러 타임라인을 올바르게 동작하도록 만드는 재사용 가능한 코드이다. 동시성 기본형은 방법은 다르지만 모두 **실행 가능한 순서**를 제한하면서 동작한다.

```jsx
function Queue() {
	var queue_items = [];
	var working = false;

	function runNext() {
		if (working) return;
		if (queue_items.length === 0) return;

		working = true;
		
		var cart = queue_item.shift();
		calc_cart_total(cart, function(total) {
			update_total_dom(total);
			working = false;
			runNext();
		});
	}

	return function(cart) {
		queue_items.push(cart);
		setTimeout(runNext, 0);
	};
}

var update_total_queue = Queue();
```

큐를 재사용할 수 있도록, 작업이 완료되었을 때 콜백 부르기

```jsx
function Queue(worker) {
	var queue_items = [];
	var working = false;
	
	function runNext() {
		if(working) return;
		if (queue_items.length === 0) return;
		
		working = true;
		
		var item = queue_items.shift();
		worker(item.data, function(val) {
			working = false;
			setTimeout(item.callback, 0, val);
			runNext();
		});
	}

	return function() {
		queue_items.push({
			data: data,
			callback: callback || function() {}
		});
		setTimeout(runNext, 0);
	};
}

function calc_cart_worker(cart, done) {
	calc_cart_total(cart, function(total) {
		update_total_dom(total);
		done(total);
	});
}

var update_total_queue = Queue(calc_cart_worker);
```

큐를 건너뛰도록 만들기

```jsx
function Droppingqueue(max, worker) {
	var queue_items = [];
	var working = false;

	function runNext() {
		if (working) return;
		if (queue_items.length === 0) return;

		working = true;
		var item = queue_items.shift();
		worker(item.ata, function(val) {
			working = false;
			setTimeout(item.callback, 0, val);
			runNext();
		});
	}

	return function(data, callback) {
		queue_items.push({
			data: data,
			callback: callback || function() {}
		});
		while(queue_items.length > max) {
			queue_items.shift();
		}
		setTimeout(runNext, 0);
	};
}

function calc_cart_worket(cart, done) {
	calc_cart_total(cart, function(total) {
		update_total_dom(total);
		done(total);
	});
}

var updat_total_queue = DroppingQueue(1, calc_cart_worker);

```

<br/>

# Chapter 17. 타임라인 조율하기

<br/>

## 기본형 2. Cut

- 순서를 보장해 주는 역할
- 컷은 실행 가능한 순서를 줄이기 때문에 애플리케이션의 복잡성을 줄여준다.

### 경쟁 조건

어떤 동작이 먼저 끝나는 타임라인에 의존할 때 발생한다.

멀티스레드를 지원하는 언어에서는 스레드가 변경 가능한 상태를 공유하기 위해 원자적 (atomic) 업데이트 같은 기능을 사용해야 한다. 하지만 자바스크립트는 단일 스레드라는 장점을 활용하여, 동기적으로 접근하는 간단한 변수로 동시성 기본형을 구현할 수 있다. 

**Promise.all**과 같은 동작

```jsx
function cut(num, callback) {
	var num_finished = 0;
	return function() {
		num_finished += 1;
		if (num_finished === num) {
			callback();
		}
	}
}
```

<br/>

## 기본형 3. 함수를 한 번만 호출하도록 하는 함수

```jsx
function JustOnce(action) {
	var alreadyCalled = false;
	return function(a, b, c) {
		if(alreadyCalled) return;
		alreadyCalled = true;
		return action(a, b, c);
	};
}
```

<br/>

## 암묵적 시간 모델과 명시적 시간 모델

모든 언어는 암묵적으로 시간에 대한 모델을 가지고 있다. 시간 모델로 실행에 관한 두 가지 관점을 알 수 있다.

순서와 반복이다.

자바스크립트의 시간 모델

1. 순차적 구문은 순서대로 실행된다. (순서)
2. 두 타임라인에 있는 단계는 왼쪽 먼저 실행되거나, 오른쪽 먼저 실행될 수 있다. (순서)
3. 비동기 이벤트는 새로운 타임라인에서 실행된다 (순서)
4. 액션은 호출할 때마다 실행된다. (반복)
