# Node Discovery Protocol

본 명세는 Node Discovery protocol version 4에 대해 정의합니다.
Node Discovery Protocol은 Kademlia-like 분산 해시태이블이며(DHT) 이더리움 노드 정보를 저장합니다.
Kademlia 구조를 사용하는 이유는 분산인덱스을 효율적으로 구성하고 짧은 경로를 가지는 네트워크 경로를 형성하기 때문입니다.
최근 프로토콜 버전은 **4** 입니다. 이전 프로토콜 버전의 변경사항은 이 문서의 마지막에 있습니다.

## Node Identities

각 노드는 secp256k1 타원곡선 암호학 알고리즘을 사용한 키값으로 구성된 identity를 가지고 있습니다.
노드의 공개 키는 `node ID` 또는 식별자로서 사용됩니다.

두 노드키 사이의 '거리'는 공개키의 해시값에 대한 'bitwise exclusive or' 연산결과를 숫자로 취한 결과값입니다.

    distance(n₁, n₂) = keccak256(n₁) XOR keccak256(n₂)

###### bitwise exclusive or : 서로 비교되는 두 비트가 서로 다르면 1을 반환, 같으면 0을 반환하는 논리연산

## Node Records

Discovery Protocol의 참여자들은 최신정보를 포함하는 ENR을 유지해야 합니다.
모든 레코드는 반드시 v4 identity 방식을 사용해야 합니다
다른 노드들은 언제든 [ENRRequest] 패킷을 보냄으로써 언제든 로컬 레코드를 요청할 수 있습니다.

최근 레코드의 노드 공개키를 해석하기위해 [FindNode] 패킷을 사용하여 Kademlia lookup을 수행해야 합니다.
노드를 찾으면 ENRRequest을 전송하고 응답에서 레코드를 반환합니다.

## Kademlia Table

Nodes in the Discovery Protocol keep information about other nodes in their neighborhood.
Neighbor nodes are stored in a routing table consisting of 'k-buckets'. For each `i` in
`0 ≤ i < 256`, every node keeps a k-bucket of neighbors with distance
`2^i ≤ distance < 2^(i+1)` from itself.

Discovery Protocol의 노드는 이웃에 있는 다른 노드들의 정보를 유지합니다.
이웃 노드는 'k-buckets' 으로 구성된 라우팅 테이블에 저장됩니다.
k-buckets은 Kademlia 네트워크에서 사용하는 데이터 구조입니다.
각 노드가 유지하는 리스트로 각 리스트는 일정한 범위의 거리를 가지는 이웃 노드들을 포함합니다.
네트워크에서 이웃 노드들을 효율적으로 관리하고 검색하기 위해 사용됩니다.
`0 ≤ i < 256` 사이에 있는 각각의 i에 대해, 모든 노드는 자신으로부터 `2^i ≤ distance < 2^(i+1)`
만큼 거리를 유지하며 이웃의 k-bucket을 유지합니다.

The Node Discovery Protocol uses `k = 16`, i.e. every k-bucket contains up to 16 node
entries. The entries are sorted by time last seen — least-recently seen node at the head,
most-recently seen at the tail.

Node Discovery Protocol은 `k = 16`을 사용합니다. k는 k-bucket에 최대로 저장되는 노드의 수를 의미합니다.
즉, `k = 16`은 각 k-bucket은 최대 16개의 노드 엔트리를 포함합니다.
k-bucket에 있는 노드 엔트리는 마지막으로 조회한 시간 기준으로 가장 오래전에 조회한 노드를 맨 앞에 정렬 시킵니다.
가장 최근에 조회된 노드는 맨 뒤에 정렬됩니다.

새로운 노드 N₁이 발견되면 해당 버킷에 삽입될 수 있습니다.
만약 bucket에 담겨진 엔트리가 `k`보다 적으면 N₁은 단순히 마지막 엔트리로 추가될 수 있습니다.
이미 bucket이 `k`만큼의 엔트리를 가지고 있으면 버킷에서 가장 마지막으로 조회된 노드인 N₂는 [Ping] 패킷을 보냄으로써 재검증되어야 합니다.
가장 마지막으로 조회된 노드인 N₂로부터 응답이 없다면 해당 노드는 죽은것으로 간주되고 지워집니다, 그리고 새로운 노드 N₁는 bucket의 맨 뒤에 추가됩니다.

