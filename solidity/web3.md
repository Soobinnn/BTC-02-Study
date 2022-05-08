## Web3.js 소개

이더리움 네트워크는 노드로 구성되어 있고, 각 노드는 블록체인의 복사본을 가지고 있다.

스마트 컨트랙트의 함수를 실행하고자 한다면, 이 노드들 중 하나에 질의를 보내 아래 내용을 전달해야 한다.

1. 스마트 컨트랙트의 주소
2. 실행하고자 하는 함수, 그리고
3. 그 함수에 전달하고자 하는 변수들

이더리움 노드들은 JSON-RPC라고 불리는 언어로만 소통할 수 있고, 이는 사람이 읽기는 불편하다. 
컨트랙트의 함수를 실행하고 싶다고 질의를 보내는 것은 이와 같이 생김.
```
{
    "jsonrpc":"2.0",
    "method":"eth_sendTransaction",
    "params":[
        {
            "from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155",
            "to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567",
            "gas":"0x76c0",
            "gasPrice":"0x9184e72a000",
            "value":"0x9184e72a",
            "data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}
    ],
    "id":1
}
```
Web3.js는 이런 골치 아픈 질의를 몰라도 되게 해준다. 편리하고 쉽게 읽을 수 있는 자바스크립트 인터페이스로 상호작용을 하면 되는 것

위의 질의문을 작성할 필요없이, 코드에서 함수를 호출하면 된다.
```
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔")
  .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```

### 시작하기

```
// NPM을 사용할 때
npm install web3

// Yarn을 사용할 때
yarn add web3

// Bower를 사용할 때
bower install web3
```

또는 단순히 github에서 간략화된 .js 파일을 다운로드하고 자네 프로젝트에 포함할 수 있다.
```
<script language="javascript" type="text/javascript" src="web3.min.js"></script>
```

## Web3 프로바이더(Provider)

처음 필요로 하는 것은 Web3 프로바이더(Provider)이다.

이더리움은 똑같은 데이터의 복사본을 공유하는 `노드`들로 구성되어 있다.
Web3.js에서 Web3 프로바이더를 설정하는 것은 우리 코드에 읽기와 쓰기를 처리하려면 어떤 노드와 통신을 해야 하는지 설정하는 것이다.

이는 전통적인 웹 앱에서 API 호출을 위해 원격 웹 서버의 URL을 설정하는 것과 같다.

나만의 이더리움 노드를 프로바이더로 운영할 수도 있다. 하지만 편리하게 쓸 수 있는 제3자 서비스가 있다. 
DApp의 사용자들을 위해 나만의 이더리움 노드를 운영할 필요가 없도록 하기 위해 사용할 수 있는 서비스 - Infura 가 있다.

### Infura
빠른 읽기를 위한 캐시 계층을 포함하는 다수의 이더리움 노드를 운영하는 서비스

접근을 위한 API를 무료로 사용할 수 있다. 
Infura를 프로바이더로 사용하면, 자네만의 이더리움을 설치하고 계속 유지할 필요 없이 이더리움 블록체인과 메세지를 확실히 주고받을 수 있다.

```
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```
하지만, 많은 사용자들이 우리의 DApp을 사용할 것이기에 
그리고 이 사용자들은 단순히 읽기만 하는 게 아니라 블록체인에 뭔가 쓰기도 할 것이기에 
우리는 이 사용자들이 그들의 개인 키로 트랜잭션에 서명을 할 수 있도록 해야 할 것이다

> 참고 : 이더리움(그리고 일반적으로 블록체인)은 트랜잭션에 전자 서명을 하기 위해 공개/개인 키 쌍을 사용한다.

말하자면 전자 서명을 위해 엄청나게 안전한 비밀번호 같은 것

이런 방식으로 내가 만약 블록체인에서 어떤 데이터를 변경하면, 나의 공개 키를 통해 내가 거기 서명을 한 사람이라고 증명할 수 있다.
-> 하지만 아무도 내 개인 키를 모르기 때문에, 내 트랜잭션을 누구도 위조할 수 없다.

