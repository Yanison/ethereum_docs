<!--
This repository contains specifications for the peer-to-peer networking protocols used by
Ethereum. The issue tracker here is for discussions of protocol changes. It's also OK to
open an issue if you just have a question.
-->
이 리포지토리는 이더리움에서 사용되는 peer-to-peer 네트워킹 프로토콜에 대한 명세에 대한 내용을 담고 있습니다.
여기서 이슈 트래커는 프로토콜 변경에 대한 토론을 위한 것입니다. 질문이 있을 경우 이슈를 열어도 괜찮습니다.

<!--
Protocol level security issues are valuable! Please report serious issues responsibly
through the [Ethereum Foundation Bounty Program].
-->
프로토콜 레벨의 보안 이슈는 매우 중요합니다! 
심각한 문제가 있을 경우 [Ethereum Foundation Bounty Program]을 통해 책임있게 보고해주세요.
<!--
We have several specifications for low-level protocols:

- [Ethereum Node Records]
- [DNS Node Lists]
- [Node Discovery Protocol v4]
- [Node Discovery Protocol v5]
- [RLPx protocol]

The repository also contains specifications of many RLPx-based application-level protocols:

- [Ethereum Wire Protocol] (eth/68)
- [Ethereum Snapshot Protocol] (snap/1)
- [Light Ethereum Subprotocol] (les/4)
- [Parity Light Protocol] (pip/1)
- [Ethereum Witness Protocol] (wit/0)
-->

여기에는 여러가지 저수준 프로토콜에 대한 명세가 있습니다.

- [Ethereum Node Records]
- [DNS Node Lists]
- [Node Discovery Protocol v4]
- [Node Discovery Protocol v5]
- [RLPx protocol]

그리고 RLPx 기반의 여러 응용 프로그램 수준 프로토콜 명세가 있습니다:

- [Ethereum Wire Protocol] (eth/68)
- [Ethereum Snapshot Protocol] (snap/1)
- [Light Ethereum Subprotocol] (les/4)
- [Parity Light Protocol] (pip/1)
- [Ethereum Witness Protocol] (wit/0)

<!--
### The Mission

devp2p is a set of network protocols which form the Ethereum peer-to-peer network.
'Ethereum network' is meant in a broad sense, i.e. devp2p isn't specific to a particular
blockchain, but should serve the needs of any networked application associated with the
Ethereum umbrella.

We aim for an integrated system of orthogonal parts, implemented in multiple programming
environments. The system provides discovery of other participants throughout the Internet
as well as secure communication with those participants.

The network protocols in devp2p should be easy to implement from scratch given only the
specification, and must work within the limits of a consumer-grade Internet connection. We
usually design protocols in a 'specification first' approach, but any specification
proposed must be accompanied by a working prototype or implementable within reasonable
time.
-->

### The Mission

devp2p는 이더리움 P2P 네트워크에서 사용되는 여러 네트워크 프로토콜의 집합입니다.
'이더리움 네트워크'는 넓은 의미로 사용되고 devp2p는 특정 블록체인을 명세하지 않습니다.
그러나 이더리움이라는 프로토콜하에 연관된 모든 네트워크 응용 프로그램의 요구사항을 충족시키기 위해 설계되었습니다.

우리는 다양한 프로그래밍 언어로 구현된 독립적으로 동작하는것들을 통합하는 시스템을 목표로합니다.
이 시스템은 인터넷을 통해 참여자들을 탐색하고 그 참여자들과 안전하게 보안된 통신할 수 있도록 지원합니다.

devp2p의 네트워크 프로토콜은 명세만으로도 구현하기 쉬워야 하며, 소비자 단계의 인터넷에서 작동해야 합니다.
우리는 명세 우선적인 접근으로 이 프로토콜을 전반적으로 설계했지만 어떤 명세가 제안되더라도 합리적인 시간 안에서
동작하거나 구현 가능해야 합니다.

<!--
### Relationship with libp2p

The [libp2p] project was started at about the same time as devp2p and seeks to be a
collection of modules for assembling a peer-to-peer network from modular components.
Questions about the relationship between devp2p and libp2p come up rather often.

It's hard to compare the two projects because they have different scope and are designed
with different goals in mind. devp2p is an integrated system definition that wants to
serve Ethereum's needs well (although it may be a good fit for other applications, too)
while libp2p is a collection of programming library parts serving no single application in
particular.

That said, both projects are very similar in spirit and devp2p is slowly adopting parts of
libp2p as they mature.
-->
[libp2p] 프로젝트는 devp2p와 거의 동시에 시작되었고 모듈화된 P2P 네트워크를 구성하기 위핸
컬렉션 모듈을 목표로 합니다. devp2p와 libp2p 사이의 관계에 대한 질문은 상당히 자주 나옵니다.

