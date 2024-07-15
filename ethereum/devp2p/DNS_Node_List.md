# DNS Node Lists (원문 : [DNS Node Lists])

DNS는 인터넷에서 도메인 이름을 IP 주소로 변환하는데 사용되는 프로토콜입니다.
사용자가 기억하기 쉬운 도메인 이름을 이용해 원하는 웹사이트나 서비스를 쉽게 이용할 수 있도록 도와줍니다.
DNS Node Lists는 DNS를 통해 제공되는 노드 목록들을 의미하는데 이는 분산 네트워크 또는 P2P 네트워크에 참여하는 노드들의 목록을 의미합니다.
각 노드들은 서로 연결할 수 있는 정보(예: IP 주소, 포트 번호 등)를 포함하고 있습니다.

Peer To Peer 노드 소프트웨어는 종종 하드코딩된 [부트스트랩]노드 리스트를 포함하고 있습니다.
쉽게 말하자면 P2P 네트워크에 참여하는 소프트웨어는 처음에 친구를 찾기 어려울 수 있기 때문에 
처음에 연락할 수 있는 친구들의 전화번호 목록(노드들의 연결정보)을 미리 가지고 있습니다.
따라서 이 목록들이 소프트웨어에 저장되어 있어서 네트워크에 쉽게 접근할수 있게되지요.
그런데 이러한 리스트를 업데이트하는건 소프웨어의 업데이트를 요구하고,
소프트웨어 유지보수자가 리스트가 최신 상태인지 확인해하는 수고도 필요합니다.
결과적으로 제공된 리스트들은 부족하고 소프트웨어가 네트워크로 진입할 수 있는 entry point를 선택할 폭이 좁습니다.

본 명세는 인증,DNS를 통해 검색 가능하고 업데이트할 수 있는 노드 리스트를 위한 체계를 설명합니다,
클라이언트는 오로지 **DNS의 이름**과 **리스트를 서명하는 공개키**에 대한 정보만 요구합니다.

DNS 기반의 탐색은 [EIP-1459]에서 처음 제안됩니다.

## Node Lists

`Node List`는 임의이 길이를 가진 ENR 리스트입니다. 리스트들은 링크를 사용하여 다른 리스트들을 참조하는데
전체 리스트는 secp256k1 개인키를 사용하여 서명되어집니다. 이에 대응되는 공개키는 클라이언트가 리스트를 검증하기 위해 알고있어야 합니다.


## URL Scheme
<!--
To refer to a DNS node list, clients use a URL with 'enrtree' scheme. 
The URL contains the DNS name on which the list can be found as well as the public key that signed the list. 
The public key is contained in the username part of the URL and is the base32 encoding of the compressed 32-byte binary public key.
-->

DNS 노드 리스트를 참조하기 위해서는 클라이언트가 `enrtree` 방식을 사용하는 URL을 사용합니다.
`enrtree`은 Ethereum Node Records tree의 약자로 Ethereum P2P 네트워크에서 노드를 효율적으로 검색을 할 수 있는 시스템입니다.
해당 방식의 URL에는 리스트가 위치한 DNS 이름과 그 리스트에 서명한 공개 키가 포함되어 있습니다.
이런 공개키는 URL의 일부분중 사용자 이름에 포함되어 있고 32-byte로 압축된 바이너리 공개키로 base32 인코딩 되어있습니다.


Example:

    enrtree://AM5FCQLWIZX2QFPNJAP7VUERCCRNGRHWZG3YYHIUV7BVDQ5FDPRT2@nodes.example.org

위의 URL은 `nodes.example.org` 라는 DNS 이름의 노드 리스트를 참조하고 있고 공개키에 의해서 서명되어졌습니다.
아래는 인코딩된 공개키입니다.

    0x049f88229042fef9200246f49f94d9b77c4e954721442714e85850cb6d9e5daf2d880ea0e53cb3ac1a75f9923c2726a4f941f7d326781baa6380754a360de5c2b6

## DNS Record Structure

머클트리의 엔트리(노드 정보)는 DNS TXT 레코드형식으로 존재합니다.
TXT 레코드는 텍스트 정보를 포함하는 레코드 유형인데, [SPF(Sender Policy Framework)] 정보 [DKIM(DomainKeys Identified Mail)] 정보 등을
설정하는 데에 사용됩니다. 트리의 루트는 아래의 컨텐츠처럼 TXT 레코드를 포함하고 있습니다.

    enrtree-root:v1 e=<enr-root> l=<link-root> seq=<sequence-number> sig=<signature>

