# fabric-invoice-demo

■ 참조 사이트 : https://medium.com/@kctheservant/a-hyperledger-fabric-use-case-invoice-financing-32142f195aea

< A Hyperledger Fabric : 송장 대출 Simple 데모 >

1. 서비스 시나리오
Alice Software House : 서비스 제공 및 invoice 발행
Bob Shop : 서비스 사용 주체 및 지불 당사자

송장 : Bob은 Alice가 제공 한 서비스를받은 후 Alice에게 지불합니다.
Alice가 Bob이 지불하기 전에 어떤 이유로 든 현금이 필요한 경우 Alice는 해당 송장을 사용하여 금융 기관 (예 : 은행)으로부터 자금을 조달 할 수 있습니다. 
이를 인보이스 파이낸싱이라 합니다.

Risk Point : 이중 자금 조달 및 차용과 같은 잠재적 인 위험이 즉시 나타날 수 있음
Risk Point Case : 예를 들어 Alice는 동일한 송장을 사용하여 두 은행에서 대출합니다. 금액이 $ 10,000 인 송장이 두 은행에서 $ 7,000를받을 수 있다고 가정하면 총 대출 금액 $ 14,000은 원래 송장 금액을 초과합니다. 
이것은 은행의 위험을 증가시킵니다.

2. 전반적인 디자인

2.1 Bank Network
- 알파 은행 : 1개의 CA, 2개의 peer
- 베타 은행 : 1개의 CA, 2개의 peer
- invoicechannel 채널 생성
- 총4개 노드가 참여
- Bank Network는 First Network of Fabric Samples에서 채택

2.2 Chaincode
- ledger data structure : InvoicedAmount and BorrowedAmount.
- Key : company name and invoice number
- Value : amount 10000 and 7000
예) Alice-invbob001 => (인보이스 금액 : 10000, 빌린 금액 : 7000)

2.2.1 세 가지 인터페이스 기능 정의
- initInv() : 송장 레코드를 초기화 (회사 명, 송장 번호 및 송장 금액이 필요)
- queryInv() : 기존 송장 레코드를 쿼리 (회사 명과 송장 번호가 필요)
- requestLoan() : 기존 송장에 대한 대출을 요청 (회사 명, 송장 번호 및 차용 금액이 필요, 빌린 금액이 인보이스 금액을 초과하는지 확인합니다.)

2.3 Client Applications
각 은행의 승인 된 직원은 클라이언트 응용 프로그램을 통해 은행 네트워크와 상호 작용하는 3개의 프로그램.
- initInv-bank.js
- queryInv-bank.js
- requestLoan-bank.js

두 은행의 공인 된 직원의 신원확인 프로그램. (wallet directory에 user-alpha 및 user-beta생성)
- enrollAdmin-bank.js
- registerUser-bank.js

3. Code
사이트정보 : https://github.com/kctam/invfinancing-banknetwork

- bank-network directory : bank network 정의 =>  crypto material (certificates and signing key for everything) 및 channel artifacts 등 자료
- chaincode directory : invfinancing.go
- starteverything.sh : 컨테이너, 채널 생성 및 모든 피어 노드에 의한 조인, 체인 코드 설치 및 인스턴스화 구동 스크립트.
- teardowneverything.sh : 컨테이너 tears down, 컨테이너 removes, images 존재여부 등

4. Demonstration of Invoice Financing on a Bank Network
The Bank Network is composed of two banks (organizations), Alpha and Beta. Each bank has two peer nodes.

4.1 prepare a Fabric Node
$ cd fabric-samples
$ git clone https://github.com/kctam/invfinancing-banknetwork.git

4.2 bring up everything for demonstration
$ cd invfinancing-banknetwork
$ ./starteverything.sh
$ docker ps -a

4.3 install the required SDK
$ npm install

4.4 Enrol user-alpha and user-beta in wallet
$ node enrollAdmin-alpha.js
$ node registerUser-alpha.js

$ node enrollAdmin-beta.js
$ node registerUser-beta.js

$ ls wallet

4.5 Perform client applications.(Let’s simulate a scenario.)

4.5.1 Alice 회사는 Bank Alpha에 송장 invbob001의 금액 10,000 소유권 기록, 송장 금융 신청 확인, 빌린 금액확인.
$ node initInv-alpha.js Alice invbob001 10000
$ node queryInv-alpha.js Alice invbob001

4.5.2 Alice는 Bank Alpha에 송장에 대해 7,000 대출 및 확인
$ node requestLoan-alpha.js Alice invbob001 7000
$ node queryInv-alpha.js Alice invbob001

4.5.3 Alice 회사는 Bank Beta에 송장을 기록하려고하지만 송장이 이미 기록되어 있습니다.
$ node initInv-beta.js Alice invbob001 10000

4.5.4 Alice 는이 인보이스에 대해 Bank Beta 에 5,000 을 대출 하도록 요청하지만, 은행 네트워크는 즉시 총 빌린 금액이 송장 금액 ( 12,000 > 10,000 )을 초과하여 허용되지 않음을 알게됩니다.
$ node requestLoan-beta.js Alice invbob001 5000

4.5.5 Bank Beta 가이 송장에 따라 Alice 에게 2,000을 대출하기로 결정 했다고 가정 합니다. 대출 요청이 완료되었으며 원장이 갱신되었습니다 (이제 대출 금액은 9,000 )
$ node requestLoan-beta.js Alice invbob001 2000
$ node queryInv-beta.js Alice invbob001

4.6 Clean up
$ ./teardowneverything.sh

5. 결론 ( 원장 / 블록 체인에 보관해야 할 것은 무엇일까요? )
- 필요한 정보 만 원장에 저장해야합니다. 블록 체인 플랫폼은 데이터베이스 또는 모든 저장소를 대체하도록 설계되지 않았습니다.
- 모든 당사자에게 공개되는 정보 만 원장에 저장됩니다. 상호 기밀보호가 요청되는 정보는 배제함.
- 공유가능한 정보기반의 비즈니스 아이디어가 서비스로 진화 가능하리라 생각합니다.

< 참고 명령어 : Clean Up >
cd /fabric-samples/first-network
./byfn.sh down
docker rm $(docker ps -aq)
docker rmi $(docker images dev-* -q)
docker network prune

< 강제로 삭제시 -f >
docker rm -f ID