## Endpoint Proof

To prevent traffic amplification attacks, implementations must verify that the sender of a
query participates in the discovery protocol. The sender of a packet is considered
verified if it has sent a valid [Pong] response with matching ping hash within the last 12
hours.

트래픽 증폭 공격을 방지하기 위해 구현체는 반드시 발신자의 쿼리가 discovery protocol에 참여하고 있는지 검증해야 합니다.
패킷 발신자는 12시간 이내로 유효한 [Pong] 응답을 보냈는지, 그리고 해당 응답값이 ping 해시값과 매칭하여 검증되어졌는디 확인되어야 합니다.

## Recursive Lookup
'lookup'은 노드 ID와 가장 가까운 `k`개의 노드를 찾는것입니다.
'lookup' initiator가 자신이 알고 있는 가장 가까운`a`개의 노드들을 선별하면서 시작됩니다.
별한 노드들에게 동시성 [FindNode] 패킷을 보내는데,
`a`는 전역변수 파라미터이며 한 번에 몇 개의 노드와 동시다발적으로 통신할지를 결정합니다.
initiator는 이전 쿼리를 학습한 노드들에게  [FindNode] 패킷을 재귀적으로 재전송합니다.
initiator가 타겟으로 삼은 가장 가까운 `k`개의 노드중 아직 쿼리되지 않은 `a`개의 노드를 선별하고
`a`에게 [FindNode] 패킷을 재전송합니다. 재빨리 응답하지 못한 노드들은 그들이 응답하거나 응답하지 않으면
`k` 고려대상에서 지워집니다.

If a round of FindNode queries fails to return a node any closer than the closest already
seen, the initiator resends the find node to all of the `k` closest nodes it has not
already queried. The lookup terminates when the initiator has queried and gotten responses
from the `k` closest nodes it has seen.

FindNode 쿼리 단계에서 먼저 찾았던 가까운 노드보다 더 가까운 노드를 찾지 못하면,
lookup initiator는 아직 쿼리되지 않았으며 가장 가까운 `k` 모두에게 FindNode를 재전송합니다.
lookup은 initiator가 찾은 가장 가까운 `k`개의 노드들로부터 응답을 받거나 그들을 쿼리했을때 종료됩니다.

## Wire Protocol
Node discovery 메시지는 UDP 데이터그램으로서 보내집니다. 모든 패킷의 최대 크기는 1280바이트입니다.

    packet = packet-header || packet-data

모든 패킷은 다음과 같은 헤더로 시작합니다.

    packet-header = hash || signature || packet-type
    hash = keccak256(signature || packet-type || packet-data)
    signature = sign(packet-type || packet-data)

The `hash` exists to make the packet format recognizable when running multiple protocols
on the same UDP port. It serves no other purpose.
`hash`는 동일한 UDP 포트에서 다양한 프로토콜이 실핼될때 패킷 형식을 구분하기 위해 존재합니다.
그것 외엔 다른 목적은 없습니다.

Every packet is signed by the node's identity key. The `signature` is encoded as a byte
array of length 65 as the concatenation of the signature values `r`, `s` and the 'recovery
id' `v`.

모든 패킷은 노드의 고유키로 서명됩니다. `signature`은 서명값에서 `r`, `s` 와 복구 ID인 `v`을 연결하여 65길이의 바이트 배열로 인코딩 됩니다.

The `packet-type` is a single byte defining the type of message. Valid packet types are
listed below. Data after the header is specific to the packet type and is encoded as an
RLP list. Implementations should ignore any additional elements in the `packet-data` list
as well as any extra data after the list.