이런 암호학은 복잡하네. 그러니 보안 전문가이고 진짜로 무엇을 하고 있는지 알지 못한다면, 
우리의 앱 프론트엔드에서 사용자들의 개인 키를 관리하려 하는 것은 아마 좋은 생각이 아닐 것이다

이를 대신 처리해주는 서비스가 이미 있다. 이중 가장 유명한 것은 메타마스크(Metamask)

### 메타마스크(Metamask)

사용자들이 이더리움 계정과 개인 키를 안전하게 관리할 수 있게 해주는 크롬과 파이어폭스의 브라우저 확장 프로그램

그리고 해당 계정들을 써서 Web3.js를 사용하는 웹사이트들과 상호작용을 할 수 있도록 해준다 
(만약 자네가 이걸 써본 적이 없다면, 자네는 분명 이걸 설치해보고 싶을 것이다 - 그러면 자네의 브라우저에서 Web3가 작동하고, 
이더리움 블록체인과 통신하는 어떤 웹사이트와도 상호작용을 할 수 있다!).

그리고 개발자로서, 사용자들이 웹 브라우저를 써서 웹사이트를 통해 자네의 DApp과 상호작용을 하길 원한다면, DApp을 메타마스크와 호환할 수 있게 하고 싶을 것이다.

> 참고 : 메타마스크는 내부적으로 Infura의 서버를 Web3 프로바이더로 사용한다.
하지만 사용자들에게 그들만의 Web3 프로바이더를 선택할 수 있는 옵션을 주기도 한다. 
즉 메타마스크의 Web3 프로바이더를 사용하면, 사용자에게 선택권을 주는 것이기도 하면서 앱에서 걱정할 거리를 하나 줄일 수 있다.

### 메타마스크의 Web3 프로바이더 사용하기

메타마스크는 web3라는 전역 자바스크립트 객체를 통해 브라우저에 Web3 프로바이더를 주입한다.

그러니 앱에서는 web3가 존재하는지 확인하고, 만약 존재한다면 web3.currentProvider를 프로바이더로서 사용하면 된다.

메타마스크에서 제공하는 템플릿 코드가 있다. 
사용자가 메타마스크를 설치했는지 확인하고 설치가 안 된 경우 우리 앱을 사용하려면 메타마스크를 설치해야 한다고 알려주는 것

```javascript
window.addEventListener('load', function() {

  // Web3가 브라우저에 주입되었는지 확인(Mist/MetaMask)
  if (typeof web3 !== 'undefined') {
    // Mist/MetaMask의 프로바이더 사용
    web3js = new Web3(web3.currentProvider);
  } else {
    // 사용자가 Metamask를 설치하지 않은 경우에 대해 처리
    // 사용자들에게 Metamask를 설치하라는 등의 메세지를 보여줄 것
  }

  // 이제 자네 앱을 시작하고 web3에 자유롭게 접근할 수 있네:
  startApp()

})
```

> 참고: 메타마스크 말고도 사용자들이 쓸 수 있는 다른 개인 키 관리 프로그램도 있다. 
미스트(Mist) 웹 브라우저 같은 것들이다. 하지만, 그것들도 모두 web3 변수를 주입하는 동일한 형태를 사용한다. 
그러니 사용자들이 다른 프로그램을 쓰더라도 여기서 설명하는 방식으로 사용자의 Web3 프로바이더를 인식할 수 있을 것이다.

### 컨트랙트와 대화하기

Web3.js는 스마트 컨트랙트와 통신을 위해 2가지를 필요로 한다.

address, ABI

#### 컨트랙트 주소

스마트 컨트랙트를 모두 작성한 후, 그걸 컴파일한 후 이더리움에 배포할 것이다.

컨트랙트를 배포한 후, 해당 컨트랙트는 영원히 존재하는, 이더리움 상에서 고정된 주소를 얻을 것이다.

example)
```
0x06012c8cf97BEaD5deAe237070F9587f8E7A266d
```