where

- `enr-root` 와 `link-root` 노드와 링크 서브트리를 포함하는 서브트리의 루트 해시를 참조합니다.
- `sequence-number`는 트리의 시퀸스 숫자를 업데이트 합니다, 10진수 정수입니다.
- `signature`는 기록 내용의 keccak256 해시위에 65-byte의 secp256k1 EC 서명되어진 것 입니다.
  `sig=` 부분을 제외하고 URL-safe base64로 인코딩되어 있습니다.

서브도메인에서 추가적인 TXT레코드들은 세가지 유형의 엔트리 타입중 하나로 매핑됩니다.
모든 엔트리에서 서브도메인의 이름은 해당 엔트리의 텍스트 컨텐츠를 keccak256 해시값을 base32로 인코딩한것입니다.

아래는 앞서 말한 매팽되는 3가지 엔트리 타입입니다: 

- `enrtree-branch:<h₁>,<h₂>,...,<hₙ>`는 서브트리 엔트리의 해시값을 포함하고 있는 중간 트리 엔트리입니다.
- `enrtree://<key>@<fqdn>`는 리프노드인데 완전한 자격이 있는 도메인 이름(FQDN)에 위치한 리스트들을 가리킵니다.
   URL 인코딩과 매치된 해당 형식을 주목하셔야합니다. 이런 유형의 엔트리는 `link-root`가 가리키는 서브트리에만 나타날 수 있습니다.
- `enr:<node-record>`는 노드 레코드를 포함한 리프노드 입니다. 해당 노드레코드는 URL-safe base64 문자열로 인코딩 되어있습니다.
  이런 유형의 엔트리는 표준 ENR 문자로 인코딩되는것과 일치한다는것을 주목해야 합니다. `enr-root` 서브트리 안에서 나타날 수 있습니다.

특정 순서나 구조가 트리를 정의하지는 않습니다. 트리가 업데이트가 될 때마다 해당 시퀀스 넘버가 증가해야 합니다. 
각 엔트리에서 TXT 레코드의 내용은 UDP DNS 패킷에서 제한하는 512 바이트의 크기에 맞도록 충분히 최소화 해야합니다.
이는 `enrtree-branch` 엔트리로 포함될 수 있는 해시값들의 수를 제한하지요.

아래는 위의 3가지 유형의 예시를 보여줍니다.

    ; name                        ttl     class type  content
    @                             60      IN    TXT   enrtree-root:v1 e=JWXYDBPXYWG6FX3GMDIBFA6CJ4 l=C7HRFPF3BLGF3YR4DY5KX3SMBE seq=1 sig=o908WmNp7LibOfPsr4btQwatZJ5URBr2ZAuxvK4UWHlsB9sUOTJQaGAlLPVAhM__XJesCHxLISo94z5Z2a463gA
    C7HRFPF3BLGF3YR4DY5KX3SMBE    86900   IN    TXT   enrtree://AM5FCQLWIZX2QFPNJAP7VUERCCRNGRHWZG3YYHIUV7BVDQ5FDPRT2@morenodes.example.org
    JWXYDBPXYWG6FX3GMDIBFA6CJ4    86900   IN    TXT   enrtree-branch:2XS2367YHAXJFGLZHVAWLQD4ZY,H4FHT4B454P6UXFD7JCYQ5PWDY,MHTDO6TMUBRIA2XWG5LUDACK24
    2XS2367YHAXJFGLZHVAWLQD4ZY    86900   IN    TXT   enr:-HW4QOFzoVLaFJnNhbgMoDXPnOvcdVuj7pDpqRvh6BRDO68aVi5ZcjB3vzQRZH2IcLBGHzo8uUN3snqmgTiE56CH3AMBgmlkgnY0iXNlY3AyNTZrMaECC2_24YYkYHEgdzxlSNKQEnHhuNAbNlMlWJxrJxbAFvA
    H4FHT4B454P6UXFD7JCYQ5PWDY    86900   IN    TXT   enr:-HW4QAggRauloj2SDLtIHN1XBkvhFZ1vtf1raYQp9TBW2RD5EEawDzbtSmlXUfnaHcvwOizhVYLtr7e6vw7NAf6mTuoCgmlkgnY0iXNlY3AyNTZrMaECjrXI8TLNXU0f8cthpAMxEshUyQlK-AM0PW2wfrnacNI
    MHTDO6TMUBRIA2XWG5LUDACK24    86900   IN    TXT   enr:-HW4QLAYqmrwllBEnzWWs7I5Ev2IAs7x_dZlbYdRdMUx5EyKHDXp7AV5CkuPGUPdvbv1_Ms1CPfhcGCvSElSosZmyoqAgmlkgnY0iXNlY3AyNTZrMaECriawHKWdDRk2xeZkrOXBQ0dfMFLHY4eENZwdufn1S1o

