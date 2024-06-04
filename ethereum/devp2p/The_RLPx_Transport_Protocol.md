# The RLPx Transport Protocol

This specification defines the RLPx transport protocol, a TCP-based transport protocol
used for communication among Ethereum nodes. The protocol carries encrypted messages
belonging to one or more 'capabilities' which are negotiated during connection
establishment. RLPx is named after the [RLP] serialization format. The name is not an
acronym and has no particular meaning.

본 명세는 RLPx transport protocol를 정의하고 있다.
TCP 베이스의 transport protocol로 이더리움 노드끼리 통신하기 위해 사용된다.
이 프로토콜은 통신이 연결됐을때 사용되는 여러기능들이 있는데 그 기능들중 하나로 암호회된 메시지를 주고받습니다.
RLPx는 [RLP] 직렬화 형식에서 이름을 따왔습니다. 이 이름은 약어가 아니며 특별한 의미가 없습니다.

현재 프로토콜 버전은 5입니다. 이전 버전의 변경 사항 목록은 이 문서의 끝에 있습니다.

## Notation

`X || Y`\
    X와Y의 문자열연결(concatenation)을 의미합니다.\
`X ^ Y`\
    X와Y의 각 바이트별로 XOR연산을 의미합니다.
XOR 는 두 비트가 서로 1이거나 0으로 같을때 0, 서로 다를때 1을 반환하는 연산입니다.\
`X[:N]`\
    X의 N-byte 접두어(prefix)를 의미합니다.\
    바이트 시퀀스 또는 배열의 특정 부분을 추출하는 데 사용되는 코드입니다. 예를들어 X[:3]이면 처음 3개의 바이트를 추출합니다.\
`[X, Y, Z, ...]`\
     RLP 리스트처럼 recursive encoding을 의미합니다, [RLP] 참고\
     ["cat", ["puppy", "cow"], "horse", [[]], "pig", [""], "sheep"] 처럼 복잡한 리스트형식이 가능하다는 의미.\
     recursive encoding\
`keccak256(MESSAGE)`\
    이더리움에서 사용되는 Keccak256 헤쉬함수 입니다.\
`ecies.encrypt(PUBKEY, MESSAGE, AUTHDATA)`\
    .RLPx에서 사용되는 비대칭 인증 암호화 함수(authenticated encryption function) 입니다. \
    AUTHDATA ciphertext결과값의 일부는 아니지만 인증된 데이터 입니다,\
    그러나 message tag을 생성하기 전에 HMAC-256으로 쓰여집니다.\
`ecdh.agree(PRIVKEY, PUBKEY)`\
     Elliptic Curve Diffie-Hellman, Elliptic curve(타원곡선) 암호화를 통해 multiple party간의 안전하게 합의할 수 있는 key를 도출하는 key-agreement-protocol이다.
     PRIVKEY와 PUBKEY를 매개변수로 합의되는 ECDH이다.

## ECIES Encryption

ECOES(Elliptic Curve Integrated Encryption Scheme)는 비대칭 암호화 방식이다. RLPx handshake에서 사용된다.
이 암호화방식이 RLPx에서 사용되는 방식은 다음과 같습니다.

- The elliptic curve secp256k1 with generator `G`.
- 생성자 `G` 를 가진 타원곡선(elliptic curve) secp256k1
- `KDF(k, len)`: NIST SP 800-56 Concatenation Key를 유도하는 함수
- `MAC(k, m)`: SHA-256 해시함수를 사용한 HMAC
- `AES(k, iv, m)`: AES-128 암호화 함수를 CTR mode로 사용

Alice wants to send an encrypted message that can be decrypted by Bobs static private key
<code>k<sub>B</sub></code>. Alice knows about Bobs static public key
<code>K<sub>B</sub></code>.

Alice는 암호화된 메시지를 전송하길 원합니다 이 암호화 메시지는 Bobs의 전역 개인키(private key) <code>k<sub>B</sub></code> 로 복호화될수 있습니다.
Alice는 Bobs의 전역 공개키(public key)인 <code>K<sub>B</sub></code>에 대해 알고 있습니다.

To encrypt the message `m`, Alice generates a random number `r` and corresponding elliptic
curve public key `R = r * G` and computes the shared secret <code>S = P<sub>x</sub></code>
where <code>(P<sub>x</sub>, P<sub>y</sub>) = r * K<sub>B</sub></code>. She derives key
material for encryption and authentication as
<code>k<sub>E</sub> || k<sub>M</sub> = KDF(S, 32)</code> as well as a random
initialization vector `iv`.

