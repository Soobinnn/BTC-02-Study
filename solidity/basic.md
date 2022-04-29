# Solidity 기본 문법

> https://cryptozombies.io/ko/course/ 에서 공부한 문법 정리

### contract

solidity 코드는 contract 안에 쌓여 있음. 

contract는 이더리움 어플리케이션의 기본적인 구성 요소, 모든 변수와 함수는 어느 한 contract에 속해있음.

```solidity
contract HelloWorld {

}
```

#### version pragma

모든 solidity 코드는 "version pragma"로 시작해야 함.

해당 코드가 이용해야 하는 solidity 버전을 선언하는 것.
이를 통해 이후에 새로운 컴파일러 버전이 나와도 기존 코드가 깨지지 않도록 예방함.

```solidity
pragma solidity ^0.4.19;

contract HelloWorld {

}
```

#### State Variables & Integers

State Variables는 contract 저장소에 영구적으로 저장됨. 
즉, 이더리움 블록체인에 기록됨. 데이터베이스에 데이터를 쓰는 것과 동일

```solidity
contract Example {
    uint myUnsignedInteger = 100;
}
```

- 부호 없는 정수 : uint
uint 자료형은 부호 없는 정수로, 값이 음수가 아니어야 한다는 의미,
부호 있는 정수를 위한 int 자료형도 있음.

> solidity에서 uint 실제 uint256, 256비트 부호없는 정수의 다른 표현임.
uint8, uint16, uint32 등과 같이 uint를 더 적은 비트로 선언할 수도 있다.
\* 특수한 경우가 아니라면 일반적으로 uint 사용

#### Math Operations

- 덧셈 : x + y
- 뺄셈 : x - y
- 곱셈 : x * y
- 나눗셈 : x / y
- 나머지 x % y
- 지수 연산 
```
5^2 = 25
uint x = 5 ** 2;
```

### Structure

```solidity
struct Person {
    uint age;
    string name;
}
```

### Array
Solidity 에는 정적배열, 동적배열 두 종류 존재

```solidity
// 2개의 원소를 담을 수 있는 고정 길이의 배열
uint[2] fixedArray;

// 또다른 고정 배열으로 5개의 스트링을 담을 수 있다.
string[5] stringArray;

// 동적배열은 고정된 크기가 없으며 계속 크기가 커질 수 있다.
uint[] dynamicArray;

// 구조체에 배열을 생성할 수도 있다.
Person[] people;

// public 배열
// public 배열을 선언할 수 있다. getter 메소드를 자동적으로 생성함.
// 다른 컨트랙트들이 이 배열을 읽을 수 있다. (쓸 수는 없음) 컨트랙트에 공개 데이터를 저장할 때 유용한 패턴
Person[] public people;
```

### Function Declarations

솔리디티에서 함수 선언은 다음과 같이함.
```solidity
function eatHamburgers(String _name, uint _amount) {\
}
```

> 함수 인자명을 언더스코어(_)로 시작해서 전역변수와 구별하는 것이 관례임.

### Working With Structs and Arrays

```solidity
struct Person {
  uint age;
  string name;
}

Person[] public people;

Person satoshi = Person(172, "Satoshi");

people.push(satoshi);

// 한줄로 표현
people.push(Person(16, "Vitalik"));
```

### Private / Public Function

Solidity에서 함수는 기본적으로 public으로 선언됨. 
-> 누구나 (다른 contract가) contract의 함수를 호출하고 코드를 실행할 수 있다.

contract를 공격에 취약하게 만듦. 그러니, 기본적으로 함수롤 private로 선언하고, 공개할 함수만 public으로 선언하는 것이 좋음.

```
uint[] numbers;

function _addToArray(uint _number) private {
    numbers.push(_number);
}
```

private 키워드는 함수명 다음에 적음. 
\* private 함숨여도 언더바(_)로 시작하는 것이 관례

### More on Functions

- Return Values

Solidity에서 함수 선언은 반환값 종류를 포함함.
```solidity
string greeting = "What's up dog";

function sayHello() public returns (string) {
  return greeting;
}
```

- Function modifiers
일반 함수는 상태를 변화시키지 않는다. 즉, 어떤 값을 변경하거나 무언가를 쓰지 않음.

이 경우엔 함수를 view 함수로 선언한다. 이는 함수가 데이터를 보기만 하고 변경하지 않는다는 의미
```
function sayHello() public view returns (string) {
}
```

Solidity는 pure함수도 가지고 있는데, 이는 함수가 앱에서 어떤 데이터도 접근하지 않는 것을 의미
```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```
이 함수는 앱에서 읽는 것도 하지 않고, 다만 반환값이 함수에 전달된 인자값에 따라서 달라짐.

> 참고 : 함수를 pure나 view로 언제 표시할지 기억하기 어려울 수 있지만, Solidity 컴파일러가 어떤 제어자를 써야하는지 경고 메세지를 알려줌.

