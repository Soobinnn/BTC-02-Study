### 블록이 생성되는 과정

1. 지갑을 통해 트랜잭션 거래 생성
2. 생성된 트랜잭션은 네트워크를 통해 모든 노드에 전파
3. 각 노드는 수신된 트랜잭션을 검증
4. 검증 완료된 트랜잭션을 각 노드마다 멤풀에 저장 
\* 멤풀 - 메모리 저장소 (임시 저장소)

5. 각 노드는 일정시간 쌓인 트랜잭션을 모아서 노드별 후보블록을 생성 -> 후보 블록 생성 과정
6. 후보블록 중 대표블록 생성 (채굴) -> 대표 블록 생성 과정
7. 선정된 대표 블록은 다시 네트워크를 통해 모든 노드에 전파
8. 각 노드는 수신된 대표 블록을 검증
9. 검증된 대표블록은 각 노드에 저장된 블록체인에 연결

### 후보 블록 생성 과정

1. 생성된 트랜잭션은 전파와 검증을 거쳐 각 노드의 멤풀에 저장됨.

2. 각 노드들은 멤풀에 저장된 트랜잭션을 선별하여 후보 블록 바디에 포함한다.

3. 각 트랜잭션의 해시값들을 하나의 해시값으로 헤더의 머클 루트에 포함

4. 이전 블록 해시값이 후보블록 헤더의 이전블록 해시에 포함 된다.

### 대표 블록 생성 과정 (채굴) 
1. 논스값을 찾기. 논스값을 찾으면 블록해시 값도 결정되어서 블록이 완성.