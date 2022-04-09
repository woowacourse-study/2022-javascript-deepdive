# this 정리

- this는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수
  - this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있음
- 자바스크립트에서 this -> 실행 컨텍스트가 생성될 때 결정됨
  - 실행 컨텍스트 -> 함수를 호출할 때 결정됨
  - ∴ this -> 함수를 호출할 때 결정됨

<br>

## 전역 공간에서의 this

- (개념상 전역 컨텍스트를 생성하는 주체가 전역 객체이기 때문에) 전역 공간에서의 this -> **전역 객체**
  - 브라우저 -> window
  - Node.js -> global
- 엄격모드에서의 this -> **undefined**

<br>

### 번외

- 자바스크립트의 모든 변수는 특정 객체(실행 컨텍스트의 L.E)의 프로퍼티로서 동작
  - 때문에 전역변수 선언 시에도 자바스크립트 엔진은 이를 전역객체 프로퍼티로 할당
- 그러나 '삭제' 명령에 대해서 전역변수 선언과 전역객체 프로퍼티 할당은 서로 다르게 동작

  - 전역변수를 선언하면 자동으로 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의

  ```js
  // 전역변수 선언
  var a = 1;
  delete window.a; // false
  // or
  delete a; // false

  // 전역객체 프로퍼티 할당
  window.b = 2;
  delete window.b; // true
  // or
  delete b; // true
  ```

<br>

## 메서드와 함수에서의 this

### 메서드로서 호출할 때 그 메서드 내부에서의 this

- 어떤 함수를 호출할 때 그 함수 이름(프로퍼티명) 앞에 객체가 명시되어 있는 경우에는 **메서드로 호출한 것**이고, 그렇지 않은 모든 경우에는 **함수로 호출한 것**
- 메서드로 호출 시, this에는 **호출한 주체에 대한 정보**가 담김
  - 호출 주체 -> **함수명(프로퍼티명) 앞의 객체**

<br>

### 일반 함수로서 호출할 때 그 함수 내부에서의 this

- 어떤 함수를 일반 함수로서 호출할 경우 호출 주체의 정보를 알 수 없기 때문에 this가 지정되지 않음
  - this가 지정되지 않은 경우 this는 전역 객체를 바라보므로, 함수에서의 this는 **전역 객체**를 가리킴

<br>

### 메서드 내부 함수에서의 this를 우회하는 방법

#### 변수 활용

- _this, that, _, **self** 등의 이름이 쓰임

```js
var obj = {
  outer: function () {
    var self = this;
    var innerFunc = function () {
      console.log(self);
    };
    innerFunc();
  },
};
obj.outer();
```

#### this를 바인딩하지 않는 화살표 함수 사용

- ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자 this를 바인딩하지 않는 화살표 함수를 새로 도입
- 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하지 않음
  - 따라서 화살표 함수 내부에는 this가 없음
  - 때문에 스코프 체인에 따라 가장 가까운 상위 스코프의 this를 그대로 사용
  - 또한 this를 바인딩하지 않으므로 call, apply, bind 등의 메서드를 사용하더라도 화살표 함수 내부의 this를 교체할 수 없음

```js
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = () => {
      console.log(this);
    };
    innerFunc();
  },
};
obj.outer();
```

#### call, apply, bind 사용

_밑에서 다룸_

<br>

### 콜백 함수 호출 시 그 함수 내부에서의 this

```js
// (1)
setTimeout(function () {
  console.log(this);
}, 300);

// (2)
[1, 2, 3, 4, 5].forEach(function (x) {
  console.log(this, x);
});

// (3)
document.body.innerHTML += `<button id="a">클릭</button>`;
document.body.querySelector("#a").addEventListener("click", function (e) {
  console.log(this, e);
});
```

- (1)의 setTimeout 함수와 (2)의 forEach 메서드는 콜백 함수 호출 시 this를 지정하지 않음
  - ∴ 콜백 함수 내부에서의 this는 전역객체를 참조