두 프로젝트를 비교하는것은 어렵습니다. 왜냐하면 두 프로젝트는 서로 다른 범위를 가지고 있고 서로 다른 목표를 가지고 설계되었습니다.
devp2p는 이더리움의 노드를 잘 지원하기위해(다른 어플리케이션과 잘 호환되기 위함도 있습니다) 설계된 통합 시스템 입니다.
반면에 libp2p는 특정 어플리케이션을 지원하는게 아닌 프로그래밍 라이브러리의 컬렉션입니다.

그렇지만 두 프로젝트의 취지는 비슷하고 devp2p는 libp2p의 일부분을 점차 채택하고 있습니다.

<!--
### Implementations

devp2p is part of most Ethereum clients. Implementations include:

- C#: Nethermind <https://github.com/NethermindEth/nethermind>
- C++: Aleth <https://github.com/ethereum/aleth>
- C: Breadwallet <https://github.com/breadwallet/breadwallet-core>
- Elixir: Exthereum <https://github.com/exthereum/ex_wire>
- Go: go-ethereum/geth <https://github.com/ethereum/go-ethereum>
- Java: Tuweni RLPx library <https://github.com/apache/incubator-tuweni/tree/master/rlpx>
- Java: Besu <https://github.com/hyperledger/besu>
- JavaScript: EthereumJS <https://github.com/ethereumjs/ethereumjs-devp2p>
- Kotlin: Tuweni Discovery library <https://github.com/apache/incubator-tuweni/tree/master/devp2p>
- Nim: Nimbus nim-eth <https://github.com/status-im/nim-eth>
- Python: Trinity <https://github.com/ethereum/trinity>
- Ruby: Ciri <https://github.com/ciri-ethereum/ciri>
- Ruby: ruby-devp2p <https://github.com/cryptape/ruby-devp2p>
- Rust: rust-devp2p <https://github.com/rust-ethereum/devp2p>
- Rust: openethereum <https://github.com/openethereum/openethereum>
- Rust: reth <https://github.com/paradigmxyz/reth>

WireShark dissectors are available here: <https://github.com/ConsenSys/ethereum-dissectors>

[Ethereum Foundation Bounty Program]: https://bounty.ethereum.org
[Ethereum Wire Protocol]: ./caps/eth.md
[Ethereum Snapshot Protocol]: ./caps/snap.md
[Light Ethereum Subprotocol]: ./caps/les.md
[Ethereum Witness Protocol]: ./caps/wit.md
[Ethereum Node Records]: ./enr.md
[DNS Node Lists]: ./dnsdisc.md
[Node Discovery Protocol v4]: ./discv4.md
[Node Discovery Protocol v5]: ./discv5/discv5.md
[Parity Light Protocol]: ./caps/pip.md
[RLPx protocol]: ./rlpx.md
[libp2p]: https://libp2p.io
-->

### 구현체

devp2p는 대부분의 이더리움 클라이언트의 일부입니다. 구현체는 다음과 같습니다:

- C#: Nethermind <https://github.com/NethermindEth/nethermind>
- C++: Aleth <https://github.com/ethereum/aleth>
- C: Breadwallet <https://github.com/breadwallet/breadwallet-core>
- Elixir: Exthereum <https://github.com/exthereum/ex_wire>
- Go: go-ethereum/geth <https://github.com/ethereum/go-ethereum>
- Java: Tuweni RLPx library <https://github.com/apache/incubator-tuweni/tree/master/rlpx>
- Java: Besu <https://github.com/hyperledger/besu>
- JavaScript: EthereumJS <https://github.com/ethereumjs/ethereumjs-devp2p>
- Kotlin: Tuweni Discovery library <https://github.com/apache/incubator-tuweni/tree/master/devp2p>
- Nim: Nimbus nim-eth <https://github.com/status-im/nim-eth>
- Python: Trinity <https://github.com/ethereum/trinity>
- Ruby: Ciri <https://github.com/ciri-ethereum/ciri>
- Ruby: ruby-devp2p <https://github.com/cryptape/ruby-devp2p>
- Rust: rust-devp2p <https://github.com/rust-ethereum/devp2p>
- Rust: openethereum <https://github.com/openethereum/openethereum>
- Rust: reth <https://github.com/paradigmxyz/reth>

WireShark dissectors는 여기에서 사용할 수 있습니다:<https://github.com/ConsenSys/ethereum-dissectors>

[Ethereum Foundation Bounty Program]: https://bounty.ethereum.org
[Ethereum Wire Protocol]: ./caps/eth.md
[Ethereum Snapshot Protocol]: ./caps/snap.md
[Light Ethereum Subprotocol]: ./caps/les.md
[Ethereum Witness Protocol]: ./caps/wit.md
[Ethereum Node Records]: ./enr.md
[DNS Node Lists]: ./dnsdisc.md
[Node Discovery Protocol v4]: ./discv4.md
[Node Discovery Protocol v5]: ./discv5/discv5.md
[Parity Light Protocol]: ./caps/pip.md
[RLPx protocol]: ./rlpx.md
[libp2p]: https://libp2p.io