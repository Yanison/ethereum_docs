# Ethereum Node Records

본 명세는 Ehtereum Node Records(ENR)을 정의한다. p2p 연결 정보를 위한 오픈 포멧이다.
하나의 노드는 ip 주소와 포트번호들과 같은 노드의 네트워크의 엔드포인트를 담고있다.
또한 다른 노드들이 해당 노드와의 연결여부를 결정할 수 있는 네트워크상의 노드의 목적에 관한 정보들을 담고있다.
 
Ethereum Node Records 는 [EIP-778]에서 처음 제안되었다.

## Record Structure

node record의 구성은 다음과 같다.
- `signature` : 레코드 내용의 암호화 서명
- `seq`: 64-bit의 unsigned integer인 시퀀스 번호. 노드들은 레코드가 변할때 혹은 재발행될때마다 이 시퀀스 숫자를 증가시켜야한다.
- 레코드의 나머지 구성요소는 임의의 키/값 쌍으로 이루어져있다.

하나의 레코드 서명은 *identity scheme* 에 따라 검증되고 생성된다.
*identity scheme* 는 DHT에서 노드의 주소를 도출하는 담당을한다.

좀 더 자세히 말하자면 분산 네트워크에서 각 노드들은 자신의 고유의 주소값을 가져야 하는데 
이 고유 주소는 분산 해시테이블(DHT)에서 데이터 저장 위치를 결정하는데 사용된다.
Identity scheme는 노드의 고유 식별자를 기반으로 이 주소값을 유도한다. 예를들어
노드의 기록들을 구성하는 내용들로 해시함수에 입력하여 얻은 값을 주소값으로 사용할 수 있다. 

<!--
The key/value pairs must be sorted by key and must be unique, i.e. any key may be present
only once. The keys can technically be any byte sequence, but ASCII text is preferred. Key
names in the table below have pre-defined meaning.
-->

key/value 쌍은 key에 의해서 정렬되고 key값들은 반드시 unique해야 한다. 즉 어떤 key도 한번만 나타날 수 있다.
key들은 기술적으로 어떻게든 byte sequence가 될수 있지만 ASCII 텍스트가 선호된다.
아래의 테이블은 미리 정의된 의미를 가진 key들을 나타낸다.

| Key         | Value                                      |
|:------------|:-------------------------------------------|
| `id`        | name of identity scheme, e.g. "v4"         |
| `secp256k1` | compressed secp256k1 public key, 33 bytes  |
| `ip`        | IPv4 address, 4 bytes                      |
| `tcp`       | TCP port, big endian integer               |
| `udp`       | UDP port, big endian integer               |
| `ip6`       | IPv6 address, 16 bytes                     |
| `tcp6`      | IPv6-specific TCP port, big endian integer |
| `udp6`      | IPv6-specific UDP port, big endian integer |


| Key         | Value                                      |
|:------------|:-------------------------------------------|
| `id`        | identity scheme의 이름, e.g. "v4"           |
| `secp256k1` | 33 bytes의 secp256k1로 압축된 공개키          |
| `ip`        | IPv4 주소, 4 bytes                          |
| `tcp`       | TCP port, big endian integer               |
| `udp`       | UDP port, big endian integer               |
| `ip6`       | IPv6 address, 16 bytes                     |
| `tcp6`      | IPv6-specific TCP port, big endian integer |
| `udp6`      | IPv6-specific UDP port, big endian integer |

*Big endian는 데이터 저장 방식에 사용되는 용어이다. 데이터를 전송하거나 저장할때의 바이트 순서를 설명한다.
가장 중요한 바이트를 가장 앞에 배치하는 방식인데 이를 통해 숫자 크기나 중요도가 큰 쪽부터 차례로 저장한다.

쉽게말하면 큰 숫자가 가장 앞에 오는 방식이다. 
Big endian과 반대되는 방식은 Little endian도 있는데 이 두가지 방식은 프로세서 아키텍처나 네트워크 프로토콜에서
데이터를 해석하는 방식에 따라 사용된다.

All keys except `id` are optional, including IP addresses and ports. A record without
endpoint information is still valid as long as its signature is valid. If no `tcp6` /
`udp6` port is provided, the `tcp` / `udp` port applies to both IP addresses. Declaring
the same port number in both `tcp`, `tcp6` or `udp`, `udp6` should be avoided but doesn't
render the record invalid.

