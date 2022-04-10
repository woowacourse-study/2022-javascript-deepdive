# 03. this

## 3-1. 전역변수와 전역객체

- 전역변수를 선언하면 JS 엔진은 이를 전역객체의 프로퍼티로 할당한다.

  ```js
  var a = 1;
  delete window.a;                   // false
  console.log(a, window.a, this.a);  // 1 1 1
  
  var b = 2;
  delete b;                          // false                       
  console.log(b, window.b, this.b);  // 2 2 2
  
  window.c = 3;
  delete window.c;                   // true 
  console.log(c, window.c, this.c);  // Uncaurght ReferenceError: c is not defined
  
  window.d = 4;
  delete d;                          // true 
  console.log(d, window.d, this.d);  // Uncaurght ReferenceError: c is not defined
  ```

  - 처음부터 전역객체의 프로퍼티로 할당한 경우에는 삭제가 되는 반면, 전역변수로 선언한 경우에는 삭제가 되지 않음.
  - 왜죠? 전역변수를 선언하면 JS 엔진이 이를 전역객체의 프로퍼티로 할당하면서 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의하기 때문.

## 3-2. this 키워드

- 메서드가 자신이 속한 객체의 프로퍼티를 참조하려면 **자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다.**
- **this는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수.**
- **this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있다.**
- **this가 가리키는 값, 즉 this 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.**

## 3-3. 함수 호출 방식과 this 바인딩

- this 바인딩은 함수 호출 방식에 따라 동적으로 결정된다.

- 렉시컬 스코프와 this 바인딩의 결정 시기

  - 렉시컬 스코프 : 함수 정의가 평가되어 함수 객체가 생성되는 시점에 상위 스코프를 결정.
  - this 바인딩 : 함수 호출 시점에 결정.

- 일반 함수 호출 

  - 기본적으로 this에는 전역 객체가 바인딩. (strict mode에서는 undefined 바인딩)
  - 메서드 내에 정의한 중첩 함수도 일반 함수로 호출되면 중첩 함수 내부의 this에는 전역 객체가 바인딩.
  - 콜백 함수가 일반 함수로 호출된다면 콜백 함수 내부의 this에도 전역 객체가 바인딩.
  - **이처럼 일반 함수로 호출된 모든 함수(중첩 함수, 콜백 함수 포함) 내부의 this에는 전역 객체가 바인딩.**

- 메서드 내에 정의한 중첩 함수 또는 메서드에게 전달한 콜백 함수는 보통 외부 함수를 돕는 헬퍼 함수를 역할을 하기 때문에 전역 객체가 바인딩 되는 것은 문제가 있다. 이를 해결하는 방법은?

  ```js
  var value = 1;
  
  const obj = {
      value: 100,
      foo() {
          // this 바인딩(obj)을 변수 that에 할당
          const that = this;
          
          // 콜백 함수 내부에서 this 대신 that을 참조
          setTimeOut(function () {
              console.log(that.value);
          }, 100);
      }
  };
  
  obj.foo();
  ```

  - 이 방법 외에도 this를 명시적으로 바인딩하는 Function.prototype.apply, Function.prototype.call, Function.prototype.bind 메서드를 제공.
  - 화살표 함수를 사용해서 this 바인딩을 일치시킬 수도 있음. (화살표 함수 내부의 this는 상위 스코프의 this를 가리킴.)

- 메서드 호출

  - 메서드 이름 앞의 마침표 연산자 앞에 기술한 객체가 바인딩.
  - 즉, 메서드를 호출한 객체에 바인딩.

- 생성자 함수 호출

  - 생성자 함수 내부의 this에는 생성자 함수가 (미래에) 생성할 인스턴스가 바인딩.

## 3-4. apply, call, bind

- apply, call, bind 메서드는 Function.prototype의 메서드. 즉, 이들 메서드는 모든 함수가 상속받아 사용 가능.

- **apply와 call 메서드의 본질적인 기능은 함수를 호출하는 것.**

- 함수를 호출하면서 첫 번째 인수로 전달한 특정 객체를 호출한 함수의 this에 바인딩.

  ```js
  function getThisBinding() {
      return this;
  }
  
  // this로 사용할 객체
  const thisArg = { a: 1 };
  
  console.log(getThisBinding()); // window
  
  console.log(getThisBinding.apply(thisArg)); // {a: 1}
  console.log(getThisBinding.call(thisArg)); // {a: 1}
  ```