`packet-type`은 메시지의 타입을 결정하는 단일 byte 입니다. 유효한 패킷 타입은 아래에 정리되어있습니다.
헤더 다음의 데이터는 패킷 타입에 따라 다르며 RLP 리스트로 인코딩됩니다.
구현은 RLP 리스트 다음의 추가적인 데이터뿐만 아니라 `packet-data` 리스트에서 추가적인 모든 요소들을 무시해야합니다.

### Ping Packet (0x01)

    packet-data = [version, from, to, expiration, enr-seq ...]
    version = 4
    from = [sender-ip, sender-udp-port, sender-tcp-port]
    to = [recipient-ip, recipient-udp-port, 0]

The `expiration` field is an absolute UNIX time stamp. Packets containing a time stamp
that lies in the past are expired may not be processed.

`expiration` 필드는 유닉스 time stamp 입니다. 패킷은 과거에 파기되어 처리되지 않은 time stamp을 포함하고 있습니다.

The `enr-seq` field is the current ENR sequence number of the sender. This field is
optional.

`enr-seq` 필드는 최근 발신자의 ENR 시퀀스 숫자입니다. 해당 필드는 선택사항입니다.

When a ping packet is received, the recipient should reply with a [Pong] packet. It may
also consider the sender for addition into the local table. Implementations should ignore
any mismatches in version.

ping 패킷을 받을때, 수신자는 반드시 [Pong] 패킷으로 응답해야 합니다.
로컬 테이블에 발신자를 추가하는 것 또한 고려될 수 있습니다.
구현은 버전이 불일치하면 무시해야합니다.

If no communication with the sender has occurred within the last 12h, a ping should be
sent in addition to pong in order to receive an endpoint proof.

만약 발신자와 소통이 마지막 12시간 이내로 발생되지 않으면, 
ping은 endpoint 증명을 받기 위해 pong을 더하여 보내져야 합니다. 

### Pong Packet (0x02)

    packet-data = [to, ping-hash, expiration, enr-seq, ...]

Pong is the reply to ping.
Pong은 ping에 대한 응답입니다.

`ping-hash` should be equal to `hash` of the corresponding ping packet. Implementations
should ignore unsolicited pong packets that do not contain the hash of the most recent
ping packet.

`ping-hash`는 본인과 대응되는 ping 패킷의 `hash`값과 같아야 합니다.
구현은 가장 최근의 ping 패킷의 해시값을 포함하고 있지 않고 요청되지 않은 pong 패킷을 무시해야 합니다.

The `enr-seq` field is the current ENR sequence number of the sender. This field is
optional.

`enr-seq` 필드는 최근 발신자의 ENR 시퀀스 번호입니다. 해당 필드는 선택사항입니다.

### FindNode Packet (0x03)

    packet-data = [target, expiration, ...]

A FindNode packet requests information about nodes close to `target`. The `target` is a
64-byte secp256k1 public key. When FindNode is received, the recipient should reply with
[Neighbors] packets containing the closest 16 nodes to target found in its local table.

FindNode 패킷은 `target`과 가까운 노드들에 대한 정보를 요청합니다. `target`은 64바이트 secp256k1 공개키입니다.
FindNode를 전달받을때 수신자는 수신자의 로컬 테이블에서 찾은 타겟에 가장 가까운 16개의 노드들을 담은 [Neighbors] 패킷을 응답해야 합니다.

To guard against traffic amplification attacks, Neighbors replies should only be sent if
the sender of FindNode has been verified by the endpoint proof procedure.

트래픽 증폭 공격을 방어하기 위해 FindNode의 발신자가 endpoint 증명 절차에 의해 검증되었을때에만 응답 패킷인 Neighbors 패킷이 전송되어야 합니다.

### Neighbors Packet (0x04)

    packet-data = [nodes, expiration, ...]
    nodes = [[ip, udp-port, tcp-port, node-id], ...]

Neighbors는 [FindNode]에 대한 응답입니다.

### ENRRequest Packet (0x05)

    packet-data = [expiration]

When a packet of this type is received, the node should reply with an ENRResponse packet
containing the current version of its [node record].