모든 키는 ip주소와 포트를 포함해 `id`를 옵션으로 가진다. 
하나의 노드에서 엔드포인트 정보가 없어도 그것의 서명이 유효다면 여전히 유효하다.
만약 `tcp6`/`udp6` 포트가 제공되더라도, `tcp6`/`udp6` 포트는 두개의 IP 주소에 모두 적용된다.
`tcp`,'tcp6' 또는 `udp`,`udp6`에 같은 포트 숫자가 선언되는것은 지양해야 하지만 레코드 자체를 유효하지 않게 만들지는 않는다.

### RLP Encoding

RLP(Recursive Length Prefix)는 주로 이더리움과 같은 블록체인 시스템에서 데이터를 인코딩 하는데 사용되는 방식이다.
RLP의 주요 목표는 임이의 데이터 구조를 효율적으로 인코딩하고 디코딩 하는 것이다. RLP는 다음과 같은 규칙에 따라 데이터를 인코딩한다.

1. 단일 바이트 (0x00 ~ 0x7F) : 단일 바이트는 그대로 인코딩 된다.
2. 짧은 문자열(0x00 ~ 0x37) : 문자열의 길이와 문자열 데이터를 결합하여 인코딩된다.
3. 긴 문자열: 길이를 먼저 인코딩한 후 문자열 데이터를 인코딩한다.
4. 리스트 : 리스트의 전체 길이를 먼저 인코딩하고  리스트의 각 항목을 순서대로 인코딩한다.

노드 레코드의 표준 인코딩은 RLP 리스트로 `[signature, seq, k, v, ...]` 이다.
하나의 노드의 최대 인코딩 사이즈는 300 bytes이다. 구현체들은 이 사이즈보다 큰 레코드를 거부해야 한다.

레코드들은 다음과 같이 인코딩되고 서명된다 : 

    content   = [seq, k, v, ...]
    signature = sign(content)
    record    = [signature, seq, k, v, ...]

### Text Encoding

노드 레코드의 텍스트 형식은 RLP 형식의 base64 인코딩 방식이다. prefix로 'enr:' 을 사용한다.
구현체들은 [URL-safe base64 alphabet] 사용하고 패딩 문자를 생략해야 한다.

### "v4" Identity Scheme

This specification defines a single identity scheme to be used as the default until other
schemes are defined by further EIPs. The "v4" scheme is backwards-compatible with the
cryptosystem used by Node Discovery v4.

이 명세는 단일 identity scheme를 정의하는데 다른 schemes들이 이후 EIPs로 정의되어질 때 까지 기본으로 사용된다.
`v4` shceme는 Node Discovery v4에서 사용되는 암호 시스템과 호환된다.