해당 스마트 컨트랙트와 통신을 하기 위해 배포 후 주소를 복사해야 한다.

#### 컨트랙트 ABI

ABI는 Application Binary Interface의 줄임말로, 

기본적으로 JSON 형태로 컨트랙트의 메소드를 표현하는 것

컨트랙트가 이해할 수 있도록 하려면 Web3.js가 어떤 형태로 함수 호출을 해야 하는지 알려주는 것이다.


이더리움에 배포하기 위해 컨트랙트를 컴파일할 때, 솔리디티 컴파일러가 ABI를 줄 것이다. 그러니 컨트랙트 주소와 함께 이를 복사하여 저장해야 함.

### Web3.js 컨트랙트 인스턴스화하기

컨트랙트의 주소와 ABI를 얻고 나면, 자네는 다음과 같이 Web3에서 인스턴스화할 수 있다

```
// myContract 인스턴스화
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```

## 컨트랙트 함수 호출하기

Web3.js는 컨트랙트의 함수를 호출하기 위해 우리가 사용할 두 개의 메소드를 가지고 있다.

call, send

### Call

call은 view와 pure 함수를 위해 사용함.

local 노드에서만 실행하고, 블록체인에 트랜잭션을 만들지 않는다.

> view와 pure 함수는 읽기 전용이고 블록체인에서 상태를 변경하지 않는다.
가스를 전혀 소모하지 않고, 메타마스크에서 트랜잭션에 서명하라고 사용자에게 창을 띄우지도 않음.

Web3.js를 사용하여, 다음과 같이 123을 매개 변수로 myMethod라는 이름의 함수를 call할 수 있다
```
myContract.methods.myMethod(123).call()
```

### Send
send는 트랜잭션을 만들고 블록체인 상의 데이터를 변경함.

view와 pure가 아닌 모든 함수에 대해 send를 사용해야 하는 것

> 트랜잭션을 send하는 것은 사용자에게 가스를 지불하도록 하고, 메타마스크에서 트랜잭션에 서명하라고 창을 띄울 것이다. 
Web3 프로바이더로 메타마스크를 사용할 때, send()를 호출하면 자동으로 이 모든 것이 이루어지고, 우리의 코드에 어떤 특별한 것도 추가할 필요가 없다

Web3.js를 사용하여, 다음과 같이 123을 매개 변수로 myMethod라는 이름의 함수를 호출하는 트랜잭션을 send할 수 있다
```
myContract.methods.myMethod(123).send()
```

### 데이터 받기

```
Zombie[] public zombies;
```
솔리디티에서, public으로 변수를 선언하면 자동으로 같은 이름의 퍼블릭 "getter" 함수를 만들어 낸다. 
그러니 ID 15인 좀비를 찾길 원한다면, 변수를 함수인 것처럼 호출할 수 있다: zombies(15)

여기에 프론트엔드에서 좀비 ID를 받아 해당 좀비에 대해 컨트랙트에 질의를 보내고, 결과를 반환하는 자바스크립트 함수를 작성하는 방법이 있다

```
function getZombieDetails(id) {
  return cryptoZombies.methods.zombies(id).call()
}

// 함수를 호출하고 결과를 가지고 무언가를 처리:
getZombieDetails(15)
.then(function(result) {
  console.log("Zombie 15: " + JSON.stringify(result));
});
```

cryptoZombies.methods.zombies(id).call()는 Web3 프로바이더와 통신하여 우리 컨트랙트의 Zombie[] public zombies에서 인덱스가 id인 좀비를 반환하도록 할 것이다

이는 외부 서버로 API 호출을 하는 것처럼 비동기적으로 일어난다. 즉 Web3는 여기서 Promise를 반환한다.

## 메타마스크 & 계정

### 메타마스크에서 사용자 계정 가져오기

```
var userAccount = web3.eth.accounts[0]
```