어떤 노드가 해당 타입에 대한 패킷을 받았을때, 노드는 최근 버전의 [node record] ENRResponse 패킷으로 응답해야 합니다.

To guard against amplification attacks, the sender of ENRRequest should have replied to a
ping packet recently (just like for FindNode). The `expiration` field, a UNIX timestamp,
should be handled as for all other existing packets i.e. no reply should be sent if it
refers to a time in the past.

트래픽 증폭 공격을 방어하기 위해서는 ENRRequest 발신자는 최근의 ping 패킷으로 응답해야 합니다.(이는 FindNode와 동일합니다.)
`expiration` 필드는 a UNIX timestamp 입니다. 그리고 `expiration` 필드는 존재하는 다른 모든 패킷과 동일하게 다루어져야 합니다.

### ENRResponse Packet (0x06)

    packet-data = [request-hash, ENR]

이 패킷은 ENRRequest에 대한 응답입니다.
- `request-hash`은 응답으로 보내질 ENRRequest 패킷에대한 해시값입니다.
- `ENR`은 노드 레코드입니다.

The recipient of the packet should verify that the node record is signed by the public key
which signed the response packet.

패킷의 수신자는 노드 레코드가 응답 패킷에서 서명받은 공개키로 노드기록의 서명도 검증해야합니다.
다시 말해서, 노드 기록이 올바르게 서명되었는지 확인하기 위해, 
그 서명은 응답 패킷을 서명한 것과 동일한 공개 키에 의해 서명된 것이어야 한다는 뜻입니다.

# Change Log

## Known Issues in the Current Version

The `expiration` field present in all packets is supposed to prevent packet replay. Since
it is an absolute time stamp, the node's clock must be accurate to verify it correctly.
Since the protocol's launch in 2016 we have received countless reports about connectivity
issues related to the user's clock being wrong.

모든 패킷에서 보여지는 `expiration` 필드는 패킷 재전송을 방지하기 위해 존재합니다.
절대시간 타임스탬프 이기때문에 노드의 clock은 올바르게 검증되기 위해 정확해야 합니다.
프로토콜이 2016년에 개시되면서부터 사용자들의 clock 오류과 관련된 연결이슈들을 수없이 받아왔습니다. 

The endpoint proof is imprecise because the sender of FindNode can never be sure whether
the recipient has seen a recent enough pong. Geth handles it as follows: If no
communication with the recipient has occurred within the last 12h, initiate the procedure
by sending a ping. Wait for a ping from the other side, reply to it and then send
FindNode.

엔드포인트 증명은 FindNode의 발신자 입장에서 수신자가 최근 충분한 pong을 확인했는지 알기 애매한 부분이 있습니다.
이부분에 대해 Geth에서는 다음과 같이 처리합니다: 만약 12시간 이내에 수신자와 어떤 소통도 없었다면 ping을 보냄으로서 절차를 시작해야 합니다.
상대방으로 부터 ping을 기다린후 그것에 응답하고 FindNode을 보내야합니다.

## EIP-868 (October 2019)

[EIP-868]는 [ENRRequest]와 [ENRResponse] 패킷을 추가합니다.
로컬 ENR 시퀀스 번호를 포함하여 [Ping]과 [Pong]또한 수정합니다.

## EIP-8 (December 2017)

[EIP-8] mandated that implementations ignore mismatches in Ping version and any additional
list elements in `packet-data`.

[EIP-8]에서는 구현할때 Ping버전의 불일치와 `packet-data`에서 모든 추가적인 리스트 요소를 무시하도록 지정했습니다.


# 요약

#### Node Discovery Protocol
본 명세는 Node Discovery protocol version 4에 대해 설명합니다.
Node Discovery protocal은 Kademlia-like 분산 해시테이블이며 이더리움의 노드 정보를 저장합니다.
Kademlia 구조를 사용하는 이유는 분산인덱스를 효율적으로 구성하고 짧은 경로를 가지는 네트워크 경로를 형성하기 때문입니다.