*[Node Discovery Protocol v4](https://github.com/ethereum/devp2p/blob/master/discv4.md)<br>
이더리움 노드들에 대한 정보를 저장하는 Kademlia와 유사한 DHT(Distributed Hash Table), 분산 해쉬테이블이다.
Kademlia 구조는 노드의 분산 인덱스와 low diameter 토플로지 구조를 얻는 효율적인 방법이기 때문에 선택되었다. 

- "v4" Identity scheme 에서 `content`를 서명하기 위해서 keccak256 해시 함수(EVM에서 사용되는)를 적용한다.
  그리고 해쉬값의 서명을 생성한다. 64-byte의 서명은 `r` 과 `s` 서명값으로 concatenation으로 인코딩된다.
  (복구 ID `v`는 생략된다.)
  *concatenation : 문자열이나 데이터를 하나로 연속된 형태로 만드는 것을 의미.
- 레코드를 검증하기 위해서 서명이 레코드의 "secp256k1" key/value 쌍으로 공개키가 생성되어졌는지 확인해야 한다.
*secp256k1 : 특정 타원 곡선의 알고리즘 이름인데, 특정한 수학적 곡선 기반으로 한 공개키 암호화 방식이다. 암호화폐의 디지털 서명및 키 생성에 사용된다.

  `keccak256(x || y)`. Note that `x` and `y` must be zero-padded up to length 32.
- 노드 주소를 유도하기위해 압축되지 않은 공개키의 keccak256 해쉬를 취한다. 즉 `keccak256(x || y)` 이다.
  `x` 와 `y`는 길이가 32byte여야 하며 32byte보다 짧을경오 0을 추가하여 패딩한다.
* x || y : x와 y를 연결한 것을 의미한다. (concatenation)
* 
## Rationale

The format is meant to suit future needs in two ways:
이 포맷은 미래의 요구사항을 충족시키기 위해 두가지 방법으로 사용된다.

- 새로운 키/값 쌍 추가 : 이것은 항상 가능하고 구현체가 합의하는것을 요구하지 않는다. 존재하는 클라이언트는
  어떤 키/값 쌍이든 그들이 내용을 해석할 수 있는지와 상관없이 받아들일 것이다.
- Adding identity schemes: these need implementation consensus because the network won't
  accept the signature otherwise. To introduce a new identity scheme, propose an EIP and
  get it implemented. The scheme can be used as soon as most clients accept it.
- 새로운 identity scheme 추가 : 이것은 구현체의 합이가 필요합니다 왜냐하면 해당 네트워크가 이 서명을 
  받아들이지 않을수도 있기 때문입니다. 새로운 identity scheme을 도입하기 위해, EIP를 제안하고 구현해야합니다.
  대부분의 클라이언트가 이를 받아들이면 이 scheme을 사용할 수 있습니다.

레코드의 크기는 제한되어져있는데 레코드는 빈번하게 전달되어지고 DNS와 같은 프로토콜에서 크기 제약을 포함할 수 있기 때문입니다.
하나의 레코드는 하나의 IPv4 주소가 포함되고 `v4`를 사용하여 서명된 scheme는 대략 120바이트를 차지합니다.
추가적인 metadata를 위해 충분한 공간을 남겨두기 위함입니다.

You might wonder about the need for so many pre-defined keys related to IP addresses and
ports. This need arises because residential and mobile network setups often put IPv4
behind NAT while IPv6 traffic—if supported—is directly routed to the same host. Declaring
both address types ensures a node is reachable from IPv4-only locations and those
supporting both protocols.

아마 글을 읽는 당신은 IP주소와 포트번호와 연관하여 미리 정의된 키들이 왜 이렇게 많이 필요한지 의문이 들것입니다.
이러한 필요는 주거 및 모바일 네트워크 설정들이 NAT 뒤에 IPv4를 배치하기 때문에 생겨날 수 있습니다.
좀 더 자세하게 설명하자면 IPv4는 NAT 뒤에 배치하는데 NAT는 여러 장치가 공용 인터넷에 하나의 공용 IP 주소로 접근할 수 있는
기술입니다. 반면 IPv6은 더 많은 IP주소로 제공하며, NAT 없이도 각 장치에 고유한 IP주소를 부여할 수 있습니다.
따라서 네트워크가 IPv6를 지원하는 경우 NAT없이도 동일한 호스트로 직접 라우팅이 됩니다.
IPv4, IPv6 두가지 방식으로 주소 타입으로 선언하는것은 노드가 IPv4만 지원하거나 두 프로토콜을 지원하는 지역에 도달하는것을
보장하기 위함입니다.

## Test Vectors

기록에 다음과 같이 정보가 담겨져 있다 가정해봅시다.
- IPv4 : `127.0.0.1`
- UDP port : `30303`
- Node ID : `a448f24c6d18e575453db13171562b71999873db5b286df957af199ec94617f7`

    enr:-IS4QHCYrYZbAKWCBRlAy5zzaDZXJBGkcnh4MHcBFZntXNFrdvJjX04jRzjzCBOonrkTfj499SZuOh8R33Ls8RRcy5wBgmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPKY0yuDUmstAHYpMa2_oxVtw0RW_QAdpzBQA8yWM0xOIN1ZHCCdl8

The record is signed using the "v4" identity scheme using sequence number `1` and this private key:
해당 레코드는 `v4`identity scheme 를 사용하려 서명되었습니다. <br> 
그리고 시퀀스 번호는 `1`을 사용하고 다음은 이 레코드의 개인키입니다 :

    b71c71a67e1177ad4e901695e1b4b9ee17ae16c6668d313eac2f96dbcda3f291

RLP 구조의 레코드는 다음과 같습니다 :

    [
      7098ad865b00a582051940cb9cf36836572411a47278783077011599ed5cd16b76f2635f4e234738f30813a89eb9137e3e3df5266e3a1f11df72ecf1145ccb9c,
      01,
      "id",
      "v4",
      "ip",
      7f000001,
      "secp256k1",
      03ca634cae0d49acb401d8a4c6b6fe8c55b70d115bf400769cc1400f3258cd3138,
      "udp",
      765f,
    ]

[EIP-778]: https://eips.ethereum.org/EIPS/eip-778
[URL-safe base64 alphabet]: https://tools.ietf.org/html/rfc4648#section-5


# 요약
Ethereum Node Records은 P2P 연결 정보를 위한 오픈 포멧인데 JSON 처럼 생각하면 되겠다.
하나의 노드에는 IP주소와 포트번호같은 정보들을 담고있는데 이정보들은 RLP(Recursive Length Prefix)규칙에 따라 인코딩 된디.