사용자가 언제든지 메타마스크에서 활성화된 계정을 바꿀 수 있기 때문에, 
우리 앱은 이 변수의 값이 바뀌었는지 확인하기 위해 계속 감시를 하고 값이 바뀌면 그에 따라 UI를 업데이트해야 할 것이다.

```
var accountInterval = setInterval(function() {
  // 계정이 바뀌었는지 확인
  if (web3.eth.accounts[0] !== userAccount) {
    userAccount = web3.eth.accounts[0];
    // 새 계정에 대한 UI로 업데이트하기 위한 함수 호출
    updateInterface();
  }
}, 100);
```

여기서는 userAccount가 여전히 web3.eth.accounts[0]과 같은지 확인하기 위해 100밀리초마다 확인하고 있다(즉 사용자가 해당 계정을 활성화해놓았는지 확인하는 것이지). 
그렇지 않다면, userAccount에 현재 활성화된 계정을 다시 할당하고, 화면을 업데이트하기 위한 함수를 호출한다.

### 좀비 스프라이트 표현
```
// 좀비의 머리를 표현하는 1-7의 정수 얻기
var head = parseInt(zombie.dna.substring(0, 2)) % 7 + 1

// 순차적인 파일 이름으로 7개의 머리 이미지를 가지고 있네:
var headSrc = "../assets/zombieparts/head-" + head + ".png"
```

> 참고 https://github.com/loomnetwork/zombie-char-component


## 트랜잭션 보내기
end 함수를 이용해 스마트 컨트랙트의 데이터를 변경하는 방법

1. 트랜잭션을 전송(send)하려면 함수를 호출한 사람의 from 주소가 필요하다(솔리디티 코드에서는 msg.sender).
 이는 우리 DApp의 사용자가 되어야 할 것이니, 메타마스크가 나타나 그들에게 서명을 해야한다.

2. 트랜잭션 전송(send)은 가스를 소모함.

3. 사용자가 트랜잭션 전송을 하고 난 후 실제로 블록체인에 적용될 때까지는 상당한 지연이 발생할 것이다.
    트랜잭션이 블록에 포함될 때까지 기다려야 하는데, 이더리움의 평균 블록 시간이 15초이기 때문이다. 
    만약 이더리움에 보류 중인 거래가 많거나 사용자가 가스 가격을 지나치게 낮게 보낼 경우, 우리 트랜잭션이 블록에 포함되길 기다려야 하고, 이는 몇 분씩 걸릴 수 있다.

### 좀비 만들기

solidity 코드를 다시보면
```
function createRandomZombie(string _name) public {
  require(ownerZombieCount[msg.sender] == 0);
  uint randDna = _generateRandomDna(_name);
  randDna = randDna - randDna % 100;
  _createZombie(_name, randDna);
}
```

다음은 메타마스크를 사용해 web3.js에서 위 함수를 호출하는 방법이다

```javascript
function createRandomZombie(name) {
  // 시간이 꽤 걸릴 수 있으니, 트랜잭션이 보내졌다는 것을
  // 유저가 알 수 있도록 UI를 업데이트해야 함
  $("#txStatus").text("Creating new zombie on the blockchain. This may take a while...");
  // 우리 컨트랙트에 전송하기:
  return CryptoZombies.methods.createRandomZombie(name)
  .send({ from: userAccount })
  .on("receipt", function(receipt) {
    $("#txStatus").text("Successfully created " + name + "!");
    // 블록체인에 트랜잭션이 반영되었으며, UI를 다시 그려야 함
    getZombiesByOwner(userAccount).then(displayZombies);
  })
  .on("error", function(error) {
    // 사용자들에게 트랜잭션이 실패했음을 알려주기 위한 처리
    $("#txStatus").text(error);
  });
}
```

Web3 프로바이더에게 트랜잭션을 전송(send)하고, 몇 가지 이벤트 리스너들을 연결한다.

1. receipt는 트랜잭션이 이더리움의 블록에 포함될 때, 즉 좀비가 생성되고 우리의 컨트랙트에 저장되었을 때 발생하게 된다.