- apply는 인수를 `배열`로 묶어서 전달(array랑 글자가 비슷하니까 연상해서 외우면 좋을듯?), call은 인수를 `쉼표로 구분하여` 전달.

- arguments 객체와 같은 유사 배열 객체에 배열 메서드를 사용하는 경우, apply와 call을 사용.

- arguments 객체는 배열이 아니기 때문에 Array.prototype.slice 같은 배열의 메서드를 사용할 수 없으나 apply와 call 메서드를 이용하면 사용 가능.

  ```js
  function convertArgToArray() {
      // Array.prototype.slice를 인수 없이 호출하면 배열의 복사본을 생성.
      const arr = Array.prototype.slice.call(arguments);
      // const arr = Array.prototype.slice.apply(arguments);
      
      return arr;
  }
  
  console.log(convertArgToArray(1, 2, 3)); // [1, 2, 3]
  ```

- bind 메서드는 apply와 call 메서드와 달리 함수를 호출하지 않고 this로 사용할 객체만 전달.

  ```js
  function getThisBinding() {
      return this;
  }
  
  const thisArg = { a: 1 };
  
  // 함수 호출 X
  console.log(getThisBinding.bind(thisArg)); // getThisBinding
  // 호출하려면 명시적으로 호출해야 함
  console.log(getThisBinding.bind(thisArg)()); // {a: 1}
  ```

- bind 메서드는 메서드의 this와 메서드 내부의 중첩 함수 또는 콜백 함수의 this가 불일치하는 문제를 해결하기 위해 유용하게 사용됨.

  ```js
  const person = {
      name: 'Mincho',
      foo(callback) {
          setTimeout(callback.bind(this), 100);
      }
  };
  
  person.foo(function () {
      console.log(`my name is ${this.name}`); // my name is Mincho
  })
  ```

## 3-5. 정리

### 호출방식에 따른 정리

| 함수 호출 방식                                             | this 바인딩                                                  |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| 일반 함수 호출                                             | 전역 객체                                                    |
| 메서드 호출                                                | 메서드를 호출한 객체                                         |
| 생성자 함수 호출                                           | 생성자 함수가 (미래에) 생성할 인스턴스                       |
| Function.prototype.apply/call/bind 메서드에 의한 간접 호출 | Function.prototype.apply/call/bind 메서드에 첫 번째 인수로 전달한 객체 |

### 명시적 this 바인딩이 없을 경우

- 전역 공간에서 this는 전역 객체를 참조.
  - 브라우저에서는 window, Node.js에서는 global.
- 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조.
- 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조. 메서드의 내부함수에서도 동일.
- 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, 정의하지 않은 경우에는 전역객체를 참조.
- 생성자 함수에서의 this는 생성될 인스턴스를 참조.

### 명시적 this 바인딩

- call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출.
- bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만듦.
- 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 함.
  - Array : forEach, map, filter, some, every, find, findIndex, flatMap, from 등.
  - Set, Map : forEach

## 3-6. 화살표 함수와 this

- 화살표 함수와 일반 함수의 차이

  - 화살표 함수는 인스턴스를 생성할 수 없는 non-constructor다.
    - 화살표 함수는 인스턴스를 생성할 수 없으므로 prototype 프로퍼티가 없고 프로토타입도 생성하지 않는다.
  - 화살표 함수는 중복된 매개변수 이름을 선언할 수 없다.
    - 일반 함수는 중복된 매개변수 이름을 선언해도 에러가 발생하지 않는다. (단, strict mode에선 에러)
  - 화살표 함수는 함수 자체의 this, arguments, super, new.target 바인딩을 갖지 않는다.
    - 화살표 함수 내부에서 this, arguments, super, new.target을 참조하면 스코프 체인을 통해 상위 스코프의 this, arguments, super, new.target을 참조한다.
    - 만약 화살표 함수와 화살표 함수가 중첩되어 있다면 상위 화살표 함수에도 this, arguments, super, new.target 바인딩이 없으므로, 스코프 체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수의 this, arguments, super, new.target을 참조한다.

- 콜백 함수 내부의 this 불일치 문제 해결.

  - **화살표 함수는 함수 자체의 this 바인딩을 갖지 않는다. **
  - **따라서 화살표 함수 내부에서 this를 참조하면 상위 스코프의 this를 그대로 참조한다.**
  - **이것을 lexcial this라 한다.**

- 화살표 함수 내부에서 this를 참조하면 일반적인 식별자처럼 스코프 체인을 통해 상위 스코프에서 this를 탐색.

  ```js
  // arrow function
  () => this.x;
  
  // 위 화살표 함수와 동일하게 동작.
  (function () { return this.x; }).bind(this);
  ```

