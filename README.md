# ethereum 황서 번역
본 Repository는 이더리움에서 사용되는 P2P 네트워크 프로토콜에 대한 명세들을 담고 있습니다.
여기서의 이슈 트래커는 프로토콜 변경에 대한 토론을 위한 것이다. 질문이 있을 경우 이슈를 열어주세요.
프로토콜 레벨의 보안 이슈 또한 중요합니다. 심각한 이슈는 이더리움 재단 바운티 프로그램을 통해 책임감 있게 보고해주세요.

우리는 여러가지의 low-level의 프로토콜 명세를 가지고 있습니다.

- [Ethereum Node Records](https://github.com/Yanison/ethereum_docs/blob/master/ethereum/devp2p/ENR.md)
- [DNS Node Lists](https://github.com/Yanison/ethereum_docs/blob/master/ethereum/devp2p/DNS_Node_List.md)
- [Node Discovery Protocol v4](https://github.com/ethereum/devp2p/blob/master/discv4.md)
- [Node Discovery Protocol v5](https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md)
- [RLPx protocol](https://github.com/ethereum/devp2p/blob/master/rlpx.md)

그리고 RLPx 기반의 여러 application-level의 프로토콜들에 대한 명세도 포함하고 있다.

- [Ethereum Wire Protocol (eth/68)](https://github.com/ethereum/devp2p/blob/master/caps/eth.md)
- [Ethereum Snapshot Protocol (snap/1)](https://github.com/ethereum/devp2p/blob/master/caps/snap.md)
- [Light Ethereum Subprotocol (les/4)](https://github.com/ethereum/devp2p/blob/master/caps/les.md)
- [Parity Light Protocol (pip/1)](https://github.com/ethereum/devp2p/blob/master/caps/pip.md)
- [Ethereum Witness Protocol (wit/0)](https://github.com/ethereum/devp2p/blob/master/caps/wit.md)

## The Mission : what is devp2p?

devp2p는 이더리움의 P2P네트워크를 형성하는 네트워크 프로토콜 집합입니다.
'이더리움 네트워크'는 넓은 의미로, devp2p는 특정 블록체인을 지칭하는것이 아니고, 
반드시 이더리움안에서 형성되고 네트워킹하는 어플리케이션들의 needs를 제공해야 합니다.

우리는 다양한 프로그래밍 환경으로 구현하여 독립적이지만 통합된 시스템을 목표로 합니다.
이 시스템은 인터넷 전반적으로 다른 참여자들을 발굴하고, 그 참여자들과 안전한 통신을 제공합니다.

dev2p 의 이 네트워크 프로토콜은 명세로부터 제공된 기초 정보들로 쉽게 구현할 수 있어야 합니다.
그리고 소비계층의 인터넷 통신 제한 안에서 작동해야 합니다.
우리는 보편적으로 '명세 우선' 접근방식으로 프로토콜을 설계합니다.
그러나 제안된 어떠한 명세들은 동작 가능한 프로토 타입이나 적절한 노력과 시간으로 구현 가능해야 합니다.

# Relationship with libp2p
libp2p 프로젝트는 devp2p와 거의 동시에 시작되었고 P2P 모듈 네트워크들을 하나로 구성하는 P2P네트워크 모듈러 집합이 되는것을 추구합니다.
devp2와 libp2p의 관계에 대한 질문은 상당히 자주 나옵니다.

두개의 프로젝트를 비효하는건 어렵습니다 왜냐하면 그들은 다른 관점과 다른 목표로 설계되었기 때문입니다.
devp2p는 이더리움의 니즈를 제공하기 위한 시스템적인 정의들을 통합한것입니다.(물론 다른 어플리케이션에도 잘 맞을 수 있습니다.)
반면 libp2p는 프로그래밍 라이브러리 요소들의 집합체입니다. 특정한 어플리케이션을 서비스하기 위함이 아니란 뜻 입니다.

That said, both projects are very similar in spirit and devp2p is slowly adopting parts of libp2p as they mature.
즉, 두 프로젝트는 서로 비슷한 정신을 가지고 있으며 그들이 성숙해져감에 따라 devp2p는 서서히 libp2p의 일부를 채택하고 있습니다.

# Implementations
devp2p is part of most Ethereum clients. Implementations include:
devp2p는 이더리움 클라이언트들의 일부분입니다. 구현체들은 다음을 포함합니다.

- C#: Nethermind https://github.com/NethermindEth/nethermind
- C++: Aleth https://github.com/ethereum/aleth
- C: Breadwallet https://github.com/breadwallet/breadwallet-core
- Elixir: Exthereum https://github.com/exthereum/ex_wire
- Go: go-ethereum/geth https://github.com/ethereum/go-ethereum
- Java: Tuweni RLPx library https://github.com/apache/incubator-tuweni/tree/master/rlpx
- Java: Besu https://github.com/hyperledger/besu
- JavaScript: EthereumJS https://github.com/ethereumjs/ethereumjs-devp2p
- Kotlin: Tuweni Discovery library https://github.com/apache/incubator-tuweni/tree/master/devp2p
- Nim: Nimbus nim-eth https://github.com/status-im/nim-eth
- Python: Trinity https://github.com/ethereum/trinity
- Ruby: Ciri https://github.com/ciri-ethereum/ciri
- Ruby: ruby-devp2p https://github.com/cryptape/ruby-devp2p
- Rust: rust-devp2p https://github.com/rust-ethereum/devp2p
- Rust: openethereum https://github.com/openethereum/openethereum
- Rust: reth https://github.com/paradigmxyz/reth
- WireShark dissectors are available here: https://github.com/ConsenSys/ethereum-dissectors


# 요약
