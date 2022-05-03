## Contract의 불변성
지금까지 본 것만으로는, 솔리디티는 자바스크립트 같은 다른 언어와 꽤 비슷해보였을 것이다. 
하지만 이더리움 DApp에는 일반적인 애플리케이션과는 다른 여러가지 특징이 있다.

1. 이더리움에 Contract를 배포하고 나면, Contract는 변하지 않는다(Immutable). 
-> 컨트랙트를 수정하거나 업데이트할 수 없다

Contract로 배포한 최초의 코드는 항상, 블록체인에 영구적으로 존재. 이것이 바로 솔리디티에 있어서 보안이 굉장히 큰 이슈인 이유. 
만약 Contract Code에 결점이 있다면, 그것을 이후에 고칠 수 있는 방법이 전혀 없다 사용자들에게 결점을 보완한 다른 스마트 컨트랙트 주소를 쓰라고 말하고 다녀야함.

그러나 이것 또한 Smart Contract의 한 특징이다. 코드는 곧 법. 어떤 Smart Contract의 코드를 읽고 검증을 했다면, 
함수를 호출할 때마다, 코드에 쓰여진 그대로 함수가 실행될 것이라고 확신할 수 있다. 그 누구도 배포 이후에 함수를 수정하거나 예상치 못한 결과를 발생시키지 못한다

## 외부 의존성
우리는 크립토키티 컨트랙트의 주소를 우리 DApp에 직접 써넣었다.
그런데 만약 크립토키티 컨트랙트에 버그가 있었고, 누군가 모든 고양이들을 파괴해버렸다면 어떻게 될 것 인가?

그럴 일은 잘 없겠지만, 만약 그런 일이 발생한다면 우리의 DApp은 완전히 쓸모가 없어진다.
우리 DApp은 주소를 코드에 직접 써넣기 때문에 어떤 고양이들도 받아올 수 없다.

우리 좀비들은 고양이를 먹을 수 없을 것이고, 우리는 그걸 고치기 위해 우리의 컨트랙트를 수정할 수도 없을 것이다.

이런 이유로, 대개의 경우 DApp의 중요한 일부를 수정할 수 있도록 하는 함수를 만들어놓는 것이 합리적이다.

예를 들자면 우리 DApp에 크립토키티 컨트랙트 주소를 직접 써넣는 것 대신, 언젠가 크립토키티 컨트랙트에 문제가 생기면 해당 주소를 바꿀 수 있도록 해주는 setKittyContractAddress 함수를 만들 수 있을 것

```solidity
ontract ZombieFeeding is ZombieFactory {
  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  KittyInterface kittyContract = KittyInterface(ckAddress);
  ...
}
```

```solidity
contract ZombieFeeding is ZombieFactory {

  KittyInterface kittyContract;

  function setKittyContractAddress(address _address) external {
    kittyContract = KittyInterface(_address);
  }
  ...
}
```

## 소유 가능한 Contract

setKittyContractAddress 함수는 external이라, 누구든 이 함수를 호출할 수 있다.
이는 아무나 이 함수를 호출해서 크립트키티 Contract의 주소를 바꿀 수 있고, 모든 사용자를 대상으로 우리 앱을 무용지물로 만들 수 있다

우리는 우리 Contract에서 이 주소를 바꿀 수 있게끔 하고 싶지만, 그렇다고 모든 사람이 주소를 업데이트할 수 있기를 원하지는 않다.
이런 경우에 대처하기 위해서, 최근에 주로 쓰는 하나의 방법은 Contract를 소유 가능하게 만드는 것이다.
-> Contract를 대상으로 특별한 권리를 가지는 소유자가 있음을 의미

## OpenZeppelin의 Ownable Contract

아래는 OpenZeppelin 솔리디티 라이브러리에서 가져온 Ownable 컨트랙트이다.