메시지 `m`을 암호화하기 위해, Alice는 난수 `r`을 생성하고 이에 대응하는 타원곡선 공개키인 `R = r * G` 그리고 공유된 비밀키 <code>S = P<sub>x</sub></code>를 연산 합니다.
여기서 <code>(P<sub>x</sub>, P<sub>y</sub>) = r * K<sub>B</sub></code> 입니다.
Alice는 initialization vector `iv`뿐만 아니라 <code>k<sub>E</sub> || k<sub>M</sub> = KDF(S, 32)</code>에서 암호화와 인증을 위해 key meterial을 유도합니다.
Alice는 Bobs에게 암호화된 메시지인 `R || iv || c || d`을 보냅니다.
여기서 <code>c = AES(k<sub>E</sub>, iv , m)</code>, <code>d = MAC(sha256(k<sub>M</sub>), iv || c)</code> 입니다.

For Bob to decrypt the message `R || iv || c || d`, he derives the shared secret
<code>S = P<sub>x</sub></code> where
<code>(P<sub>x</sub>, P<sub>y</sub>) = k<sub>B</sub> * R</code> as well as the encryption and
authentication keys <code>k<sub>E</sub> || k<sub>M</sub> = KDF(S, 32)</code>. Bob verifies
the authenticity of the message by checking whether
<code>d == MAC(sha256(k<sub>M</sub>), iv || c)</code> then obtains the plaintext as
<code>m = AES(k<sub>E</sub>, iv || c)</code>.

밥에게 복호화된 메시지인 `R || iv || c || d`을 보내기 위해, 그는 공유된 비밀키인 <code>S = P<sub>x</sub></code>을 유도합니다.
여기서 <code>(P<sub>x</sub>, P<sub>y</sub>) = k<sub>B</sub> * R</code>이며 암호화와 인증키인 <code>k<sub>E</sub> || k<sub>M</sub> = KDF(S, 32)</code>도 유도합니다.
밥은 <code>d == MAC(sha256(k<sub>M</sub>), iv || c)</code>가 유효한지 확인함으로써 메시지의 신뢰성을 확인합니다.
그리고 <code>m = AES(k<sub>E</sub>, iv || c)</code>로 plaintext를 얻습니다.

## Node Identity

All cryptographic operations are based on the secp256k1 elliptic curve. Each node is
expected to maintain a static secp256k1 private key which is saved and restored between
sessions. It is recommended that the private key can only be reset manually, for example,
by deleting a file or database entry.

모든 암호학 연산은 secp256k1 타원곡선을 기반으로 합니다. 각 노드는 전역 secp256k1 개인키를 유지해야 합니다. 이 개인키는 세션간에 저장되고 복원됩니다.
이는 개인키가 수동으로만 재설정될 수 있도록 권장합니다. 예를들어 파일이나 데이터베이스에서 수동으로 삭제하는식으로 말이죠.

## Initial Handshake

An RLPx connection is established by creating a TCP connection and agreeing on ephemeral
key material for further encrypted and authenticated communication. 
The process of creating those session keys is the 'handshake' and is carried out between the 'initiator'
(the node which opened the TCP connection) and the 'recipient' (the node which accepted it).

RLPx 연결은 TCP 연결이 생성되고 추가 암호화 및 인증 통신을 위해 임시의 키 구성요소(key material)에 합의함으로써 수립됩니다.
이러한 세션 키(those session keys)를 생성하는 과정은 'handshake'라고 표현하고 TCP 연결을 시작하는 'initiator'와 연결을 수락하는 'recipient'간에 수행됩니다.

1. initiator는 수신자에게 연결하고 `auth` 메시지를 보냅니다.
2. 수신자가 수락하면, 복호화하고 `auth`를 검증합니다(검증 방법은 signature ==`keccak256(ephemeral-pubk)`인지 확인합니다.)
3. 수신자는 `remote-ephemeral-pubk` 와 `nonce`로 부터 `auth-ack` 메시지를 생성합니다.
4. 수신자는 비밀키를 얻고 첫번째 암호화 frame을 보냅니다. 이 frame은 [Hello] 메시지를 포함합니다.
5. initiator는 `auth-ack`을 받고 비밀키를 얻습니다.
6. initiator는 첫번째 암호화 frame을 보냅니다. 이 frame은 initiator [Hello] 메시지를 포함합니다.
7. 수신자는 첫번째 암호화 프래임을 수신받고 인증합니다.
8. initiator는 첫번째 암호화 프래임을 수신받고 인증합니다.
9. 만약 첫번째 암호화 frame의 MAC이 양방향에서 유효하면 cryptographic handshake가 완료됩니다.

