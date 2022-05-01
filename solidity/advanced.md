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