2. error는 트랜잭션이 블럭에 포함되지 못했을 때, 예를 들어 사용자가 충분한 가스를 전송하지 않았을 때 발생하게 된다. 
    우리는 우리의 UI를 통해 사용자에게 트랜잭션이 전송되지 않았음을 알리고, 다시 시도할 수 있도록 할 것이다.

> 참고 : send를 호출할 때 gas와 gasPrice를 선택적으로 지정할 수 있다. 
.send({ from: userAccount, gas: 3000000 }). 만약 지정하지 않는다면, 메타마스크는 사용자가 이 값들을 선택할 수 있도록 할 것이다.


## Payable 함수 호출하기

 Web3.js에서 특별한 처리가 필요한 다른 종류의 함수 -  payable 함수

 ```
 function levelUp(uint _zombieId) external payable {
  require(msg.value == levelUpFee);
  zombies[_zombieId].level++;
}
 ```

### Wei란?

wei는 이더의 가장 작은 하위 단위 - 하나의 이더는 10^18개의 wei

```
// 이렇게 하면 1 ETH를 Wei로 바꿀 것이네
web3js.utils.toWei("1");
```

우리는 levelUpFee = 0.001 ether로 설정했다. 
그러니 levelup 함수를 호출할 때, 아래의 코드를 써서 사용자가 0.001 이더를 보내게 할 수 있다

```
CryptoZombies.methods.levelUp(zombieId)
.send({ from: userAccount, value: web3js.utils.toWei("0.001") })
```

### 이벤트(Event) 구독하기

zombiefactory.sol에 다음과 같은 이벤트가 있었다.
```
event NewZombie(uint zombieId, string name, uint dna);
```

Web3.js에서 이벤트를 구독하여 해당 이벤트가 발생할 때마다 Web3 프로바이더가 자네의 코드 내의 어떠한 로직을 실행시키도록 할 수 있다

```
cryptoZombies.events.NewZombie()
.on("data", function(event) {
  let zombie = event.returnValues;
  // `event.returnValue` 객체에서 이 이벤트의 세 가지 반환 값에 접근할 수 있네:
  console.log("새로운 좀비가 태어났습니다!", zombie.zombieId, zombie.name, zombie.dna);
}).on("error", console.error);
```
\* 이건 DApp에서 어떤 좀비가 생성되든지 항상 알림을 보낸다. (현재 사용자의 좀비만이 아니라는 것)


이벤트를 필터링하고 현재 사용자와 연관된 변경만을 수신하기 위해, 
우리의 ERC721을 구현할 때 Transfer 이벤트에서 했던 것처럼 우리의 솔리디티 컨트랙트에 indexed 키워드를 사용해야 된다.

```
event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
```

이 경우, _from과 _to가 indexed 되어 있기 때문에, 우리 프론트엔드의 이벤트 리스너에서 이들을 필터링할 수 있다.

event와 indexed 영역을 사용하는 것은 컨트랙트에서 변화를 감지하고 프론트엔드에 반영할 수 있게 하는 유용한 방법
```
// `filter`를 사용해 `_to`가 `userAccount`와 같을 때만 코드를 실행
cryptoZombies.events.Transfer({ filter: { _to: userAccount } })
.on("data", function(event) {
  let data = event.returnValues;
  // 현재 사용자가 방금 좀비를 받았네!
  // 해당 좀비를 보여줄 수 있도록 UI를 업데이트할 수 있도록 여기에 추가
}).on("error", console.error);
```

### 지난 이벤트에 대해 질의하기

 getPastEvents를 이용해 지난 이벤트들에 대해 질의를 하고, fromBlock과 toBlock 필터들을 이용해 이벤트 로그에 대한 시간 범위를 솔리디티에 전달할 수 있다.

 ```
 cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
.then(function(events) {
  // `events`는 우리가 위에서 했던 것처럼 반복 접근할 `event` 객체들의 배열이네.
  // 이 코드는 생성된 모든 좀비의 목록을 우리가 받을 수 있게 할 것이네.
});
 ```