# [Node Discovery Protocol v5]

**Protocol version v5.1**

Node Discovery Protocol v5 명세에 오신걸 환영합니다.
본 명세는 작업중이며 사전 공지 없이 변경될 수 있습니다. 

Node Discovery는 peer-to-peer 네트워크에서 다른 참가자들을 찾는 시스템입니다. 
이 시스템은 프로토콜을 실행하고 제한된 수의 다른 노드들을 저장하는것 외에는 어떠한 목적으로 사용하든 어떠한 비용도 들지 않습니다.
모든 노드가 네트워크에서 entry point로 사용될 수 있습니다.

이 시스템의 디자인은 Kademlia DHT에서 대략적인 영감을 받았지만 대부분의 DHT 처럼 임의의 키와 값을 저장하지 않습니다.
대신 Kademlia DHT는 네트워크에서 노드에 대한 정보를 제공하여 서명받은 노드레코드들을 저장하고 릴레이합니다.
Node Discovery는 네트워크에서 활성화된 모든 노드의 데이터베이스 역할을 합니다, 그리고 다음과 같이 세가지 기본적인 기능을 수행합니다.

- 활성화된 모든 참여자들의 집합을 샘플링합니다 : DHT를 통해 네트워크를 열거할 수 있습니다.
- 특정 서비스를 제공하는 참여자들을 탐색합니다 : Node Discovery v5는 'topic advertisements'를 등록할 수 있는 확장성을 가지고 있습니다.
  이런 advertisements는 쿼리될 수 있고 topic을 advertising하는 노드들을 찾을 수 있습니다.
- 신뢰있는 노드레코드 검증 : 만약 노드ID가 알려져있다면, 해당 노드ID의 노드레코드의 가장 최신버전을 다시 불러올 수 있습니다.

## Specification Overview

이 명세는 세가지 부분으로 나뉩니다.

- [discv5-wire.md] 와이어 프로토콜을 정의합니다.
- [discv5-theory.md] 알고리즘과 데이터구조를 설명합니다.
- [discv5-rationale.md] 디자인 이유를 포함합니다.

## Comparison With Other Discovery Mechanisms

MDNS/Bonjour와 같은 시스템은 로컬 네트워크에서 호스트를 찾을수 있게 해줍니다.
Node Discovery Protocol는 인터넷에서 작동될 수 있도록 설계되었으며,
인터넷상에서 분산된(spread across the Internet) 많은 참가자들이 사용하는 어플리케이션에 가장 유용합니다.

rendezvous 서버를 사용하는 시스템 : 이 시스템은 데스크탑 어플리케이션이나 참여자들이 서로 통신할 수 있는 클라우드 서비스등에 주로 사용됩니다.
의심의 여지없이 효율적이지만, 해당 시스템은 rendezvous 서버의 운영자에 대한 신뢰가 요구되고 이런 시스템들은 의도적인 검열에 취약합니다.
rendezvous 서버와 비교해서 Node Discovery Protocol은 단일 운영자에 의존하지 않고 모든 참가자에게 약간의 신뢰성을 부여합니다.
네트워크의 규모가 증가하고 서로다른 여러개의 peer to peer 네트워크 참여자가 발견한 네트워크를 서로 공유함으로서 안정성을 더욱 증가시키디 때문에
검열에 대한 내성이 생깁니다.

*검열(censorship)이란 의미는 단일 operator(운영자)에 의한 의도적인 통제와 같은 검열을 의미합니다.

Node Discovery Protocol의 취약점은 네트워크에 참여하는 과정에 있습니다 : 
다른 노드가 entry point로서 사용될수도 있지만, 해당 노드는 먼저 다른 메커니즘을 통해 찾아져야 합니다.
DNS에 초기 entry points를 확장 가능하게 나열하거나 
로컬 네트워크게 있는 참여자들이 안전하게 네트워크에 진입하는데 사용할 수 있습니다.

## Comparison With Node Discovery v4

- advertisement topic이 추가되었습니다.
- 임의의 노드 메타데이터가 저장/릴레이될 수 있습니다.
- 암호화된 노드의 identity는 확장 가능하며, secp256k1 키의 사용은 엄격하게 필요하지 않습니다.
- 프로토콜은 더이상 시스템 클럭에 의존하지 않습니다.
- 통신은 암호화되어있고 passive observers로부터 의도적인 topic 탐색과 레코드 lookups은 보호됩니다.

[discv5-wire.md]: ./discv5-wire.md
[discv5-theory.md]: ./discv5-theory.md
[discv5-rationale.md]: ./discv5-rationale.md
[Node Discovery Protocol v5]:https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md