만약 첫번째 frame packet 인증이 실패하면 양쪽 연결이 끊길수도 있습니다.

Handshake messages 형식은 다음과 같습니다.

    auth = auth-size || enc-auth-body
    auth-size = size of enc-auth-body, encoded as a big-endian 16-bit integer
    auth-vsn = 4
    auth-body = [sig, initiator-pubk, initiator-nonce, auth-vsn, ...]
    enc-auth-body = ecies.encrypt(recipient-pubk, auth-body || auth-padding, auth-size)
    auth-padding = arbitrary data

    ack = ack-size || enc-ack-body
    ack-size = size of enc-ack-body, encoded as a big-endian 16-bit integer
    ack-vsn = 4
    ack-body = [recipient-ephemeral-pubk, recipient-nonce, ack-vsn, ...]
    enc-ack-body = ecies.encrypt(initiator-pubk, ack-body || ack-padding, ack-size)
    ack-padding = arbitrary data

구현에서 `auth-vsn` 과 `ack-vsn` 일치하지 않는 부분은 반드시 무시해야 합니다.
또한 `auth-body` and `ack-body` 에 추가적인 리스트 요소(additional list elements)가 있어도 무시해야 합니다.

비밀키는 다음처럼 handshake 메시지 교환 후에 생성됩니다:

    static-shared-secret = ecdh.agree(privkey, remote-pubk)
    ephemeral-key = ecdh.agree(ephemeral-privkey, remote-ephemeral-pubk)
    shared-secret = keccak256(ephemeral-key || keccak256(nonce || initiator-nonce))
    aes-secret = keccak256(ephemeral-key || shared-secret)
    mac-secret = keccak256(ephemeral-key || aes-secret)

## Framing

All messages following the initial handshake are framed. A frame carries a single
encrypted message belonging to a capability.

초기 핸드쉐이크에서 모든 메시지는 프레임화 됩니다. 하나의 프레임은 단일 암호화 메시지를 가지며 이 메시지는 capability에 속합니다.

The purpose of framing is multiplexing multiple capabilities over a single connection.
Secondarily, as framed messages yield reasonable demarcation points for message
authentication codes, supporting an encrypted and authenticated stream becomes
straight-forward. Frames are encrypted and authenticated via key material generated during
the handshake.

프레임화의 목적은 단일 연결위에서 여러 기능을 다중화(multiplexing multiple capabilities)하는 것입니다.
두번째로 프레임 메시지는 매시지 인증코드의 타당한 경계점(demarcation points)를 생성하면서 암호화되고 인중된 스트림을 지원하는 것이 간단해집니다.
프레임은 핸드쉐이크가 지속될 동안 생성된 key metrial을 통해 암호화되고 인증됩니다.

The frame header provides information about the size of the message and the message's
source capability. Padding is used to prevent buffer starvation, such that frame
components are byte-aligned to block size of cipher.

프레임 헤더는 메시지의 정보와 메시지 소스의 cabaibility에 대한 정보를 제공합니다.
프레임 구성요소가 암호의 블록사이즈 크기에 맞춰 바이트 정렬이 되는것 처럼 버퍼의 고갈을 방지하기 위해 패딩이 사용됩니다.
다음은 프레임 헤더의 구조입니다:

    frame = header-ciphertext || header-mac || frame-ciphertext || frame-mac
    header-ciphertext = aes(aes-secret, header)
    header = frame-size || header-data || header-padding
    header-data = [capability-id, context-id]
    capability-id = integer, always zero
    context-id = integer, always zero
    header-padding = zero-fill header to 16-byte boundary
    frame-ciphertext = aes(aes-secret, frame-data || frame-padding)
    frame-padding = zero-fill frame-data to 16-byte boundary

`frame-data` 과 `frame-size.`에 대한 정의를 참고하고 싶으시면 [Capability Messaging] 섹션을 참고하세요.

### MAC

Message authentication in RLPx uses two keccak256 states, one for each direction of
communication. The `egress-mac` and `ingress-mac` keccak states are continuously updated
with the ciphertext of bytes sent (egress) or received (ingress). Following the initial
handshake, the MAC states are initialized as follows:

RLPx에서 메시지 인증은 두개의 keccak256 states를 사용합니다. 통신의 각 방향에 대해 하나씩입니다.
`egress-mac` 와 `ingress-mac` keccak states는 전송된 바이트의 암호화된 텍스트(egress)와 수신된 바이트(ingress)로 지속적으로 업데이트 됩니다.
초기 핸드쉐이크에따라 MAC states는 다음과 같이 초기화됩니다:

Initiator(초기 발신자):

    egress-mac = keccak256.init((mac-secret ^ recipient-nonce) || auth)
    ingress-mac = keccak256.init((mac-secret ^ initiator-nonce) || ack)

Recipient(수신자):

    egress-mac = keccak256.init((mac-secret ^ initiator-nonce) || ack)
    ingress-mac = keccak256.init((mac-secret ^ recipient-nonce) || auth)

When a frame is sent, the corresponding MAC values are computed by updating the
`egress-mac` state with the data to be sent. The update is performed by XORing the header
with the encrypted output of its corresponding MAC. This is done to ensure uniform
operations are performed for both plaintext MAC and ciphertext. All MACs are sent
cleartext.

프레임이 전송되면 대응되는 MAC 값은 전송할 데이터와 같이 `egress-mac` state를 업데이트 합니다.
MAC에 대응되는 헤더를 암호화하여 얻은 결과값을 XOR연산하여 업데이트 합니다.
이는 평문 MAC과 암호화된 텍스트 양쪽에 대해 동일한 연산을 보장기위해 실행됩니다.
모든 MAC은 평문으로 전송됩니다.

    header-mac-seed = aes(mac-secret, keccak256.digest(egress-mac)[:16]) ^ header-ciphertext
    egress-mac = keccak256.update(egress-mac, header-mac-seed)
    header-mac = keccak256.digest(egress-mac)[:16]

Computing `frame-mac`:

    egress-mac = keccak256.update(egress-mac, frame-ciphertext)
    frame-mac-seed = aes(mac-secret, keccak256.digest(egress-mac)[:16]) ^ keccak256.digest(egress-mac)[:16]
    egress-mac = keccak256.update(egress-mac, frame-mac-seed)
    frame-mac = keccak256.digest(egress-mac)[:16]

Verifying the MAC on ingress frames is done by updating the `ingress-mac` state in the
same way as `egress-mac` and comparing to the values of `header-mac` and `frame-mac` in
the ingress frame. This should be done before decrypting `header-ciphertext` and
`frame-ciphertext`.

# Capability Messaging

All messages following the initial handshake are associated with a 'capability'. Any
number of capabilities can be used concurrently on a single RLPx connection.

A capability is identified by a short ASCII name (max eight characters) and version number. The capabilities
supported on either side of the connection are exchanged in the [Hello] message belonging
to the 'p2p' capability which is required to be available on all connections.

## Message Encoding

The initial [Hello] message is encoded as follows:

    frame-data = msg-id || msg-data
    frame-size = length of frame-data, encoded as a 24bit big-endian integer

where `msg-id` is an RLP-encoded integer identifying the message and `msg-data` is an RLP
list containing the message data.

All messages following Hello are compressed using the Snappy algorithm.

    frame-data = msg-id || snappyCompress(msg-data)
    frame-size = length of frame-data encoded as a 24bit big-endian integer

Note that the `frame-size` of compressed messages refers to the compressed size of
`msg-data`. Since compressed messages may inflate to a very large size after
decompression, implementations should check for the uncompressed size of the data before
decoding the message. This is possible because the [snappy format] contains a length
header. Messages carrying uncompressed data larger than 16 MiB should be rejected by
closing the connection.

## Message ID-based Multiplexing

While the framing layer supports a `capability-id`, the current version of RLPx doesn't
use that field for multiplexing between different capabilities. Instead, multiplexing
relies purely on the message ID.

Each capability is given as much of the message-ID space as it needs. All such
capabilities must statically specify how many message IDs they require. On connection and
reception of the [Hello] message, both peers have equivalent information about what
capabilities they share (including versions) and are able to form consensus over the
composition of message ID space.

Message IDs are assumed to be compact from ID 0x10 onwards (0x00-0x0f is reserved for the
"p2p" capability) and given to each shared (equal-version, equal-name) capability in
alphabetic order. Capability names are case-sensitive. Capabilities which are not shared
are ignored. If multiple versions are shared of the same (equal name) capability, the
numerically highest wins, others are ignored.

## "p2p" Capability

The "p2p" capability is present on all connections. After the initial handshake, both
sides of the connection must send either [Hello] or a [Disconnect] message. Upon receiving
the [Hello] message a session is active and any other message may be sent. Implementations
must ignore any difference in protocol version for forward-compatibility reasons. When
communicating with a peer of lower version, implementations should try to mimic that
version.

At any time after protocol negotiation, a [Disconnect] message may be sent.

