## 매핑과 주소

## 주소

이더리움 블록체인은 은행 계좌와 같은 accounts로 이루어짐.

계정은 이더리움 블록체인 상의 통화인 Eth의 잔액을 가짐.

자네의 은행 계좌에서 다른 계좌로 돈을 송금할 수 있듯이, 계정을 통해 다른 계정과 이더를 주고 받을 수 있다.

각 계정은 은행 계좌 번호와 같은 주소를 가지고 있다. 주소는 특정 계정을 가리키는 고유 식별자로, 다음과 같이 표현됨.

```solidity
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```

\* 주소는 특정 유저(혹은 스마트 컨트랙트)가 소유한다 라는 점만 이해하면됨.

## 매핑

매핑은 기본적으로 키-값 (key-value) 저장소로, 데이터를 저장하고 검색하는 데 이용된다.
```
// 금융 앱용으로, 유저의 계좌 잔액을 보유하는 uint를 저장한다: 
mapping (address => uint) public accountBalance;
// 혹은 userID로 유저 이름을 저장/검색하는 데 매핑을 쓸 수도 있다 
mapping (uint => string) userIdToName;
```

## Msg.sender
솔리디티에는 모든 함수에서 이용 가능한 특정 전역 변수들이 있다. 
그 중의 하나가 현재 함수를 호출한 사람 (혹은 스마트 컨트랙트)의 주소를 가리키는 msg.sender

> 솔리디티에서 함수 실행은 항상 외부 호출자가 시작한다. 
컨트랙트는 누군가가 컨트랙트의 함수를 호출할 때까지 블록체인 상에서 아무 것도 안 하고 있을 것이다.
그러니 항상 msg.sender가 있어야한다.

example)
```solidity
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // `msg.sender`에 대해 `_myNumber`가 저장되도록 `favoriteNumber` 매핑을 업데이트한다 `
  favoriteNumber[msg.sender] = _myNumber;
  // ^ 데이터를 저장하는 구문은 배열로 데이터를 저장할 떄와 동일하다 
}

function whatIsMyNumber() public view returns (uint) {
  // sender의 주소에 저장된 값을 불러온다 
  // sender가 `setMyNumber`을 아직 호출하지 않았다면 반환값은 `0`이 될 것이다
  return favoriteNumber[msg.sender];
}
```
누구나 setMyNumber을 호출하여 본인의 주소와 연결된 우리 컨트랙트 내에 uint를 저장할 수 있다.

\* msg.sender를 활용하면 이더리움 블록체인의 보안성을 이용할 수 있게 된다. 
즉, 누군가 다른 사람의 데이터를 변경하려면 해당 이더리움 주소와 관련된 개인키를 훔치는 것 밖에는 다른 방법이 없다

## Require
require를 활용하면 특정 조건이 참이 아닐 때 함수가 에러 메시지를 발생하고 실행을 멈추게 된다.

```solidity
function sayHiToVitalik(string _name) public returns (string) {
  // _name이 "Vitalik"인지 비교한다. 참이 아닐 경우 에러 메시지를 발생하고 함수를 벗어난다
  // (참고: 솔리디티는 고유의 스트링 비교 기능을 가지고 있지 않기 때문에 
  // 스트링의 keccak256 해시값을 비교하여 스트링 값이 같은지 판단한다)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 참이면 함수 실행을 진행한다:
  return "Hi!";
}
```

## 상속

```solidity
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
BabyDoge 
```

## import

다수의 파일이 있고 어떤 파일을 다른 파일로 불러오고 싶을 때, 솔리디티는 import라는 키워드를 이용

```solidity
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

## Storage vs Memory
변수를 저장할 수 있는 공간으로 storage와 memory 두 가지가 존재.

- Storage : 블록체인 상에 영구적으로 저장되는 변수를 의미

- Memory : 임시적으로 저장되는 변수로, 컨트랙트 함수에 대한 외부 호출들이 일어나는 사이에 지워지지. 두 변수는 각각 컴퓨터 하드 디스크와 RAM과 같음

\* 대부분의 경우에 자네는 이런 키워드들을 이용할 필요가 없네. 왜냐면 솔리디티가 알아서 처리해 줌.

상태 변수(함수 외부에 선언된 변수)는 초기 설정상 storage로 선언되어 블록체인에 영구적으로 저장되는 반면
, 함수 내에 선언된 변수는 memory로 자동 선언되어서 함수 호출이 종료되면 사라짐

\* 하지만 이 키워드들을 사용해야 하는 때가 있음. 바로 함수 내의 구조체와 _배열_을 처리할 떄

```solidity
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 꽤 간단해 보이나, 솔리디티는 여기서 
    // `storage`나 `memory`를 명시적으로 선언해야 한다는 경고 메시지를 발생한다. 
    // 그러므로 `storage` 키워드를 활용하여 다음과 같이 선언해야 한다:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...이 경우, `mySandwich`는 저장된 `sandwiches[_index]`를 가리키는 포인터이다.
    // 그리고 
    mySandwich.status = "Eaten!";
    // ...이 코드는 블록체인 상에서 `sandwiches[_index]`을 영구적으로 변경한다. 

    // 단순히 복사를 하고자 한다면 `memory`를 이용하면 된다: 
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...이 경우, `anotherSandwich`는 단순히 메모리에 데이터를 복사하는 것이 된다. 
    // 그리고 
    anotherSandwich.status = "Eaten!";
    // ...이 코드는 임시 변수인 `anotherSandwich`를 변경하는 것으로 
    // `sandwiches[_index + 1]`에는 아무런 영향을 끼치지 않는다. 그러나 다음과 같이 코드를 작성할 수 있다: 
    sandwiches[_index + 1] = anotherSandwich;
    // ...이는 임시 변경한 내용을 블록체인 저장소에 저장하고자 하는 경우이다.
  }
}
```