OpenZeppelin은 DApp에서 사용할 수 있는, 안전하고 커뮤니티에서 검증받은 스마트 컨트랙트의 라이브러리이다.
[https://openzeppelin.com/](https://openzeppelin.com/)

```solidity
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}
```

- 생성자(Constructor) 
  function Ownable()는 생성자이다.
  Contract의와 동일한 이름을 가진,생략할 수 있는 특별한 함수로, 이 함수는 Contract의가 생성될 때 딱 한 번만 실행.

- 함수 제어자(Function Modifier)
  modifier onlyOwner()
  제어자는 다른 함수들에 대한 접근을 제어하기 위해 사용되는 일종의 유사 함수.
  보통 함수 실행 전의 요구사항 충족 여부를 확인하는 데에 사용
  onlyOwner의 경우에는 접근을 제한해서 오직 Contract의 소유자만 해당 함수를 실행할 수 있도록 하기 위해 사용될 수 있다.

즉, Ownable Contract는 기본적으로 다음과 같은 것들을 함.

1. Contract가 생성되면 Contract의 생성자가 owner에 msg.sender(Contract를 배포한 사람)를 대입한다.

2. 특정한 함수들에 대해서 오직 소유자만 접근할 수 있도록 제한 가능한 onlyOwner 제어자를 추가한다.

3. 새로운 소유자에게 해당 Contract의 소유권을 옮길 수 있도록 한다.

onlyOwner는 컨트랙트에서 흔히 쓰는 것 중 하나라, 대부분의 솔리디티 DApp들은 Ownable 컨트랙트를 복사/붙여넣기 하면서 시작함.
그리고 첫 컨트랙트는 이 컨트랙트를 상속해서 만듦.

## onlyOwner 함수 제어자

### 함수제어자

함수 제어자는 함수처럼 보이지만, function 키워드 대신 modifier 키워드를 사용한다
그리고 함수를 호출하듯이 직접 호출할 수는 없다.

대신에 함수 정의부 끝에 해당 함수의 작동 방식을 바꾸도록 제어자의 이름을 붙일 수 있다.
```solidty
/**
 * @dev Throws if called by any account other than the owner.
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
```

이 제어자를 다음과 같이 사용할 것이다.
```solidty
contract MyContract is Ownable {
  event LaughManiacally(string laughter);

  // 아래 `onlyOwner`의 사용 방법을 잘 보게:
  function likeABoss() external onlyOwner {
    LaughManiacally("Muahahahaha");
  }
}
```
likeABoss 함수를 호출하면, onlyOwner의 코드가 먼저 실행되네. 그리고 onlyOwner의 _; 부분을 likeABoss 함수로 되돌아가 해당 코드를 실행하게 됨.
제어자를 사용할 수 있는 다양한 방법이 있지만, 가장 일반적으로 쓰는 예시 중 하나는 함수 실행 전에 require 체크를 넣는 것

onlyOwner의 경우에는, 함수에 이 제어자를 추가하면 오직 Contract의 소유자(자네가 배포했다면 자네겠지)만이 해당 함수를 호출할 수 있다.

> 참고 : 이렇게 소유자가 Contract에 특별한 권한을 갖도록 하는 것은 자주 필요하지만, 이게 악용될 수도 있다.
예를 들어, 소유자가 다른 사람의 좀비를 뺏어올 수 있도록 하는 백도어 함수를 추가할 수도 있다.

그러니 잘 기억해야한다. 이더리움에서 돌아가는 DApp이라고 해서 그것만으로 분산화되어 있다고 할 수는 없다.
반드시 전체 소스 코드를 읽어보고, 자네가 잠재적으로 걱정할 만한, 소유자에 의한 특별한 제어가 불가능한 상태인지 확인해야 한다.

개발자로서는 자네가 잠재적인 버그를 수정하고 DApp을 안정적으로 유지하도록 하는 것과, 
사용자들이 그들의 데이터를 믿고 저장할 수 있는 소유자가 없는 플랫폼을 만드는 것 사이에서 균형을 잘 잡는 것이 중요

## 가스
또 다른 솔리디티와 다른 프로그래밍 언어들의 차이점

### 가스 - 이더리움 Dapp이 사용하는 연료
솔리디티에서는 사용자들이 자네가 만든 DApp의 함수를 실행할 때마다 _가스_라고 불리는 화폐를 지불해야 함.

사용자는 이더(ETH, 이더리움의 화폐)를 이용해서 가스를 사기 때문에, 자네의 DApp 함수를 실행하려면 사용자들은 ETH를 소모해야만 한다.

함수를 실행하는 데에 얼마나 많은 가스가 필요한지는 그 함수의 로직(논리 구조)이 얼마나 복잡한지에 따라 달라진다.
각각의 연산은 소모되는 가스 비용(gas cost)이 있고, 그 연산을 수행하는 데에 소모되는 컴퓨팅 자원의 양이 이 비용을 결정한다.

\* 예를 들어, storage에 값을 쓰는 것은 두 개의 정수를 더하는 것보다 훨씬 비용이 높다
함수의 전체 가스 비용은 그 함수를 구성하는 개별 연산들의 가스 비용을 모두 합친 것과 같다.

함수를 실행하는 것은 자네의 사용자들에게 실제 돈을 쓰게 하기 때문에, 이더리움에서 코드 최적화는 다른 프로그래밍 언어들에 비해 훨씬 더 중요하다
만약 코드가 엉망이라면, 사용자들은 함수를 실행하기 위해 일종의 할증료를 더 내야 한다. 그리고 수천 명의 사용자가 이런 불필요한 비용을 낸다면 할증료가 수십 억 원까지 쌓일 수 있다


### 가스는 왜 필요한가?

이더리움은 크고 느린, 하지만 굉장히 안전한 컴퓨터와 같다고 할 수 있다.
어떤 함수를 실행할 때, 네트워크상의 모든 개별 노드가 함수의 출력값을 검증하기 위해 그 함수를 실행해야 한다.

모든 함수의 실행을 검증하는 수천 개의 노드가 바로 이더리움을 분산화하고, 데이터를 보존하며 누군가 검열할 수 없도록 하는 요소이다.

이더리움을 만든 사람들은 누군가가 무한 반복문을 써서 네트워크를 방해하거나, 자원 소모가 큰 연산을 써서 네트워크 자원을 모두 사용하지 못하도록 만들길 원했다. 
그래서 그들은 연산 처리에 비용이 들도록 만들었고, 사용자들은 저장 공간 뿐만 아니라 연산 사용 시간에 따라서도 비용을 지불해야 한다

> 참고: 사이드체인에서는 반드시 이렇지는 않다.

이더리움 메인넷에서 월드 오브 워크래프트 같은 게임을 직접적으로 돌리는 것은 절대 말이 되지 않을 것이다.
가스 비용이 엄청나게 높을 것이기 때문이지. 하지만 다른 합의 알고리즘을 가진 사이드체인에서는 가능할 수 있다.

### 가스를 아끼기 위한 구조체 압축
uint에 다른타입들 uint8, uint16, uint가 있지만,
기본적으로는 이런 하위 타입들을 쓰는 것은 아무런 이득이 없다. 왜냐하면 솔리디티에서는 uint의 크기에 상관없이 256비트의 저장 공간을 미리 잡아놓기 때문이다.

\* 예를 들자면, uint(uint256) 대신에 uint8을 쓰는 것은 가스 소모를 줄이는 데에 아무 영향이 없다.

하지만 여기에 예외가 하나 있다. 바로 struct의 안

만약 구조체 안에 여러 개의 uint를 만든다면, 가능한 더 작은 크기의 uint를 써야함.
솔리디티에서 그 변수들을 더 적은 공간을 차지하도록 압축할 것

```solidity
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini`는 구조체 압축을 했기 때문에 `normal`보다 가스를 조금 사용할 것이네.
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```

이런 이유로, 구조체 안에서는 가능한 한 작은 크기의 정수 타입을 쓰는 것이 좋다.
또한 동일한 데이터 타입은 하나로 묶어놓는 것이 좋다. 즉, 구조체에서 서로 옆에 있도록 선언하면 솔리디티에서 사용하는 저장 공간을 최소화한다.
예를 들면, uint c; uint32 a; uint32 b;라는 필드로 구성된 구조체가 uint32 a; uint c; uint32 b; 필드로 구성된 구조체보다 가스를 덜 소모한다.
uint32 필드가 묶여있기 때문

## 시간 단위 (Time units)

솔리디티는 시간을 다룰 수 있는 단위계를 기본적으로 제공

now 변수를 쓰면 현재의 유닉스 타임스탬프(1970년 1월 1일부터 지금까지의 초 단위 합) 값을 얻을 수 있다.

> 참고 : 유닉스 타임은 전통적으로 32비트 숫자로 저장됨. 

이는 유닉스 타임스탬프 값이 32비트로 표시가 되지 않을 만큼 커졌을 때 많은 구형 시스템에 문제가 발생할 "Year 2038" 문제를 일으킬 것
그러니 만약 우리 DApp이 지금부터 20년 이상 운영되길 원한다면, 우리는 64비트 숫자를 써야 할 것이다.
하지만 우리 유저들은 그동안 더 많은 가스를 소모해야 하는 문제 존재. 설계를 보고 결정을 해야 함.

솔리디티는 또한 seconds, minutes, hours, days, weeks, years 같은 시간 단위 또한 포함하고 있음.

이들은 그에 해당하는 길이 만큼의 초 단위 uint 숫자로 변환됨.
example)
1 minutes = 60
1 hours = 3600
1 days = 86400

```
uint lastUpdated;

// `lastUpdated`를 `now`로 설정
function updateTimestamp() public {
  lastUpdated = now;
}

// 마지막으로 `updateTimestamp`가 호출된 뒤 5분이 지났으면 `true`를, 5분이 아직 지나지 않았으면 `false`를 반환
function fiveMinutesHavePassed() public view returns (bool) {
  return (now >= (lastUpdated + 5 minutes));
}
```
> 참고 : now가 기본적으로 uint256을 반환하기 때문에, uint32(...)로 변환이 필수적이다.
이렇게 함으로써 해당 데이터를 uint32로 명시적으로 변환함.
example ) uint32(now + 1 days)

##구조체를 인수로 전달하기
 private 또는 internal 함수에 인수로서 구조체의 storage 포인터를 전달할 수 있다.
 ```solidity
 function _doStuff(Zombie storage _zombie) internal {
  // _zombie로 할 수 있는 것들을 처리
}
 ```
함수에 좀비 ID를 전달하고 좀비를 찾는 대신, 좀비에 대한 참조를 전달할 수 있다.

## Public 함수 & 보안
모든 public과 external 함수를 검사 필요

함수들이 onlyOwner 같은 제어자를 갖지 않는 이상, 어떤 사용자든 이 함수들을 호출하고 자신들이 원하는 모든 데이터를 함수에 전달할 수 있다.

남용을 막을 가장 쉬운 방법은 internal로 만드는 것

## 함수 제어자의 또 다른 특징

### 인수를 가지는 함수 제어자

```solidity
// 사용자의 나이를 저장하기 위한 매핑
mapping (uint => uint) public age;

// 사용자가 특정 나이 이상인지 확인하는 제어자
modifier olderThan(uint _age, uint _userId) {
  require (age[_userId] >= _age);
  _;
}

// 차를 운전하기 위햐서는 16살 이상이어야 하네(적어도 미국에서는).
// `olderThan` 제어자를 인수와 함께 호출하려면 이렇게 하면 되네:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // 필요한 함수 내용들
}
```

## 'VIew' 함수를 사용해 가스 절약하기

함수가 블록체인에서 데이터를 읽기만 하면, view 함수로 만들 수 있다. 이 부분은 가스 최적화를 말할 때 가장 중요한 내용이다

### View 함수는 가스를 소모하지 않는다.

view 함수는 사용자에 의해 외부에서 호출되었을 때 가스를 전혀 소모하지 않는다.
이건 view 함수가 블록체인 상에서 실제로 어떤 것도 수정하지 않기 떄문 - 데이터를 읽기만 함.

그러니 함수에 view 표시를 하는 것은 web3.js에 이렇게 말하는 것과 같다. 
"이 함수는 실행할 때 자네 로컬 이더리움 노드에 질의만 날리면 되고, 블록체인에 어떤 트랜잭션도 만들지 않아"
-> 트랜잭션은 모든 개별 노드에서 실행되어야 하고, 가스를 소모함.

\* 지금은 자네가 사용자들을 위해 DApp의 가스 사용을 최적화하는 비결은 가능한 모든 곳에 읽기 전용의 external view 함수를 쓰는 것이라는 것만 명심

> 참고 : 만약 view 함수가 동일 Contract 내에 있는, view 함수가 아닌 다른 함수에서 내부적으로 호출될 경우, 여전히 가스를 소모할 것
이것은 다른 함수가 이더리움에 트랜잭션을 생성하고, 이는 모든 개별 노드에서 검증되어야 하기 때문
** 그러니 view 함수는 외부에서 호출됐을 때에만 무료.

## Storage는 비싸다.

솔리디티에서 더 비싼 연산 중 하나는 바로 storage를 쓰는 것 - 그중에서도 쓰기 연산

이건 자네가 데이터의 일부를 쓰거나 바꿀 때마다, 블록체인에 영구적으로 기록되기 때문이다.

영원히! 지구상의 수천 개의 노드들이 그들의 하드 드라이브에 그 데이터를 저장해야 하고, 블록체인이 커져가면서 이 데이터의 양 또한 같이 커져간ㄷ.. 그러니 이 연산에는 비용이 든다.

비용을 최소화하기 위해서, 진짜 필요한 경우가 아니면 storage에 데이터를 쓰지 않는 것이 좋다. 이를 위해 때때로는 겉보기에 비효율적으로 보이는 프로그래밍 구성을 할 필요가 있다
-> 어떤 배열에서 내용을 빠르게 찾기 위해, 단순히 변수에 저장하는 것 대신 함수가 호출될 때마다 배열을 memory에 다시 만드는 것처럼

\* 대부분의 프로그래밍 언어에서는, 큰 데이터 집합의 개별 데이터에 모두 접근하는 것은 비용이 비싸다.
하지만, 솔리디티에서는 그 접근이 external view 함수라면 storage를 사용하는 것보다 더 저렴한 방법이다.

view 함수는 사용자들의 가스를 소모하지 않기 때문이지(가스는 사용자들이 진짜 돈을 쓰는 것이다!)

### 메모리에 배열 선언하기

Storage에 아무것도 쓰지 않고도 함수 안에 새로운 배열을 만들려면 배열에 memory 키워드를 쓰면 된다.
이 배열은 함수가 끝날 때까지만 존재할 것이고, 이는 storage의 배열을 직접 업데이트하는 것보다 가스 소모 측면에서 훨씬 저렴하다.
-> 외부에서 호출되는 view 함수라면 무료

```solidity
function getArray() external pure returns(uint[]) {
  // 메모리에 길이 3의 새로운 배열을 생성한다.
  uint[] memory values = new uint[](3);
  // 여기에 특정한 값들을 넣는다.
  values.push(1);
  values.push(2);
  values.push(3);
  // 해당 배열을 반환한다.
  return values;
}
```

> 참고 : 메모리 배열은 반드시 길이 인수와 함께 생성되어야 한다.
메모리 배열은 현재로서는 storage 배열처럼 array.push()로 크기가 조절되지는 않는다.
이후 버전의 솔리디티에서는 변경될수 도 있음. 

## 반복문

함수 내에서 배열을 다룰 때, 그냥 storage에 해당 배열을 저장하는 것이 아니라 for 반복문을 사용해서 구성해야 할 때가 있을 것이다.

```
mapping (address => uint[]) public ownerToZombies;