### Hello (0x00)

`[protocolVersion: P, clientId: B, capabilities, listenPort: P, nodeKey: B_64, ...]`

First packet sent over the connection, and sent once by both sides. No other messages may
be sent until a Hello is received. Implementations must ignore any additional list elements
in Hello because they may be used by a future version.

- `protocolVersion` the version of the "p2p" capability, **5**.
- `clientId` Specifies the client software identity, as a human-readable string (e.g.
  "Ethereum(++)/1.0.0").
- `capabilities` is the list of supported capabilities and their versions:
  `[[cap1, capVersion1], [cap2, capVersion2], ...]`.
- `listenPort` (legacy) specifies the port that the client is listening on (on the
  interface that the present connection traverses). If 0 it indicates the client is
  not listening. This field should be ignored.
- `nodeId` is the secp256k1 public key corresponding to the node's private key.

### Disconnect (0x01)

`[reason: P]`

Inform the peer that a disconnection is imminent; if received, a peer should disconnect
immediately. When sending, well-behaved hosts give their peers a fighting chance (read:
wait 2 seconds) to disconnect to before disconnecting themselves.

`reason` is an optional integer specifying one of a number of reasons for disconnect:

| Reason | Meaning                                                      |
|--------|:-------------------------------------------------------------|
| `0x00` | Disconnect requested                                         |
| `0x01` | TCP sub-system error                                         |
| `0x02` | Breach of protocol, e.g. a malformed message, bad RLP, ...   |
| `0x03` | Useless peer                                                 |
| `0x04` | Too many peers                                               |
| `0x05` | Already connected                                            |
| `0x06` | Incompatible P2P protocol version                            |
| `0x07` | Null node identity received - this is automatically invalid  |
| `0x08` | Client quitting                                              |
| `0x09` | Unexpected identity in handshake                             |
| `0x0a` | Identity is the same as this node (i.e. connected to itself) |
| `0x0b` | Ping timeout                                                 |
| `0x10` | Some other reason specific to a subprotocol                  |

### Ping (0x02)

`[]`

Requests an immediate reply of [Pong] from the peer.

### Pong (0x03)

`[]`

Reply to the peer's [Ping] packet.

# Change Log

### Known Issues in the current version

- The frame encryption/MAC scheme is considered 'broken' because `aes-secret` and
  `mac-secret` are reused for both reading and writing. The two sides of a RLPx connection
  generate two CTR streams from the same key, nonce and IV. If an attacker knows one
  plaintext, they can decrypt unknown plaintexts of the reused keystream.
- General feedback from reviewers has been that the use of a keccak256 state as a MAC
  accumulator and the use of AES in the MAC algorithm is an uncommon and overly complex
  way to perform message authentication but can be considered safe.
- The frame encoding provides `capability-id` and `context-id` fields for multiplexing
  purposes, but these fields are unused.

### Version 5 (EIP-706, September 2017)

[EIP-706] added Snappy message compression.

### Version 4 (EIP-8, December 2015)

[EIP-8] changed the encoding of `auth-body` and `ack-body` in the initial handshake to
RLP, added a version number to the handshake and mandated that implementations should
ignore additional list elements in handshake messages and [Hello].

# References

- Elaine Barker, Don Johnson, and Miles Smid. NIST Special Publication 800-56A Section 5.8.1,
  Concatenation Key Derivation Function. 2017.\
  URL <https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-56ar.pdf>

- Victor Shoup. A proposal for an ISO standard for public key encryption, Version 2.1. 2001.\
  URL <http://www.shoup.net/papers/iso-2_1.pdf>

- Mike Belshe and Roberto Peon. SPDY Protocol - Draft 3. 2014.\
  URL <http://www.chromium.org/spdy/spdy-protocol/spdy-protocol-draft3>

- Snappy compressed format description. 2011.\
  URL <https://github.com/google/snappy/blob/master/format_description.txt>

Copyright &copy; 2014 Alex Leverington.
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">
This work is licensed under a
Creative Commons Attribution-NonCommercial-ShareAlike
4.0 International License</a>.

[Hello]: #hello-0x00
[Disconnect]: #disconnect-0x01
[Ping]: #ping-0x02
[Pong]: #pong-0x03
[Capability Messaging]: #capability-messaging
[EIP-8]: https://eips.ethereum.org/EIPS/eip-8
[EIP-706]: https://eips.ethereum.org/EIPS/eip-706
[RLP]: https://ethereum.org/en/developers/docs/data-structures-and-encoding/rlp
[snappy format]: https://github.com/google/snappy/blob/master/format_description.txt