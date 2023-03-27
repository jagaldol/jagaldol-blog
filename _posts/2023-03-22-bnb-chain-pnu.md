---
layout: single
title: "2023 BNB CHAIN Innovation University Tour Korea in 부산대 후기"
date: 2023-03-22 18:00:00 +0900
last_modified_at: 2023-03-23 00:32:00 +0900
categories: experience
---

<img src="/assets/images/2023-03-22/semina-view.jpg" width="500px"/>

안녕하세요! 이번에 저희 학교에서 바이낸스의 BNB Chain에서 블록체인 세미나를 진행한다더라고요.

<img src="/assets/images/2023-03-22/information.png" width="500px" />

단순히 강연 뿐만 아니라 기술 세션으로 직접 어플리케이션을 만들어 본다니 참을 수 없어서 바로 다녀왔어요.😎

세미나 내용들 정리한거 올려볼게요!

## ⛓️BNB Chain Tech Workshop
### 🦊MetaMask 지갑 생성
<img src="/assets/images/2023-03-22/meta-mask.png" width="500px">

크롬에 [MetaMask](https://metamask.io/download/)를 추가하여 지갑을 생성합니다!

백업 암호 구문(12개 단어)까지 전부 만들어 주고 잘 기록해 둡니다.

메타마스크의 설정 -> 네트워크 -> 네트워크 추가 -> 네트워크 수동 추가로 테스트 네트워크를 만듭니다.
```
Network Name
Smart Chain - Testnet
New RPC URL
https://data-seed-prebsc-1-s1.binance.org:8545/
ChainID
97
Symbol
tBNB
Block Explorer URL
https://testnet.bscscan.com
```
입력하시면 메인 화면의 코인이 ETH(이더리움)에서 새로만든 TBNB로 뜰거에요.

이제 https://testnet.bnbchain.org/faucet-smart 여기로 이동해서 자신의 지갑 account를 넣고 Give me BNB를 누릅니다!

<img src="/assets/images/2023-03-22/get-tbnb.png" width="500px">

돈이 들어와서 0TBNB에서 0.1TBNB가 될거에요!(이후 과정에서 조금 써서 저는 0.1이 아닙니다.)

### 🛠️dApp 만들기
`node v16.13.0`이 필요합니다. [설치 링크](https://nodejs.org/download/release/v16.13.0/)에서 설치하시거나 `nvm`으로 `16.13.0`을 설치해주세요!

nvm의 설치의 경우 윈도우는 [여기](https://github.com/coreybutler/nvm-windows)에서 Download now! 클릭하여 설치하시면 됩니다.

* `nvm`으로 `node 16.13.0` 변경
```terminal
nvm install 16.13.0
nvm use 16.13.0
```

이제 필요 패키지를 설치하고 bnb-chain의 튜토리얼을 clone합니다.

```terminal
npm install -g truffle
npm install -g ganache-cli
git clone https://github.com/bnb-chain/bnb-chain-tutorial.git
cd bnb-chain-tutorial
cd 01- Hello World Full Stack dApp on BSC
npm install
```

`01- Hello World Full Stack dApp on BSC` 안에 `.secret 파일`을 만들어줘야합니다. `.secret`를 메타 마스크의 백업 암호 구문 12개 단어로 작성해 주세요.

```terminal
echo word1 word2 word3 word4 word5 word6 word7 word8 word9 word10 word11 word12 > .secret
```

이제 컴파일을 하고 smart contract를 migrate할게요.
```terminal
truffle compile

truffle migrate --reset --network bscTestnet
```

새로운 터미널을 만들어서 ganache로 실행을 시킬게요.
```terminal
cd bnb-chain-tutorial
cd 01- Hello World Full Stack dApp on BSC
ganachi-cli
```

다시 돌아와 테스트 해봅니다.
```terminal
truffle test
```

뭔가 잘 되는것 같나요? 이제 빌드하고 실행해줍니다!
```terminal
npm run build
npm run dev
```
[localhost:3000](http://localhost:3000/)에 접속해 보면\...!!

<img src="/assets/images/2023-03-22/smart-contract-test.png" width="500px">

페이지가 하나 뜹니다!

확장 프로그램의 MetaMask에서 연결된 사이트를 누르고 현재 페이지 추가(localhost:3000 추가)를 합니다!

<img src="/assets/images/2023-03-22/meta-mask-setting.png" width="500px">

그러고 사이트에서 이름을 Save Name하면!

<img src="/assets/images/2023-03-22/smart-contract.png" width="500px">

Meta-mask에서 자동으로 contract가 열리고 확인을 누르면 거래가 됩니다!

<img src="/assets/images/2023-03-22/hello-success.png" width="500px">

Greet을 눌러보면 페이지도 Hello, hyejun으로 잘 바뀌었네요~

<img src="/assets/images/2023-03-22/contract-view.png" width="500px">

거래 내역을 보시면 From, To가 다 기록이 되며 잘 거래가 된것을 볼 수 있습니다.

이상으로 dApp으로 smart contract 실습입니다! 세션에서 시키는 대로만 했는데 뭔가 뚝딱 만들어졌네요😅 그래도 나중에 블록체인 활용할때 이번 경험을 참고하면 좋을 것 같네요. 재밌었어요!

<br/>

<br/>

아래는 강연이라 내용들만 간략히 적었어요!

---

## 📈Binance Academy 발표 세션
### Portfolio
자산의 총 집합(현금, 주식, 코인, 부동산 등)

### 투자 위험성의 종류
* Market Risk 자산 하락 위험
* Liquidity Risk 원할 때 못 사거나 못 팔 수 있음
* Operational Risk 사람이나 하드웨어적으로 발생하는 위험
* Systemic Risk 산업 자체의 붕괴(ex 코인이 해킹되었다)

### Trading 기법
<table>
    <tr>
        <th>기법</th>
        <th>설명</th>
    </tr>
    <tr>
        <td>Day</td>
        <td>일일 매매</td>
    </tr>
    <tr>
        <td>Swing</td>
        <td>일정 기간(2주 ~ 한달)을 잡고 목표가 도달 시 매매</td>
    </tr>
    <tr>
        <td>Scalping</td>
        <td>짧게 들어갔다가 바로 빠지는것(몇 분, 몇 초 사이의 매매)</td>
    </tr>
    <tr>
        <td>Buy & Hold</td>
        <td>그저 '존버'</td>
    </tr>
</table>

### 보조 지표
#### RSI(Relative Strength Index)
* 30 밑으로 내려가면 매수 타이밍
* 70 위로 올라가면 매도 타이밍

#### Moving Average 이평선(이동평균선)

* 50일선과 200일선이 있음
* 200일선 위로 50일선이 가면 golden cross(영차영차)
* 반대는 dead cross(돔황챠!)

#### Bollinger Bands
* 위를 터치하면 하락 신호
* 아래를 터치하면 상승 신호

#### 피보나치 되돌림
* 0.618에서 0.382를 뚫으면 반등 신호
* 0382에서 0.618을 뚫으면 하락 신호

### 분산 투자를 하자!
은행조차 bankrun을 하게 됨. 위험하다! 현명한 투자자는 여러군데에 분산해야한다.

---

## 🕹️pomerium web3 game studio

* 게임파이(gamefi : game + defi + NFT)와 P2E(Play to Earn)
    * 게임 아이템을 NFT로 교환할 수 있어 실제 자산이 될 수 있다.
* Social 계정으로 지갑을 쉽게 생성할 수 있게 함
* on-chain이 아닌 off-chain 데이터는 검증 프로토콜을 적용하여 신뢰성 확보
* 기존 게이머(WEB2 유저)를 WEB3로 온보딩하는게 목표
* 한국은 규제때문에 해외 서비스가 더 활발
* 주로 모바일 게임 타깃으로 캐주얼 게임 위주

---

## 후기

2시간동안 진행이 빠르게 되서 받아적느라 힘들었네요...

WEB3와 블록체인 기술이 뜨고 있는데 제가 관련 기술을 잘 알지 못해 아쉬웠단 말이죠? 이번 세미나를 참가하고 WEB3와 관련된 기술에 좀 친숙해진 느낌이라 뿌듯하네요.

<img src="/assets/images/2023-03-22/binance-hoodie.jpg" width="500px">

기념품으로 후드도 하나 받았어요!! 쵝오!!👍