function getZombiesByOwner(address _owner) external view returns (uint[]) {
  return ownerToZombies[_owner];
}
```
for문을 쓰지 않고 매핑을 사용하면, 이러한 접근 방법은 구현의 간단함 때문에 매력적으로 보인다.

하지만 만약 나중에 한 좀비를 원래 소유자에서 다른 사람에게 전달하는 함수를 구현하게 된다면..?

좀비 전달 함수는 이런 내용이 필요할 것이다

1. 전달할 좀비를 새로운 소유자의 ownerToZombies 배열에 넣는다.
2. 기존 소유자의 ownerToZombies 배열에서 해당 좀비를 지운다.
3. 좀비가 지워진 구멍을 메우기 위해 기존 소유자의 배열에서 모든 좀비를 한 칸씩 움직인다.
4. 배열의 길이를 1 줄인다.

3번째 단계는 극단적으로 가스 소모가 많을 것이다. 왜냐하면 위치를 바꾼 모든 좀비에 대해 쓰기 연산을 해야 하기 때문이다.
-> 소유자가 20마리의 좀비를 가지고 있고 첫 번째 좀비를 거래한다면, 배열의 순서를 유지하기 위해 우린 19번의 쓰기를 해야 할 것

솔리디티에서 storage에 쓰는 것은 가장 비용이 높은 연산 중 하나이기 때문에, 이 전달 함수에 대한 모든 호출은 가스 측면에서 굉장히 비싸게 될 것
더 안 좋은 점은, 이 함수가 실행될 때마다 다른 양의 가스를 소모할 것이라는 점.
사용자가 자신의 군대에 얼마나 많은 좀비를 가지고 있는지, 또 거래되는 좀비의 인덱스에 따라 달라짐. 즉 사용자들은 거래에 가스를 얼마나 쓰게 될지 알 수 없게 됨

> 참고: 물론, 빈 자리를 채우기 위해 마지막 좀비를 움직인 다음, 배열의 길이를 하나 줄여도 되겠지. 하지만 그렇게 하면 교환이 일어날 때마다 좀비 군대의 순서가 바뀌게 될 것

view 함수는 외부에서 호출될 때 가스를 사용하지 않기 때문에, 우린 getZombiesByOwner 함수에서 for 반복문을 사용해서 좀비 배열의 모든 요소에 접근한 후 
특정 사용자의 좀비들로 구성된 배열을 만들 수 있을 것.

그러고 나면, transfer 함수는 훨씬 비용을 적게 쓰게 될 것이다.
왜냐하면 storage에서 어떤 배열도 재정렬할 필요가 없기 떄문이다.
-> 일반적인 직관과는 반대로 이런 접근법이 전체적으로 비용 소모가 더 적다

### for 반복문 사용하기
```
// 이 함수는 [2, 4, 6, 8, 10]를 가지는 배열을 반환
function getEvens() pure external returns(uint[]) {
  uint[] memory evens = new uint[](5);
  // 새로운 배열의 인덱스를 추적하는 변수
  uint counter = 0;
  // for 반복문에서 1부터 10까지 반복함
  for (uint i = 1; i <= 10; i++) {
    // `i`가 짝수라면...
    if (i % 2 == 0) {
      // 배열에 i를 추가함
      evens[counter] = i;
      // `evens`의 다음 빈 인덱스 값으로 counter를 증가시킴
      counter++;
    }
  }
  return evens;
}
```

### payable

#### 함수 제어자(function modifier) 정리
함수가 언제, 어디서 호출될 수 있는지 제어하는 접근 제어자(visibility modifier)

- private : Contract 내부의 다른 함수들에서만 호출될 수 있음을 의미
- internal : private과 비슷하지만, 해당 컨트랙트를 상속하는 컨트랙트에서도 호출될 수 있음.
- external : 오직 컨트랙트 외부에서만 호출될 수 있다.
- public : 내외부 모두에서, 어디서든 호출될 수 있다.

#### 상태 제어자(state modifier) 정리

이 제어자는 블록체인과 상호작용 하는 방법에 대해 알려줌.

- view : 해당 함수를 실행해도 어떤 데이터도 저장/변경되지 않음
- pure : 해당 함수가 어떤 데이터도 블록체인에 저장하지 않을 뿐만 아니라, 블록체인으로부터 어떤 데이터도 읽지 않음

* 이들 모두는 Contract 외부에서 불렸을 때 가스를 전혀 소모하지 않는다 (하지만 다른 함수에 의해 내부적으로 호출됐을 경우에는 가스를 소모).

#### 정의 제어자 정리

- onlyOwner : 함수에 이 제어자들이 어떻게 영향을 줄지를 결정하는 우리만의 논리를 구성할 수 있다.

example)
```solidity
function test() external view onlyOwner anotherModifier { /* ... */ }
```

#### payable 제어자
payable 함수는 솔리디티와 이더리움을 아주 멋지게 만드는 것 중 하나
-> 이더를 받을 수 있는 특별한 함수 유형

자네가 일반적인 웹 서버에서 API 함수를 실행할 때에는, 함수 호출을 통해서 US 달러를 보낼 수 없다. - 비트코인도 보낼 수 없다

하지만 이더리움에서는, 돈(_이더_), 데이터(transaction payload), 
그리고 Contract 코드 자체 모두 이더리움 위에 존재하기 때문에, 함수를 실행하는 동시에 Contract에 돈을 지불하는 것이 가능하다.

이를 통해 굉장히 흥미로운 구성을 만들어낼 수 있네. 함수를 실행하기 위해 컨트랙트에 일정 금액을 지불하게 하는 것과 같이 
example)
```solidity
contract OnlineStore {
  function buySomething() external payable {
    // 함수 실행에 0.001이더가 보내졌는지 확실히 하기 위해 확인:
    require(msg.value == 0.001 ether);
    // 보내졌다면, 함수를 호출한 자에게 디지털 아이템을 전달하기 위한 내용 구성:
    transferThing(msg.sender);
  }
}
```
여기서, msg.value는 Contract로 이더가 얼마나 보내졌는지 확인하는 방법이고, ether는 기본적으로 포함된 단위이다.

여기서 일어나는 일은 누군가 web3.js(DApp의 자바스크립트 프론트엔드)에서 다음과 같이 함수를 실행할 때 발생
```solidity
// `OnlineStore`는 자네의 이더리움 상의 Contract를 가리킨다고 가정:
OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```

value 필드에서. 자바스크립트 함수 호출에서 이 필드를 통해 ether를 얼마나 보낼지 결정함(여기서는 0.001).
트랜잭션을 봉투로 생각하고, 함수 호출에 전달하는 매개 변수를 써넣은 편지의 내용이라 생각한다면, value는 봉투 안에 현금을 넣는 것과 같다 - 편지와 돈이 모두 수령인에게 전달됨

>참고: 만약 함수가 payable로 표시되지 않았는데 위에서 본 것처럼 이더를 보내려 한다면, 함수에서 트랜잭션을 거부할 것이다.

## 출금
Contract로 이더를 보내면, 해당 Contract의 이더리움 계좌에 이더가 저장되고 거기에 갇히게 된다. - Contract부터 이더를 인출하는 함수를 만들지 않는다면 

다음과 같이 Contract에서 이더를 인출하는 함수를 작성할 수 있다.
```solididy
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    owner.transfer(this.balance);
  }
}
```
transfer 함수를 사용해서 이더를 특정 주소로 전달할 수 있다.
그리고 this.balance는 컨트랙트에 저장돼있는 전체 잔액을 반환함.

그러니 100명의 사용자가 우리의 컨트랙트에 1이더를 지불했다면, this.balance는 100이더가 될 것
transfer 함수를 써서 특정한 이더리움 주소에 돈을 보낼 수 있네. 예를 들어, 만약 누군가 한 아이템에 대해 초과 지불을 했다면, 이더를 msg.sender로 되돌려주는 함수를 만들 수도 있다.

```solidity
uint itemFee = 0.001 ether;
msg.sender.transfer(msg.value - itemFee);
```
혹은 구매자와 판매자가 존재하는 컨트랙트에서, 판매자의 주소를 storage에 저장하고, 
누군가 판매자의 아이템을 구매하면 구매자로부터 받은 요금을 그에게 전달할 수도 있다.

```
seller.transfer(msg.value)
```
이런 것들이 이더리움 프로그래밍을 아주 멋지게 만들어주는 예시이다. - 자네는 이것처럼 누구에게도 제어되지 않는 분산 장터들을 만들 수도 있다.

## 난수 (Random Numbers)

일정 수준의 무작위성을 필요로 한다. 그럼 솔리디티에서는 어떻게 난수를 발생 시키겠는가?

진정한 답은, 할 수 없다. 글쎄, 적어도 안전하게 할 수는 없다.

### keccak256을 통한 난수 생성
솔리디티에서 난수를 만들기에 가장 좋은 방법은 keccak256 해시 함수를 쓰는 것

```solidity
// Generate a random number between 1 and 100:
uint randNonce = 0;
uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
randNonce++;
uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```
이 코드는 now의 타임스탬프 값, msg.sender, 증가하는 nonce(딱 한 번만 사용되는 숫자, 즉 똑같은 입력으로 두 번 이상 동일한 해시 함수를 실행할 수 없게 함)를 받고 있다.

그리고서 keccak을 사용하여 이 입력들을 임의의 해시 값으로 변환하고, 변환한 해시 값을 uint로 바꾼 후, % 100을 써서 마지막 2자리 숫자만 받도록 했다.

이를 통해 0과 99 사이의 완전한 난수를 얻을 수 있다.

### 이 메소드는 정직하지 않은 노드의 공격에 취약

이더리움에서는 Contract의 함수를 실행하면 트랜잭션(transaction)으로서 네트워크의 노드 하나 혹은 여러 노드에 실행을 알리게 된다.

그 후 네트워크의 노드들은 여러 개의 트랜잭션을 모으고, "작업 증명"으로 알려진 계산이 매우 복잡한 수학적 문제를 먼저 풀기 위한 시도를 하게 된다.

그리고서 해당 트랜잭션 그룹을 그들의 작업 증명(PoW)과 함께 _블록_으로 네트워크에 배포하게된다.

한 노드가 어떤 PoW를 풀면, 다른 노드들은 그 PoW를 풀려는 시도를 멈추고 해당 노드가 보낸 트랜잭션 목록이 유효한 것인지 검증한다. 
유효하다면 해당 블록을 받아들이고 다음 블록을 풀기 시작한다.

이것이 우리의 난수 함수를 취약하게 만든다.

우리가 동전 던지기 컨트랙트를 사용한다고 해보자 
- 앞면이 나오면 돈이 두 배가 되고, 뒷면이 나오면 모두 다 잃는 것이다. 앞뒷면을 결정할 때 위에서 본 난수 함수를 사용한다고 가정해본다.
(random >= 50은 앞면, random < 50은 뒷면이네).

내가 만약 노드를 실행하고 있다면, 나는 오직 나의 노드에만 트랜잭션을 알리고 이것을 공유하지 않을 수 있다.
그 후 내가 이기는지 확인하기 위해 동전 던지기 함수를 실행할 수 있다 
- 그리고 만약 내가 진다면, 내가 풀고 있는 다음 블록에 해당 트랜잭션을 포함하지 않는 것을 선택한다.
난 이것을 내가 결국 동전 던지기에서 이기고 다음 블록을 풀 때까지 무한대로 반복할 수 있고, 이득을 볼 수 있다.