#### Node Identities
각 노드는 고유한 identity를 가집니다. 
노드의 공개키는 식별자로서 사용되는데 이 공개키를 사용하여 두 노드 사이의 거리를 알 수 있습니다.

#### Node Records
Discovery Protocol의 참여자들은 "v4" identity 방식으로 [node record],ENR을 최신 정보로 유지해야 합니다.
그리고 노드들은 [ENRRequest] 패킷을 보내어 로컬 레코드를 요청하거나 [FindNode] 패킷을 보내어
Kademlia lookup을 실행할 수 있습니다.

#### Kademlia Table
Discovery Protocol에서 노드 주위에 있는 이웃노드들은 'k-buckets'으로 구성된 라우팅 테이블에 저장됩니다.
'k-buckets'으로 구성된다는건 `0 ≤ i < 256` 사이에 있는 각각의 i에 대해, 모든 노드는 자신으로부터 `2^i ≤ distance < 2^(i+1)`
만큼의 거리를 유지하는 노드들을 리스팅 하는것인데,
각 노드가 자신과 위와 같은 규칙으로 일정한 거리를 유지하는 이웃노드들로 구성한 리스트로 저장됩니다.

'k-buckets'에는 최대 16개('k=16')의 노드 엔트리를 가질수 있는데, 가장 오래전에 조회안 노드를 맨 앞에 정렬시킵니다.
그리고 가장 최근에 조회한 노드를 맨 뒤에 정렬시킵니다. 새로운 노드 N₁이 발경되면 해당 버킷에 추가될 수 있는데
만약 bucket에 노드 엔트리가 'k=16' 보다 적으면 새로운 노드 N₁는 마지막에 정렬되고
bucket이 가득 찼다면 가장 마지막으로 조회된 노드 N₂에게 [Ping] 패킷을 보내면서 살아있는지 확인합니다.
[Ping] 패킷을 보내서 응답이 없다면 해당 노드는 죽은것으로 간주되어 bucket에서 지워집니다.
그 다음 발견한 새로운 노드인 N₁은 bucket의 마지막에 추가됩니다.

#### Endpoint Proof
구현은 쿼리의 발신자가 discovery protocol에 참여중인지 검증해야 하고 이는 트래픽 증폭 공격을 방지하기 위함입니다.
해당 검증과정은 발신자가 유효한 [Pong] 응답값을 보냈는지 확인함으로써 이루어집니다.
유효한 [Pong] 응답값은 최근 12시간 이내에 [Ping]패킷의 해시값과 매칭되어야 합니다.

#### Recursive Lookup
'lookup'은 자신과 가까운 노드들을 선별해서 'k'개의 노드를 선별하는 과정입니다.
lookup initiator 가 우선 자신이 알고있는 가장 가까운 'a'개의 노드에게 [FindNode] 패킷을 재귀적으로 보냄으로서
원활히 응답하지 못한 노드들은 고려대상에서 제거하면서 'k'개의 노드를 찾습니다.

FindNode 쿼리 단계에서 먼저 찾았던 가까운 노드보다 더 가까운 노드를 찾지 못하면,
lookup initiator는 아직 쿼리되지 않았으며 가장 가까운 `k` 모두에게 FindNode를 재전송합니다.
lookup은 initiator가 찾은 가장 가까운 `k`개의 노드들로부터 응답을 받거나 그들을 쿼리했을때 종료됩니다.

#### Wire Protocol

Wire Protocol에서
Node discovery 메시지는 UDP 데이터그램으로서 보내집니다. 
모든 패킷의 크기는 1280바이트로 제한되고 다음과 같은 헤더로 시작합니다.

    packet-header = hash || signature || packet-type
    hash = keccak256(signature || packet-type || packet-data)
    signature = sign(packet-type || packet-data)

- `hash` : 동일한 UDP 포트에서 다양한 프로토콜이 실행될때 패킷 형식을 구분하기 위해 존재합니다.
- `signature` : 서명값에서 `r`, `s` 와 복구 ID인 `v`를 연결하여 65길이의 바이트 배열로 인코딩 됩니다.
- `packet-type` : 메시지의 타입을 결정하는 단일 byte 입니다. 유효한 패킷 타입은 아래에 정리되어있습니다.