###### *ttl(Time to Live)는 DNS 레코드의 유효시간을 의미합니다. 이는 레코드가 캐시되는 시간을 의미합니다.

## Client Protocol

mynodes.org라는 DNS 이름에서 노드를 찾는다고 가정하면 아래의 단계를 따라야 합니다 :

1. 이름과 유효한 "enrtree-root=v1" 엔트리의 포함여부를 확인하고 TXT 레코드의 이름을 해석해야합니다.
   엔트리에 포함된 `enr-root` 해시가 "CFZUWDU7JNQR4VTCZVOJZ5ROV4" 라고 가정해봅시다.
2. 루트노드에서 알려진 공개키에 대하여 서명을 검증하고 시퀀스 숫자가 해당 이름에서 이전에 본 숫자보다 크거나 같은지 확인합니다.
3. 서브도매인 해시의 TXT 기록을 해석해야 합니다 예를들어 "CFZUWDU7JNQR4VTCZVOJZ5ROV4.mynodes.org" 를 해석하고
   해당 해시와 내용이 일치하는지 검증해야합니다.
4. The next step depends on the entry type found:
    - for `enrtree-branch`: parse the list of hashes and continue resolving them (step 3).
    - for `enr`: decode, verify the node record and import it to local node storage.
4. 다은 단계는 주어진 엔트리 유형에 따라 달라집니다.
   - `enrtree-branch`의 경우 : 해시들의 리스트를 파싱하고 해시들을 계속 해석해야합니다.(step3)
   - `enr`의 경우 : 디코딩해야 합니다, 노드레코드를 검증한 후 로컬 노드 스토리지에 추가(import)해야 합니다. 

mynodes.org라는 DNS 이름을 탐색하는 동안 클라이언트는 해시값과 무한루프를 사전방지한 도메인을 추적해야합니다.
때문에 임의의 순서로 트리를 탐색하는것이 클라이언트에게 가장 좋습니다. 
그리고 클라이언트 구현부는 일반적인 동작중에 전체 트리를 한번에 다운받는것을 피해야합니다.
필요할때만 DNS를 통해 엔트리를 요청하는것이 훨씬 좋습니다.

## Rationale : Reason for using DNS

DNS는 항상 가용성있는 저지연 프로토콜이기 때문에 사용됩니다. 쉽게말하자면 DNS는 인터넷이 존제하는 한 항상 사용이 가능합니다.
전 세계적으로 표준화 되어있고 모든 인터넷 연결장치에서 사용하기 때문에 신뢰할수 있지요.

머클트리가 된다는건 모든 노드리스트들이 루트노드에서 단일 서명으로 인증/인가가 될 수 있다는 것을 의미합니다.
해시 서브도메인은 리스트의 무결성을 보호합니다. 최악의 경우 중간 리졸버는 리스트에 대한 접근을 차단하거나 업데이트를 하는것을 금지합니다,
그러나 해당 리스트의 내용을 오염시킬순 없습니다. 시퀀스 숫자가 루트노드가 이전 버전으로 대체되는것을 방지하기 때문이죠.

클라이언트 사이드에서 업데이트를 동기화 하는것은 점진적으로 실행될 수 있는데, 이는 규모가 큰 리스트에서 중요합니다.
트리에서 개별적인 엔트리들은 단일 UDP 패킷맞게 충분히 작습니다 때문에 기본 UDP DNS만 사용되는 환경에서 호환성을 보장합니다.
트리 형식은 또한 캐싱 리졸버와 무난한게 작동 됩니다. 트리의 루트노드만 짧은 TTL을 필요로 합니다. 
중간 엔트리들과 리프들은 한동안 캐싱될 수 있습니다.

### Why does the link subtree exist?

리스트들 사이의 연결들은 여러 독립적인 시스템이나 사용자가 협력할 수 있게하고 분산된 신뢰 네트워크를 구축할 수 있게 합니다.
큰 리스트의 오퍼레이터는 다른 리스트 제공자에게 유지보수를 위임할 수 있습니다. 만약 두개의 노드리스트가 서로 연결되어 있다면
유저들은 어떤리스트를 사용하든 두 리스트들로부터 노드를 얻을 수 있습니다.