## 함수 접근 제어자 더 알아보기

### Internal 과 External
private 트랙트를 상속하는 어떤 컨트랙트도 이 함수에 접근할 수 없다는 뜻.

public과 private 이외에도 솔리디티에는 internal과 external이라는 함수 접근 제어자가 있다.

- internal
internal은 함수가 정의된 컨트랙트를 상속하는 컨트랙트에서도 접근이 가능하다 점을 제외하면 private과 동일

- external
함수가 컨트랙트 바깥에서만 호출될 수 있고 컨트랙트 내의 다른 함수에 의해 호출될 수 없다는 점을 제외하면 public과 동일

```solidity
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // eat 함수가 internal로 선언되었기 때문에 여기서 호출이 가능하다 
    eat();
  }
}
```

### 다른 Contract와 상호작용하기

블록체인 상에 있으면서 우리가 소유하지 않은 컨트랙트와 우리 컨트랙트가 상호작용을 하려면 우선 인터페이스를 정의해야 함.

```solidity
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

이 컨트랙트는 아무나 자신의 행운의 수를 저장할 수 있는 간단한 컨트랙트이고, 각자의 이더리움 주소와 연관이 있을 것이다. 
이 주소를 이용해서 누구나 그 사람의 행운의 수를 찾아 볼 수 있다.

약간 다르지만, 인터페이스를 정의하는 것이 컨트랙트를 정의하는 것과 유사하다는 걸 참고.
```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

먼저, 다른 컨트랙트와 상호작용하고자 하는 함수만을 선언할 뿐(이 경우, getNum이 바로 그러한 함수이지) 다른 함수나 상태 변수를 언급하지 않는다.

그러니 인터페이스는 컨트랙트 뼈대처럼 보인다고 할 수 있다. 컴파일러도 그렇게 인터페이스를 인식함.

우리의 dapp 코드에 이런 인터페이스를 포함하면 컨트랙트는 다른 컨트랙트에 정의된 함수의 특성, 호출 방법, 예상되는 응답 내용에 대해 알 수 있다.

## 인터페이스 활용하기

아래와 같이 인터페이스가 정의되면,
```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

다음과 같이 컨트랙트에서 인터페이스를 이용할 수 있다.
```solidity
contract MyContract {
  address NumberInterfaceAddress = 0xab38...
  // ^ 이더리움상의 FavoriteNumber 컨트랙트 주소이다
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress)
  // 이제 `numberContract`는 다른 컨트랙트를 가리키고 있다.

  function someFunction() public {
    // 이제 `numberContract`가 가리키고 있는 컨트랙트에서 `getNum` 함수를 호출할 수 있다:
    uint num = numberContract.getNum(msg.sender);
    // ...그리고 여기서 `num`으로 무언가를 할 수 있다
  }
}
```

## 다수의 반환값 처리하기

```solidity
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 다음과 같이 다수 값을 할당한다:
  (a, b, c) = multipleReturns();
}

// 혹은 단 하나의 값에만 관심이 있을 경우: 
function getLastReturnValue() external {
  uint c;
  // 다른 필드는 빈칸으로 놓기만 하면 된다: 
  (,,c) = multipleReturns();
}
```

## IF문

```solidity
function eatBLT(string sandwich) public {
  // 스트링 간의 동일 여부를 판단하기 위해 keccak256 해시 함수를 이용해야 한다는 것을 기억하자 
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```

## chap2
```javascript
var abi = /* abi generated by the compiler */
var ZombieFeedingContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFeeding = ZombieFeedingContract.at(contractAddress)

// 우리 좀비의 ID와 타겟 고양이 ID를 가지고 있다고 가정하면 
let zombieId = 1;
let kittyId = 1;

// 크립토키티의 이미지를 얻기 위해 웹 API에 쿼리를 할 필요가 있다. 
// 이 정보는 블록체인이 아닌 크립토키티 웹 서버에 저장되어 있다.
// 모든 것이 블록체인에 저장되어 있으면 서버가 다운되거나 크립토키티 API가 바뀌는 것이나 
// 크립토키티 회사가 크립토좀비를 싫어해서 고양이 이미지를 로딩하는 걸 막는 등을 걱정할 필요가 없다 ;) 
let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
$.get(apiUrl, function(data) {
  let imgUrl = data.image_url
  // 이미지를 제시하기 위해 무언가를 한다 
})

// 유저가 고양이를 클릭할 때:
$(".kittyImage").click(function(e) {
  // 우리 컨트랙트의 `feedOnKitty` 메소드를 호출한다 
  ZombieFeeding.feedOnKitty(zombieId, kittyId)
})

// 우리의 컨트랙트에서 발생 가능한 NewZombie 이벤트에 귀를 기울여서 이벤트 발생 시 이벤트를 제시할 수 있도록 한다: 
ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  // 이 함수는 레슨 1에서와 같이 좀비를 제시한다: 
  generateZombie(result.zombieId, result.name, result.dna)
})
```