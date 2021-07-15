# BlockChain Development

## BlockChain

- block 을 형태를 대략적으로 표현한 sample은 다음과 같다
```
block = {
    'index': 1,
    'timestamp': 1506057125.900785,
    'transactions': [
        {
            'sender': "8527147fe1f5426f9dd545de4b27ee00",
            'recipient': "a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount': 5,
        }
    ],
    'proof': 324984774000,
    'previous_hash': "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}
```
- transactions 과 block 은 이전 블록의 해쉬를 가지고 있다. 
- 

### 직접 블록체인 개발소스 
- [Learn Blockchains by Building One | Hacker Noon](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46)
    - python으로 구현한 소스 #python 
- [frogg/Blockchain-Swift-Playground: An educational implementation of a simple blockchain in Swift Playgrounds. (github.com)](https://github.com/frogg/Blockchain-Swift-Playground)
    - 좀 된 소스이긴 하지만 swift playground book 구현 #swiftPlayground 
 - [danistefanovic/build-your-own-x: 🤓 Build your own (insert technology here) (github.com)](https://github.com/danistefanovic/build-your-own-x#build-your-own-blockchain--cryptocurrency)
	 - 위 프로젝트에서 블록체인 파트 에서 go, javascript, python 등등 으로 구현한것들에 리스트 
	 - 특히 끝에 typescript로 구현 두개의 프로젝트가 자세한 설명이 나와있다. #typescript
		 - [Naivecoin: a tutorial for building a cryptocurrency (lhartikk.github.io)](https://lhartikk.github.io/)
		 - [NaivecoinStake | A tutorial for building a Proof of Stake cryptocurrency (learn.uno)](https://naivecoinstake.learn.uno/)
		 - [A Practical Introduction to Blockchain with Python // Adil Moujahid // Data Analytics and more](http://adilmoujahid.com/posts/2018/03/intro-blockchain-bitcoin-python/)
		 

## Crypto Coin
- [cosme12/SimpleCoin](https://github.com/cosme12/SimpleCoin)
    - python 으로 구현된 말그대로 simple coin 단 3개의 파일로 되어있다. 


## 이더리움
### smart contract

### erc721
- NFT 표준 
    - 

### ipfs
- 탈중앙화된 파일 시스템 
- 파일 업로드시 사용

## Klaytn
- [Welcome - Klaytn Docs KO](https://ko.docs.klaytn.com/)
    - 요기에 일단 자세히 나와있음 
- Token
    - [Klaytn 호환 토큰 - Klaytn Docs KO](https://ko.docs.klaytn.com/smart-contract/token-standard)
    - KIP-17 (ERC-721) 
        - NFT 토큰 
        - KIP-13(interface query standard) 도 같이 구현해야 함
        - wallnet interface 를 구현해 지갑어플리케이션 개발 가능
    - KIP-7(ERC-20)

## Link
- Course 
    - [암호화폐 101 with Nico - YouTube](https://www.youtube.com/playlist?list=PL7jH19IHhOLOJfXeVqjtiawzNQLxOgTdq)
    - [블록체인 개발 강의 - 추천순 블록체인 개발 온라인 강의 | 인프런 (inflearn.com)](https://www.inflearn.com/courses/it-programming/dev-blockchain)
        - [[무료] Klaytn 클레이튼 블록체인 어플리케이션 만들기 - 이론과 실습 - 인프런 | 강의 (inflearn.com)](https://www.inflearn.com/course/%ED%81%B4%EB%A0%88%EC%9D%B4%ED%8A%BC)
    - [Home - EatTheBlocks](https://eattheblocks.com/)
        - 요기가 상당히 퀄리티와 비용이 둘다 높은듯.. 
        - 소스코드는 [jklepatch/eattheblocks: Source code for Eat The Blocks, a screencast for Ethereum Dapp Developers (github.com)](https://github.com/jklepatch/eattheblocks) 요기에 

----
- tags: #blockchain