- (3)의 addEventListener 메서드는 콜백 함수 호출 시 자신의 this를 상속하도록 정의되어 있음
  - ∴ 메서드명의 점 앞부분, 즉 버튼 엘리먼트가 this가 됨
- 콜백 함수에서의 this는 콜백 함수 제어권을 가지는 함수(메서드)에게 this를 무엇으로 할지 권한이 있기 때문에 무엇이다라고 정의할 수 없음
  - 대신 this를 정의하지 않은 경우, 기본적으로 전역객체를 바라봄

#### 별도의 인자로 this를 받는 경우

- 콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체를 인자로 지정할 수 있는 경우가 있음
  - 주로 내부 요소에 대해 같은 공작을 반복 수행해야 하는 **배열 메서드**에 많이 포진되어 있음
    - 비슷하게, Set, Map 등의 메서드에도 일부 존재

```js
Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(callback[, thisArg])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

<br>

### 생성자 함수 내부에서의 this

- 생성자 함수
  - 구체적인 인스턴스를 만들기 위한 틀
  - new 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작
- 어떤 함수가 생성자 함수로서 호출된 경우 내부에서의 this는 새로 만들어질 인스턴스 자신이 됨

<br>

## 명시적으로 this를 바인딩하는 방법

### call 메서드

```js
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

- 메서드의 호출 주체인 함수를 즉시 실행
- 메서드의 첫 번째 인자 -> this로 바인딩
- 이후의 인자들 -> 호출할 함수의 매개변수

<br>

### apply 메서드

```js
Function.prototype.apply(thisArg[, argsArray])
```

- call과 동일
- 단, 두 번째 인자부터는 인자들을 배열 또는 유사배열 객체로 받는다는 점이 다름

<br>

### call, apply 활용

```js
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}

function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}

function Employee(name, gender, company) {
  Person.apply(this, [name, gender]);
  this.company = company;
}

var by = new Student("보영", "female", "단국대");
var jn = new Employee("재난", "male", "구골");
```

- by -> `Student { name: '보영', gender: 'female', school: '단국대' }`
- jn -> `Employee { name: '재난', gender: 'male', company: '구골' }`

<br>

### bind 메서드

```js
Function.prototype.bind(thisArg[, arg1[, arg2]])
```

- 메서드의 호출 주체인 함수를 즉시 실행하지 않고 넘겨받은 this와 인수들을 바탕으로 새롭게 함수를 구성하여 반환
- 메서드의 첫 번째 인자 -> this로 바인딩
- 이후의 인자들 -> 호출할 함수의 매개변수
- 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 것이 모두 가능

```js
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};

var bindFunc = func.bind({ x: 1 }, 1, 2);
bindFunc(3, 4); // { x: 1 }, 1, 2, 3, 4
bindFunc(5, 6); // { x: 1 }, 1, 2, 5, 6
```

#### name 프로퍼티

- bind된 함수의 name을 출력해보면 'bound'라는 접두어가 붙음

  ```js
  var func = function () {
    console.log(this);
  };
  var bindFunc = func.bind({ x: 1 });

  console.log(func.name); // func
  console.log(bindFunc); // bound func
  ```

<br>

_밑에는 테코톡에서 다뤘던 내용~_

<br>

## 규칙 사이의 우선순위

- 위에서 본 네 가지 규칙
  1. 일반 함수 호출
  2. 메서드 호출
  3. 생성자 함수 호출
  4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출

<br>

### 우선순위

```
생성자 함수 호출
→ apply/call/bind 메서드에 의한 간접 호출
→ 메서드 호출
→ 일반 함수 호출
```

<br>

## 클래스에서의 this

- 클래스 내부는 암묵적으로 **엄격 모드**
  - 일반 메서드 선언 시에는 this로 **인스턴스 객체**가 바인딩
  - 중첩 함수나 콜백 함수 등으로 일반 함수 선언 시에는 this로 **undefined**가 바인딩