링크 서브트리는 ENR을 포함하는 트리와 별도로 존재합니다. 클라이언트 구현함에 있어 독립된 트리들과 동기화되는것을
보장하기 위함입니다. 가능한 많은 노드들을 얻고자 하는 클라이언트는 우선 링크트리를 동기화하고 동기화 범위내에 연결된 모든 이름들을 추가할 것입니다.

## References
1. base64와 base32 인코딩은 [RFC 4648] 정의되어 있으며 바이너리데이터를 표현하는데 사용됩니다. 
   base64와 base32 데이터에는 패딩이 사용되지 않습니다.


# 요약
#### DNS Node Lists
본문은 DNS를 통해 제공되는 노드 리스트들을 설명하고 있습니다.
P2P 노드 소프트웨어의 한계점 때문에 DNS를 사용하여 노드 리스트를 관리하는 방법을 제안하고 있습니다
#### Node Lists
노드리스트는 임의의 길이를 가진 ENR 리스트이며 해당 리스트들은 링크를 사용하여 다른 리스트들을 참조합니다.
전체 리스트는 secp256k1 개인키를 사용하여 서명되어집니다. 서명된 리스트는에 대응하는 공개키는 클라이언트가 검증하기 위해 알고있어야 합니다.
####  URL Scheme
다른 리스트들을 참조하는데에 사용되는 URL 방식이 존재하는데 이는 `enrtree` 방식을 사용합니다.
이더리움 네트워크에서 노드를 효율적으로 검색할 수 있는 방식이죠.
`enrtree` 방식을 사용한 URL 링크는 특정 인코딩 방식으로 표현됩니다.
#### DNS Record Structure
DNS 레코드 구조는 우선 리스트의 노드들은 DNS 프로토콜에 배포되기 위해 머클트리 방식으로 인코딩 됩니다.
이는 분산네트워크에서 노드 리스트를 관리하기 위해 DNS를 사용하고 머클트리를 사용하여 데이터의 무결성을 보장합니다.
여기서 머클트리의 각 엔트리들은 DNS TXT 레코드 방식으로 존재합니다.
머클트리의 엔트리,서브도메인에 추가되는 TXT 레코드 이는 3가지 엔트리 타입으로 매핑됩니다.
각 엔트리에서 TXT 레코드의 크기는 UDP DNS 패킷에 맞춰서 512바이트의 크기로 충분히 최소화 되어야 합니다.
#### Client Protocol
클라이언트 프로토콜로에서 DNS 이름을 통해 노드를 찾는 방법을 설명하고 있습니다.
#### Rationale : Reason for using DNS
DNS를 사용하는 이유에 대해 설명하고 있습니다.
DNS는 항상 가용성있는 저지연 프로토콜입니다. 이는 인터넷이 존재하는 한 항상 사용이 가능하다는 것을 의미합니다.
분산데이터를 위해 노드리스트들은 DNS에 배포하고 해당 노드리스트는 머클트리로 인코딩되어져 배포됩니다.
쉽게 말하자면 데이터 무결성을 보장하기 위해 머클트리로 노드리스트를 관리하고 저지연 프로토콜이면서 항상 사용가능한 DNS의 장점을 이용하고 있습니다.
그리고 클라이언트에서 업데이트 할때 점진적으로 업데이트를 동기화할 수 있는데 특히 규모가 큰 리스트를 관리할때 중요한 부분입니다.
트리의 투르노드는 짧은 TTL을 가지는데 이는 빈번하게 업데이트 되어야 하기 때문입니다. 반면 중간 엔트리들과 리프들은 몇일동안 캐싱될 수 있습니다.
#### link subtree의 존재이유
리스트들 사이의 링크는 신뢰성 있는 분산네트워크를 구축할 수 있게 합니다.
규모가 큰 리스트는 다른 리스트 제공자에게 유지보수를 위임할 수 있습니다.
만약 두개의 노드리스트가 서로 연결되어 있다면 유저는 어떤 리스트를 사용하든 두 리스트들로부터 노드를 얻을수 있지요.
그리고 link subtree는 ENR을 포함하는 트리와 별도르 존재합니다. 클라이언트 구현부는 독립된 트리들과 동기화되는것을 보장하기 위함입니다.


[부트스트랩]: https://en.wikipedia.org/wiki/Bootstrapping
[EIP-1459]: https://eips.ethereum.org/EIPS/eip-1459
[RFC 4648]: https://tools.ietf.org/html/rfc4648
[DNS Node Lists]: https://github.com/ethereum/devp2p/blob/master/dnsdisc.md