### packet-type
- Ping Packet (0x01) : 해당 패킷의 수명과, 수신자와 발신자의 정보를 포함하고 있습니다.
해당 패킷을 받으면 수신자는 반드시 [Pong]으로 응답합니다. 12시간 이내로 소통이 없다면 추가 ping을 보내어 endpoint 증명을 받아야합니다. 구성은 다음과 같습니다.

      packet-data = [version, from, to, expiration, enr-seq ...]
      version = 4
      from = [sender-ip, sender-udp-port, sender-tcp-port]
      to = [recipient-ip, recipient-udp-port, 0]

- Pong Packet (0x02) : Ping에 대한 응답입니다.응답할 목적지([Pong] 발신자)와 ping-hash, 최근 [Pong] 발신자의 ENR 시퀀스 번호를 포함합니다. 구성은 다음과 같습니다.

      packet-data = [to, ping-hash, expiration, enr-seq, ...]

- FindNode Packet (0x03) : `target`과 가까운 노드정보들 요청합니다. 해당 패킷에 대한 응답은 Neighbors 입니다. 해당 응답은 endpoint 증명절차로 검증되어졌을때만 전송합니다. 구성은 다음과 같습니다.

      packet-data = [target, expiration, ...]

- Neighbors Packet (0x04) : [FindNode]에 대한 응답입니다. 구성은 다음과 같습니다.

      packet-data = [nodes, expiration, ...]
      nodes = [[ip, udp-port, tcp-port, node-id], ...]

- ENRRequest Packet (0x05) : 이 패킷을 수신받으면 최신버전의 [node record]을 담은 ENRResponse 패킷으로 응답해야합니다. 트래픽 증폭 공격을 방지하기 위해 ENRRequest 발신자는 최근의 ping 패킷으로 응답해야 합니다. 구성은 다음과 같습니다.
    
        packet-data = [expiration]

- ENRResponse Packet (0x06) : ENRRequest에 대한 응답입니다. 구성은 다음과 같습니다.

      packet-data = [request-hash, ENR]

#### Known Issues in the Current Version
최근 버전에서 몇몇 알려진 이슈가 있습니다.
위 패킷타입들에 있는 `expiration` 필드는 패킷 재전송을 방지하기 위해 존재합니다.
이 필드는 절대 시간 타임스탬프 이기때문에 노드의 clock은 `expiration`을 올바르게 검증되기 위해 반드시 정확해야 합니다.

그러나 엔드포인트 증명은 FindNode의 발신자 입장에서 수신자가 최근 충분한 pong을 확인했는지 알기 애매한 부분이 있습니다. 
이부분에 대해 Geth에서는 다음과 같이 처리합니다: 만약 12시간 이내에 수신자와 어떤 소통도 없었다면 ping을 보냄으로서 절차를 시작해야 합니다.
상대방으로 부터 ping을 기다린후 그것에 응답하고 FindNode을 보내야합니다.

#### EIP-868 (October 2019)

[EIP-868]는 [ENRRequest]와 [ENRResponse] 패킷을 추가합니다.
로컬 ENR 시퀀스 번호를 포함하여 [Ping]과 [Pong]또한 수정합니다.

#### EIP-8 (December 2017)

[EIP-8]에서는 구현할때 Ping버전이 일치하지 않는부분과 
`packet-data`에서 모든 추가적인 리스트 요소(any additional
list elements)를 무시하도록 지정했습니다.

[Ping]: #ping-packet-0x01
[Pong]: #pong-packet-0x02
[FindNode]: #findnode-packet-0x03
[Neighbors]: #neighbors-packet-0x04
[ENRRequest]: #enrrequest-packet-0x05
[ENRResponse]: #enrresponse-packet-0x06
[EIP-8]: https://eips.ethereum.org/EIPS/eip-8
[EIP-868]: https://eips.ethereum.org/EIPS/eip-868
[node record]: ./enr.md