- 화살표 함수가 전역 함수라면 화살표 함수의 this는 전역 객체를 가리킴.

  ```js
  const foo = () => console.log(this);
  foo(); // window
  ```

- 프로퍼티에 할당한 화살표 함수 : 스코프체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수의 this를 참조.

  ```js
  // increase 프로퍼티에 할당한 화살표 함수의 상위 스코프는 전역
  // 화살표 함수의 this는 전역 객체를 가리킴.
  
  const counter = {
      num: 1,
      increase: () => ++this.num
  };
  
  console.log(counter.increase()); // NaN
  ```

- 화살표 함수는 함수 자체의 this 바인딩을 갖지 않기 때문에 call, apply, bind 메서드를 사용해도 화살표 함수 내부 this를 교체할 수 없음.

- 메서드를 화살표 함수로 정의하는 것은 피해야한다. 메서드를 정의할 때는 ES6 메서드 축약 표현으로 정의하는 것이 좋음.

  ```js
  // Bad
  // Halee가 나올 것이라 예상하고 작성했지만 this가 전역이므로 window.name 출력.
  const person = {
      name: 'Halee',
      sayHi: () => console.log(`Hi ${this.name}`); 
  }
  person.sayHi(); // Hi
  
  // Bad
  function Person(name) {
      this.name = name;
  }
  
  Person.prototype.sayHi = () => console.log(`Hi ${this.name}`);
  
  const person = new Person('Hope');
  person.sayHi(); // Hi
  
  // Good
  const perseon = {
      name: 'Ahn',
      sayHi () {
          console.log(`Hi ${this.name}`);
      }
  };
  
  person.sayHi(); // Hi Ahn
  
  // Good
  function Person(name) {
      this.name = name;
  }
  
  Person.prototype.sayHi = function () { console.log(`Hi ${this.name}`); );
  
  const person = new Person('Mincho');
  person.sayHi(); // Hi Mincho 
  ```

## 3-7. 주의

### 메서드를 추출할 때 사라지는 this - 1

```js
var counter = {
  count: 0,
  inc: function () {
    this.count++;
  }
}

var func = counter.inc;
func();
counter.count; // 0
```

- count.inc 값을 함수로 호출했으므로 this는 전역 객체.
- func()는 window.count++를 시도한 것과 마찬가지이며, window.count는 존재하지 않으므로 undefined.
- 만일 메서드 inc()가 strirct mode라면 에러를 표시.

### 메서드를 제대로 추출하려면?

```js
var func3 = counter.inc.bind(counter);
func3();

counter.count; // 1
```

### 메서드를 추출할 때 사라지는 this - 2 (콜백)

```js
function callIt(callback) {
  callback();
}

callIt(counter.inc);
// TypeError: Cannot read property 'count' of undefined
```

- 함수로 호출했으므로 this는 전역 객체.

### 메서드를 제대로 추출하려면?

```js
callit(counter.inc.bind(counter));

counter.count; // 2
```

<br>

### 메서드 내부의 함수

```js
var obj = {
  name: 'Jane',
  friends: ['Tarzan', 'Cheeta'],
  loop: function() {
    'user strict';
    this.friends.forEach(
    	function (friend) {
        console.log(this.name + ' knows ' + friend);
      }
    )
  }
}
```

- 개발자가 출력하길 원하는 건 'Jane'인데 undefined가 출력됨!
- 어떻게 해결하나요?

### 방법 1 : that = this

```js
loop: function() {
  'use strict';
  var that = this;
  this.friends.forEach(function (friend) {
    console.log(that.name + ' knows ' + friend);
  })
}
```

### 방법 2 : bind()

```js
loop: function() {
  'use strict';
  this.friends.forEach(function (friend) {
    console.log(that.name + ' knows ' + friend);
  }).bind(this);
}
```

- 주의 : bind는 호출할 때마다 새로운 함수를 새로 생성

### 방법 3 : forEach()에 thisValue를 넘기기

```js
loop: function() {
  'use strict';
  this.friends.forEach(function (friend) {
    console.log(that.name + ' knows ' + friend);
  }, this);
}
```

# 참고

- 코어 자바스크립트

  - `3. this`
- 모던 자바스크립트 Deep Dive

  - ` 22.this`
  - `26.3.3 화살표 함수에서의 this`
- 자바스크립트를 말하다

  - `17.3 함수와 메서드의 묵시적 매개변수인 this`

