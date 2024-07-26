<!--
# Ethereum Wire Protocol (ETH)

'eth' is a protocol on the [RLPx] transport that facilitates exchange of Ethereum
blockchain information between peers. The current protocol version is **eth/68**. See end
of document for a list of changes in past protocol versions.
-->
# Ethereum Wire Protocol (ETH)
'eth'는 [RLPx] 프로토콜 위에서 동작하는 프로토콜입니다. [RLPx] 프로토콜은 피어간 이더리움 블록체인 정보를 주고받는것을 돕습니다.
최근 프로토콜 버전은 **eth/68** 입니다. 이전 프로토콜 버전의 변경 사항은 본 문서 끝에 나와 있습니다.

<!--
### Basic Operation

Once a connection is established, a [Status] message must be sent. Following the reception
of the peer's Status message, the Ethereum session is active and any other message may be
sent.

Within a session, three high-level tasks can be performed: chain synchronization, block
propagation and transaction exchange. These tasks use disjoint sets of protocol messages
and clients typically perform them as concurrent activities on all peer connections.

Client implementations should enforce limits on protocol message sizes. The underlying
RLPx transport limits the size of a single message to 16.7 MiB. The practical limits for
the eth protocol are lower, typically 10 MiB. If a received message is larger than the
limit, the peer should be disconnected.

In addition to the hard limit on received messages, clients should also impose 'soft'
limits on the requests and responses which they send. The recommended soft limit varies
per message type. Limiting requests and responses ensures that concurrent activity, e.g.
block synchronization and transaction exchange work smoothly over the same peer
connection.
-->

### Basic Operation

연결이 시작되면, [Status] 메시지가 반드시 전송되어야 합니다. 다른 peer의 Status 메시지를 수신하고 나서, 이더리움 세션이 활성화 되고 다른 메시지를 전송할 수 있습니다.

세션이 활성화 되면, 세가지의 높은 수준의 작업들이 수행될 수 있습니다 : 체인의 동기화, 블록의 전파와 트래잭션 교환이 그것들 입니다.
이런 작업들은 서로 겹치지 않는 프로토콜 메시지의 집합을 사용하고 클라이언트들은 일반적으로 이 작업들을 모든 피어 연결에서 동시에 수행합니다.

클라이언트 구현은 프로토콜 메시지 사이즈에 대한 제한을 강제해야 합니다. 기본 RLPx wjsthddms eksdlf aptlwlfmf 16.7 MiB로 제한을 둡니다.
eth 프로토콜에서 실질적으로 제한은 더 낮습고 보통 10 MiB입니다. 만약 수신된 메시지가 제한크기보다 크다면 해당 메시지를 수신한 peer는 연결을 끊어야 합니다.

수신한 메시지에 좀 더 제한을 두기위해서, 클라이언트는 수신자와 발신자 사이 요청과 반응에서 'soft' 제한을 두어야 합니다.
권장되는 'soft' 제안은 메시지 타입마다 다양합니다. 요청과 응답을 제한하는것은 동시성을 보장합니다, 예를들어 블록 동기화와 트랜젝션 교환 작업은 같은 피어 연결에서 원활하게 작동합니다.


<!--
### Chain Synchronization
Nodes participating in the eth protocol are expected to have knowledge of the complete
chain of all blocks from the genesis block to current, latest block. The chain is obtained
by downloading it from other peers.

Upon connection, both peers send their [Status] message, which includes the Total
Difficulty (TD) and hash of their 'best' known block.

The client with the worst TD then proceeds to download block headers using the
[GetBlockHeaders] message. It verifies proof-of-work values in received headers and
fetches block bodies using the [GetBlockBodies] message. Received blocks are executed
using the Ethereum Virtual Machine, recreating the state tree and receipts.

Note that header downloads, block body downloads and block execution may happen
concurrently.
-->

### 체인 동기화
eth 프로토콜에 참여중인 노드들은 제네시스 블록부터 최근 블록까지의 모든 블록의 완전한 체인에 대한 정보를 알고 있어야(expect) 합니다.
해당 체인은 다른 피어들로부터 다운로드 받아서 얻을수 있습니다.

연결이 되고나서, 양측 피어들은 그들의 [Status] 메시지를 서로에게 보냅니다, 이 [Status] 메시지는 Total Difficulty(TD)와 'best'로 알려진 블록의 해시를 포함합니다.

최악의 TD를 가진 클라이언트라면 [GetBlockHeaders] 메시지를 사용하여 블록 헤더를 다운로드 합니다.[GetBlockHeaders] 메시지는 수신된 헤더의 작업증명값을 검증하고 [GetBlockBodies]를 사용하여 블록의 바디를 가져옵니다.
전달받은 블록들은 EVM(Ethereum Virtual Machine)을 사용하여 실행되고, 상태트리(status tree)와 receipts 재생성합니다.

헤더 다운로드,블록 바디 다운로드와 블록의 실행은 병렬적으로 발생할 수 있습니다.

<!--
### State Synchronization (a.k.a. "fast sync")

Protocol versions eth/63 through eth/66 also allowed synchronizing the state tree. Since
protocol version eth/67, the Ethereum state tree can no longer be retrieved using the eth
protocol, and state downloads are provided by the auxiliary [snap protocol] instead.

State synchronization typically proceeds by downloading the chain of block headers,
verifying their validity. Block bodies are requested as in the Chain Synchronization
section but transactions aren't executed, only their 'data validity' is verified. The
client picks a block near the head of the chain (the 'pivot block') and downloads the
state of that block.
-->

### 상태 동기화(State Synchronization) (a.k.a. "fast sync")
프로토콜 버전 eth/63 부터 eth/66까지 state tree를 동기화하는것을 허용합니다.
프로토콜 버전 eth/67 부터는, 이더리룸 state tree는 더이상 eth 프로토콜을 사용하여 검색할 수 없고, state 다운로드는 보조 [snap protocol]을 통해 제공됩니다.

상태 동기화(State Synchronization)는 전형적으로 블록헤더의 체인을 다운로드하고 그들의 유효성을 검증을 하면서 진행됩니다.
블록 바디는 체인 동기화 섹션 같은곳에서 요청되지만 트랜젝션은 실행되지 않고 오직 그들의 '데이터 유효성(data validity)'만 검증됩니다.
클라이언트는 'pibot block'이라고 불리는 체인의 헤드와 가까운 블록을 선택하고 선택한 블록의 상태를 다운로드 합니다.

<!--
### Block Propagation

**Note: after the PoW-to-PoS transition ([The Merge]), block propagation is no longer
handled by the 'eth' protocol. The text below only applies to PoW and PoA (clique)
networks. Block propagation messages (NewBlock, NewBlockHashes...) will be removed from
the protocol in a future version.**

Newly-mined blocks must be relayed to all nodes. This happens through block propagation,
which is a two step process. When a [NewBlock] announcement message is received from a
peer, the client first verifies the basic header validity of the block, checking whether
the proof-of-work value is valid. It then sends the block to a small fraction of connected
peers (usually the square root of the total number of peers) using the [NewBlock] message.

After the header validity check, the client imports the block into its local chain by
executing all transactions contained in the block, computing the block's 'post state'. The
block's `state-root` hash must match the computed post state root. Once the block is fully
processed, and considered valid, the client sends a [NewBlockHashes] message about the
block to all peers which it didn't notify earlier. Those peers may request the full block
later if they fail to receive it via [NewBlock] from anyone else.

A node should never send a block announcement back to a peer which previously announced
the same block. This is usually achieved by remembering a large set of block hashes
recently relayed to or from each peer.

The reception of a block announcement may also trigger chain synchronization if the block
is not the immediate successor of the client's current latest block.
-->
### 블록전파(Block Propagation)

**참고: PoW 에서 PoS로 전환하고 나서([The Merge]), 블록 전파는 더이상 'eth'프로토콜에서 처리되지 않습니다.
아래의 내용은 PoW와 PoA (clique) 네트워크에서만 적용됩니다. 블록 전파 메시지 (NewBlock, NewBlockHashes...)는 향후 버전에서 프로토콜에서 제거될 것입니다.**

새로 채굴된 블록들은 모든 노드들에게 전달되어져야 합니다.
이런 과정은 블록 전파를 통해서 일어납니다, 블록 전파는 2단계의 진행과정을 거칩니다.
[NewBlock] 알림 메시지가 피어로부터 수신받았을때, 해당 메시지를 받은 클라이언트는 먼저 블록의 basic header 유효성을 검증하고 작업증명값이 유효한지 확인합니다.
그런 다음에 블록을 연결된 일부 피어에게(small fraction of connected peer) (보통 전체 피어 수의 제곱근) 에 [NewBlock] 메시지를 사용하여 전송합니다.

[NewBlock] 메시지를 수신 받고 헤더 유효성을 체크한 후에, [NewBlock]으로 블록을 수신받은 클라이언트는 블록에 포함된 모든 트랜잭션을 실행해서 본인의 로컬체인에서 블록을 import합니다.
블록의 'state-root' 해쉬는 반드시 연산된 post state root와 일치해야 합니다. 
블록이 완전히 처리되고 나서, 유효하다고 판단되면 클라이언트는 [NewBlockHashes] 메시지를 아직 알리지 않은 모든 피어에게 보냅니다.
만약 다른 피어들이 다른 피어들로부터 [NewBlock]을 통해 블록을 받지 못했다면 나중에 전체 블록을 요청할 수 있습니다.

노드는 최근에 같은 블록을 알린 피어에게 블록알림을 보내지 않습니다. 이는 보통 각 피어로부터 최근에 전달받은 거대간 블록해시 집합을 기억할 수 있기 때문입니다.

블록 알림의 수신은 클라이언트의 최신 블록의 즉각적인 후속 블록이 아닐 경우에 체인 동기화의 트리거가 될 수 있습니다.

<!--
### Transaction Exchange

All nodes must exchange pending transactions in order to relay them to miners, which will
pick them for inclusion into the blockchain. Client implementations keep track of the set
of pending transactions in the 'transaction pool'. The pool is subject to client-specific
limits and can contain many (i.e. thousands of) transactions.

When a new peer connection is established, the transaction pools on both sides need to be
synchronized. Initially, both ends should send [NewPooledTransactionHashes] messages
containing all transaction hashes in the local pool to start the exchange.

On receipt of a NewPooledTransactionHashes announcement, the client filters the received
set, collecting transaction hashes which it doesn't yet have in its own local pool. It can
then request the transactions using the [GetPooledTransactions] message.

When new transactions appear in the client's pool, it should propagate them to the network
using the [Transactions] and [NewPooledTransactionHashes] messages. The Transactions
message relays complete transaction objects and is typically sent to a small, random
fraction of connected peers. All other peers receive a notification of the transaction
hash and can request the complete transaction object if it is unknown to them. The
dissemination of complete transactions to a fraction of peers usually ensures that all
nodes receive the transaction and won't need to request it.

A node should never send a transaction back to a peer that it can determine already knows
of it (either because it was previously sent or because it was informed from this peer
originally). This is usually achieved by remembering a set of transaction hashes recently
relayed by the peer.
-->
### 트랜젝션 교환(Transaction Exchange)
모든 노드는 반드시 보류중인 트랜잭션을 교환하여 채굴자들에게 중계해야 하고, 채굴자들은 블록체인에 포함시키기 위해 트랜잭션을 선택합니다.
클라이언트 구현은 'transaction pool'에서 보류중인 트랜젝션 집합을 계속 추적합니다. 이 트랜젝션 풀은 클라이언트 특정 제한(client-specific
limits)과 많은 트랜젝션(예를들어 수천개)을 포함할 수 있습니다.

새로운 피어와 연결이 되었을때, 양쪽의 트랜젝션 풀은 동기화되어야 합니다. 초기에, 양쪽 모두 [NewPooledTransactionHashes] 메시지를 보냅니다.
[NewPooledTransactionHashes]는 교환을 시작하기 위한 로컬풀에 있는 모든 트랜젝션 해쉬를 포함하고 있습니다.

[NewPooledTransactionHashes] 알림을 수신하면, 클라이언트는 수신된 집합(set)을 필터링합니다.
필터링은 본인의 로컬 풀에 아직 가지고 있지 않은 트랜젝션 해쉬들을 수집하는 것 입니다.
그런 다음에 [GetPooledTransactions] 메시지를 사용하여 트랜젝션을 요청할 수 있습니다.

새로운 트랜젝션이 클라언트 풀에서 나타나면, [Transactions] 과 [NewPooledTransactionHashes] 메시지를 사용하여 네트워크로 전파해야 합니다.
헤딩 트랜젝션 메시지는 완전한 트렌젝션 오브젝트를 전달하고 일반적으로 랜덤하게 연결된 일부 피어들에게 보냅니다. 모든 피어는 아니고 주변의 작은 수의 일부 피어들에게 전송합니다.
다른 모든 피어들은 트랜잭션 해쉬의 알림을 받고 만약 알림 받은것들이 알려져있지 않다면 완전한 트랜젝션을 요청합니다.
완전한 트랜젝션에 대한 정보를 일부 피어들에게 전파하는 것은 보통 모든 노드가 트랜젝션을 수신받았고 요청할 필요가 없다는것을 보장합니다.

이전에 보내졌거나, 원래 해당 피어에게 정보를 전달받았기 때문에 이미 알고있다고 판단한 피어에게 트랜잭션을 다시 보내지 않습니다.
이는 최근에 피어로부터 전달받은 트랜젝션 해쉬 집합을 기억함으로써 달성됩니다.
<!--
### Transaction Encoding and Validity

Transaction objects exchanged by peers have one of two encodings. In definitions across
this specification, we refer to transactions of either encoding using the identifier `txₙ`.

    tx = {legacy-tx, typed-tx}

Untyped, legacy transactions are given as an RLP list.

    legacy-tx = [
        nonce: P,
        gas-price: P,
        gas-limit: P,
        recipient: {B_0, B_20},
        value: P,
        data: B,
        V: P,
        R: P,
        S: P,
    ]

[EIP-2718] typed transactions are encoded as RLP byte arrays where the first byte is the
transaction type (`tx-type`) and the remaining bytes are opaque type-specific data.

    typed-tx = tx-type || tx-data

Transactions must be validated when they are received. Validity depends on the Ethereum
chain state. The specific kind of validity this specification is concerned with is not
whether the transaction can be executed successfully by the EVM, but only whether it is
acceptable for temporary storage in the local pool and for exchange with other peers.

Transactions are validated according to the rules below. While the encoding of typed
transactions is opaque, it is assumed that their `tx-data` provides values for `nonce`,
`gas-price`, `gas-limit`, and that the sender account of the transaction can be determined
from their signature.

- If the transaction is typed, the `tx-type` must be known to the implementation. Defined
  transaction types may be considered valid even before they become acceptable for
  inclusion in a block. Implementations should disconnect peers sending transactions of
  unknown type.
- The signature must be valid according to the signature schemes supported by the chain.
  For typed transactions, signature handling is defined by the EIP introducing the type.
  For legacy transactions, the two schemes in active use are the basic 'Homestead' scheme
  and the [EIP-155] scheme.
- The `gas-limit` must cover the 'intrinsic gas' of the transaction.
- The sender account of the transaction, which is derived from the signature, must have
  sufficient ether balance to cover the cost (`gas-limit * gas-price + value`) of the
  transaction.
- The `nonce` of the transaction must be equal or greater than the current nonce of the
  sender account.
- When considering the transaction for inclusion in the local pool, it is up to
  implementations to determine how many 'future' transactions with nonce greater than the
  current account nonce are valid, and to which degree 'nonce gaps' are acceptable.

Implementations may enforce other validation rules for transactions. For example, it is
common practice to reject encoded transactions larger than 128 kB.

Unless noted otherwise, implementations must not disconnect peers for sending invalid
transactions, and should simply discard them instead. This is because the peer might be
operating under slightly different validation rules.
-->
### 트랜젝션 인코딩과 유효성검증(Transaction Encoding and Validity)

피어들 사이에서 교환된 트랜젝션 오브잭트들은 두가지 타입중 하나의 인코딩을 가지고 있습니다.
명세서에 전반에 따른 정의에 의하면, 우리는 앞서 말한 두가지 타입의 인코딩중 `txₙ` 식별자를 사용하는 인코딩의 트랜젝션을 참조합니다.

    tx = {legacy-tx, typed-tx}

타입이 지정되지 않았거나 레거시 트랜젝션(legacy transaction)은 RLP 리스트로 표현됩니다.

    legacy-tx = [
        nonce: P,
        gas-price: P,
        gas-limit: P,
        recipient: {B_0, B_20},
        value: P,
        data: B,
        V: P,
        R: P,
        S: P,
    ]

[EIP-2718] 타입으로 지정된 트랜젝션은 첫번째 바이트 트랜젝션 유형이 (`tx-type`)이고 남은 바이트가 불투명한 타입별 데이터 일때 RLP 바이트 배열로 인코딩됩니다. 

    typed-tx = tx-type || tx-data


트랜젝션은 반드시 수신받았을때 유효성을 검증해야 합니다. 유효성 검사는 이더리움 체인 상태에 의존합니다.
이 명세에서 이런 특수한 유형의 유효성 검증은 로컬풀의 임시 스토리지와 다른 피어들과의 교환을 위해 수용 가능한지에 대한 것이지, 
트랜젝션이 EVM에 의해서 성공적으로 실행 될 수 있는지에 대한 것이 아닙니다.

트랜젝션은 아래의 규칙에 따라 유효성을 검증합니다. 타입이 지정된 트랜젝션의 인코딩 방법은 불투명한 반면에 
트랜젝션의 `tx-data`가 `nonsce`,`gas-price`,`gas-limit`의 값을 제공하고 있고, 트랙젝션의 송금인의 계좌가 트랜젝션 서명을 통해서 결정 될 수 있다고 가정합니다.

- 만약 트랜젝션 타입이 지정되어 있다면, `tx-type`은 반드시 구현할때 알려져 있어야 합니다.
  정의된 트랜젝션 타입은 블록에 포함될 수 있다고 판단되기 전에라도 유효하다고 간주될(may) 수 있습니다.
- 서명은 체인에서 지원되는 서명방식에 따라 검증되어야 합니다. 타입이 지정된 트랜젝션의 경우, 서명을 다루는 방식은 도입된 EIP 타입에 의해서 정의되어져야 합니다.
  레거시 트랜젝션의 경우, 활발하게 사용되는 두가지 방식이 있는데 basic `Homestead` 방식과 [EIP-155] 방식입니다.
- `gas-limit`은 트랜젝션의 'intrinsic gas'를 커버해야 합니다.
- 서명으로부터 파생된 트랜젝션의 송금인의 계좌는 트랜젝션 발생에 필요한 비용(`gas-limit * gas-price + value`)만큼의 충분한 이더리움 잔액이 있어야 합니다.
- 트랜젝션의 `nonce`는 송금인의 계좌의 최근 nonce보다 크거나 같아야 합니다.
- 트랜젝션이 로컬 풀에 포함하는것을 고려할때, 현재 계좌의 논스보다 큰 미래에 발생할 트랜잭션 논스가 얼마나 유효한지, 
  그리고 얼마 정도의 `nonce gaps`가 수용가능한지 결정하는것은 어떻게 구현하느냐에 따라 다릅니다.

구현하는것은 트랜젝션에 다른 방식의 유효성 검증 규칙을 강제할 수도 있습니다.
예를 들어 트랜젝션이 128kb보다 큰 사이즈로 인코딩 되는 것을 거부하는것이 일반적인 방식입니다. 

Unless noted otherwise, implementations must not disconnect peers for sending invalid
transactions, and should simply discard them instead. This is because the peer might be
operating under slightly different validation rules.

별도로 언급되지 않는 한, 구현할때 반드시 유효하지 않은 트랜젝션을 보내기 위해 피어와 연결을 끊어서는 안되고 대신에 그것들을 단순히 폐기해야 합니다.

<!--
### Block Encoding and Validity

Ethereum blocks are encoded as follows:

    block = [header, transactions, ommers]
    transactions = [tx₁, tx₂, ...]
    ommers = [header₁, header₂, ...]
    withdrawals = [withdrawal₁, withdrawal₂, ...]
    header = [
        parent-hash: B_32,
        ommers-hash: B_32,
        coinbase: B_20,
        state-root: B_32,
        txs-root: B_32,
        receipts-root: B_32,
        bloom: B_256,
        difficulty: P,
        number: P,
        gas-limit: P,
        gas-used: P,
        time: P,
        extradata: B,
        mix-digest: B_32,
        block-nonce: B_8,
        basefee-per-gas: P,
        withdrawals-root: B_32,
    ]

In certain protocol messages, the transaction and ommer lists are relayed together as a
single item called the 'block body'.

    block-body = [transactions, ommers, withdrawals]

The validity of block headers depends on the context in which they are used. For a single
block header, only the validity of the proof-of-work seal (`mix-digest`, `block-nonce`)
can be verified. When a header is used to extend the client's local chain, or multiple
headers are processed in sequence during chain synchronization, the following rules apply:

- Headers must form a chain where block numbers are consecutive and the `parent-hash` of
  each header matches the hash of the preceding header.
- When extending the locally-stored chain, implementations must also verify that the
  values of `difficulty`, `gas-limit` and `time` are within the bounds of protocol rules
  given in the [Yellow Paper].
- The `gas-used` header field must be less than or equal to the `gas-limit`.
- `basefee-per-gas` must be present in headers after the [London hard fork]. It must be
  absent for earlier blocks. This rule was added by [EIP-1559].
- For PoS blocks after [The Merge], `ommers-hash` must be the empty keccak256 hash since
  no ommer headers can exist.
- `withdrawals-root` must be present in headers after the [Shanghai fork]. The field must
  be absent for blocks before the fork. This rule was added by [EIP-4895].

For complete blocks, we distinguish between the validity of the block's EVM state
transition, and the (weaker) 'data validity' of the block. The definition of state
transition rules is not dealt with in this specification. We require data validity of the
block for the purposes of immediate [block propagation] and during [state synchronization].

To determine the data validity of a block, use the rules below. Implementations should
disconnect peers sending invalid blocks.

- The block `header` must be valid.
- The `transactions` contained in the block must be valid for inclusion into the chain at
  the block's number. This means that, in addition to the transaction validation rules
  given earlier, validating whether the `tx-type` is permitted at the block number is
  required, and validation of transaction gas must take the block number into account.
- The sum of the `gas-limit`s of all transactions must not exceed the `gas-limit` of the
  block.
- The `transactions` of the block must be verified against the `txs-root` by computing and
  comparing the merkle trie hash of the transactions list.
- The `withdrawals` of block body must be verified against the `withdrawals-root` by
  computing and comparing the merkle trie hash of the withdrawals list. Note that no
  further validation of withdrawals is possible, since they are injected into the block by
  the consensus layer.
- The `ommers` list may contain at most two headers.
- `keccak256(ommers)` must match the `ommers-hash` of the block header.
- The headers contained in the `ommers` list must be valid headers. Their block number
  must not be greater than that of the block they are included in. The parent hash of an
  ommer header must refer to an ancestor of depth 7 or less of its including block, and it
  must not have been included in any earlier block contained in this ancestor set.

### Receipt Encoding and Validity

Receipts are the output of the EVM state transition of a block. Like transactions,
receipts have two distinct encodings and we will refer to either encoding using the
identifier `receiptₙ`.

    receipt = {legacy-receipt, typed-receipt}

Untyped, legacy receipts are encoded as follows:

    legacy-receipt = [
        post-state-or-status: {B_32, {0, 1}},
        cumulative-gas: P,
        bloom: B_256,
        logs: [log₁, log₂, ...]
    ]
    log = [
        contract-address: B_20,
        topics: [topic₁: B, topic₂: B, ...],
        data: B
    ]

[EIP-2718] typed receipts are encoded as RLP byte arrays where the first byte gives the
receipt type (matching `tx-type`) and the remaining bytes are opaque data specific to the
type.

    typed-receipt = tx-type || receipt-data

In the Ethereum Wire Protocol, receipts are always transferred as the complete list of all
receipts contained in a block. It is also assumed that the block containing the receipts
is valid and known. When a list of block receipts is received by a peer, it must be
verified by computing and comparing the merkle trie hash of the list against the
`receipts-root` of the block. Since the valid list of receipts is determined by the EVM
state transition, it is not necessary to define any further validity rules for receipts in
this specification.
-->

### 블록 인코딩과 유효성 검사(Block Encoding and Validity)

이더리움 블록은 다음과 같이 인코딩 됩니다.

    block = [header, transactions, ommers]
    transactions = [tx₁, tx₂, ...]
    ommers = [header₁, header₂, ...]
    withdrawals = [withdrawal₁, withdrawal₂, ...]
    header = [
        parent-hash: B_32,
        ommers-hash: B_32,
        coinbase: B_20,
        state-root: B_32,
        txs-root: B_32,
        receipts-root: B_32,
        bloom: B_256,
        difficulty: P,
        number: P,
        gas-limit: P,
        gas-used: P,
        time: P,
        extradata: B,
        mix-digest: B_32,
        block-nonce: B_8,
        basefee-per-gas: P,
        withdrawals-root: B_32,
    ]

특정 프로토콜 메시지에서 트란젝션과 ommer 리스트는 'block body'라고 불리는 단일 항목으로 같이 전달됩니다. 

    block-body = [transactions, ommers, withdrawals]

블록 헤더의 유효성은 사용되는 컨텍스트에 따라 다릅니다. 단일 블록헤더의 경우, 작업증명 서명 (`mix-digest`, `block-nonce`)의 유효성만 증명 될 수 있습니다.
헤더가 클라이언트의 로컬체인을 확장하기 위해 사용될때, 또는 다수의 헤더가 체인이 동기화 중에 순차적으로 처리될때, 다음의 규칙들을 따릅니다.

- 헤더는 반드시 블록번호가 연속적이고 각각의 헤더의 `parent-hash`가 이전 헤더의 해시와 일치한 체인에서 형성되어야 합니다.
- 만약 로컬에 저장된 체인을 확장할때에는, 반드시 `difficulty`, `gas-limit` 그리고 `time`의 값들이 [Yellow Paper]에서 정한 프로토콜 규칙 범위 내에 있는지 검증해야 합니다.
- `gas-used` 헤더 필드는 반드시 `gas-limit`보다 작거나 같아야 합니다.
- `basefee-per-gas`는 반드시 [London hard fork]이후 헤더에 명시되어야 합니다. 이전 블록에 대해서는 반드시 제외해야 합니다. 이 규칙은 [EIP-1559]에 의해 추가되었습니다.
- [The Merge]이후 PoS 블록의 경우, ommer header들이 존재하지 않기 때문에 `ommers-hash`는 반드시 빈 keccak256 해시여야 합니다.
- `withdrawals-root`는 반드시 [Shanghai fork]이후에는 헤더에 반드시 명시되어야 합니다. 이 필드는 [Shangai fork]이전 블록들에서는 반드시 제외되어야 합니다. 이 규칙은 [EIP-4895]에 의해 추가되었습니다.

완전한 블록의 경우, 블록의 EVM 상태전이에 대한 유효성과, 블록의 (waeker) `data-validty`에 대해 구분해야 합니다.
상태전이 규칙에 대한 정의는 본 명세에서 다루지는 않습니다. 우리는 즉각적인 [block propagation]을 목적으로 그리고 [state synchroniztion]중에 블록의 유효성 데이터를 요청합니다.

블록의 유효성 데이터를 결정짓는건, 아래의 규칙을 따릅니다. 유효하지 않은 블록을 보내는 피어들과 연결을 끊도록 구형해야합니다.

- 블록 `header`는 반드시 유효해야 합니다.
- 블록에 포함되어 있는 `transaction`은 블록의 번호에서 체인에 포함되도록 유효해야 합니다.
  이는 이전에 주어진 트랜젝션 유효성 검사 규칙에 추가적으로, `tx-type`이  블록 숫자에서 허용되는지를 요구하는지를 검증하는것과 트랜젝션 가스의 유효성은 반드시 

## Protocol Messages
<!--
In most messages, the first element of the message data list is the `request-id`. For
requests, this is a 64-bit integer value chosen by the requesting peer. The responding
peer must mirror the value in the `request-id` element of the response message.
-->

대부분의 메시지에서, 메시지 데이터 리스트의 첫번쨰 요소는 `request-id` 입니다.
요청의 경우 요청 peer에서 선택한 64-bit의 정수값입니다. 응답하는 peer는 반드시 응답메시지 요소에서 `request-id` 값을 반드시 반영해야 합니다.

### Status (0x00)
<!--
`[version: P, networkid: P, td: P, blockhash: B_32, genesis: B_32, forkid]`

Inform a peer of its current state. This message should be sent just after the connection
is established and prior to any other eth protocol messages.

- `version`: the current protocol version
- `networkid`: integer identifying the blockchain, see table below
- `td`: total difficulty of the best chain. Integer, as found in block header.
- `blockhash`: the hash of the best (i.e. highest TD) known block
- `genesis`: the hash of the genesis block
- `forkid`: An [EIP-2124] fork identifier, encoded as `[fork-hash, fork-next]`.

This table lists common Network IDs and their corresponding networks. Other IDs exist
which aren't listed, i.e. clients should not require that any particular network ID is
used. Note that the Network ID may or may not correspond with the EIP-155 Chain ID used
for transaction replay prevention.

| ID | chain                     |
|----|---------------------------|
| 0  | Olympic (disused)         |
| 1  | Frontier (now mainnet)    |
| 2  | Morden testnet (disused)  |
| 3  | Ropsten testnet (disused) |
| 4  | Rinkeby testnet (disused) |
| 5  | Goerli testnet            |

For a community curated list of chain IDs, see <https://chainid.network>.
-->

`[version: P, networkid: P, td: P, blockhash: B_32, genesis: B_32, forkid]`

피어의 최근 상태를 알려줍니다. 위의 메시지는 연결이 성립된 직후에 보내지고 어떤 eth protocol 메시지보다도 우선시 해야합니다.

- `version`: the current protocol version
- `networkid`: integer identifying the blockchain, see table below
- `td`: total difficulty of the best chain. Integer, as found in block header.
- `blockhash`: the hash of the best (i.e. highest TD) known block
- `genesis`: the hash of the genesis block
- `forkid`: An [EIP-2124] fork identifier, encoded as `[fork-hash, fork-next]`.

- `version`: 최근 프로토콜의 버전을 나타냅니다.
- `networkid`: 정수로 식별된 블록체인입니다, 아래의 테이블을 참조하세요.
- `td`: 체인의 최고 난이도를 의미합니다. 블록 헤더에서 찾을 수 있는 정수입니다.
- `blockhash`:  현재까지 알려진 블록 중에서 총 난이도가 높은 블록입니다.
- `genesis`: 제네시스 블록의 해시입니다.
- `forkid`: [EIP-2124] fork 식별자입니다, `[fork-hash, fork-next]`로 인코딩됩니다.

This table lists common Network IDs and their corresponding networks. Other IDs exist
which aren't listed, i.e. clients should not require that any particular network ID is
used. Note that the Network ID may or may not correspond with the EIP-155 Chain ID used
for transaction replay prevention.

아래의 테이블은 일반적인 네트워크 아이디와 아이디에 대응하는 네트워크를 나열한 테이블입니다.
테이블에 없는 ID들은, 예를들어 클라이언트가 특정 네트워크 ID를 특정하여 사용하지 않는 ID들 입니다.
네트워크 ID는 EIP-155 Chain ID와 일치할 수도 있고 그렇지 않을 수도 있습니다.

| ID | chain                     |
|----|---------------------------|
| 0  | Olympic (disused)         |
| 1  | Frontier (now mainnet)    |
| 2  | Morden testnet (disused)  |
| 3  | Ropsten testnet (disused) |
| 4  | Rinkeby testnet (disused) |
| 5  | Goerli testnet            |

커뮤니티가 관리하는 체인 ID 목록은 <https://chainid.network>를 참조하세요.

<!--
### NewBlockHashes (0x01)

`[[blockhash₁: B_32, number₁: P], [blockhash₂: B_32, number₂: P], ...]`

Specify one or more new blocks which have appeared on the network. To be maximally
helpful, nodes should inform peers of all blocks that they may not be aware of. 
Including hashes that the sending peer could reasonably be considered to know 
(due to the fact they were previously informed of because that node has itself advertised knowledge of the 
hashes through NewBlockHashes) 
is considered bad form, and may reduce the reputation of
the sending node. Including hashes that the sending node later refuses to honour with a
proceeding [GetBlockHeaders] message is considered bad form, and may reduce the reputation
of the sending node.
-->

### NewBlockHashes (0x01)

`[[blockhash₁: B_32, number₁: P], [blockhash₂: B_32, number₂: P], ...]`

네트워크에 나타난 하나 이상의 새로운 블록을 지정합니다. 최대한 도움이 되기 위해서, 노드들은 피어들에게 알려지지 않은 모든 블록들을 알려주어야 합니다.
타당한 이유로 발신하는 피어가 이미 알고 있다고 간주될 수 있는 해쉬를 포함하는 것은
(노드가 NewBlockHashes 통하셔 정보를 알고 있다고 알려진 이유로 피어들이 사전에 통보 받았기 때문에)
좋지 않은 방법이며 발신한 노드의 평판을 저하시킬 수 있습니다.
발신하는 노드가 이후 [GetBlockHeaders] 메시지를 통해 제공하기를 거부하는 해쉬들을 포함하는 것도 좋지 않은 방법이며 발신하는 노드의 평판을 저하시킬 수 있습니다.


<!--
### Transactions (0x02)

`[tx₁, tx₂, ...]`

Specify transactions that the peer should make sure is included on its transaction queue.
The items in the list are transactions in the format described in the main Ethereum
specification. Transactions messages must contain at least one (new) transaction, empty
Transactions messages are discouraged and may lead to disconnection.

Nodes must not resend the same transaction to a peer in the same session and must not
relay transactions to a peer they received that transaction from. In practice this is
often implemented by keeping a per-peer bloom filter or set of transaction hashes which
have already been sent or received.
-->

### Transactions (0x02)

`[tx₁, tx₂, ...]`

피어가 자신의 트랜젝션 큐에 반드시 포함해야할 하는 트랜잭션을 지정합니다.
큐의 항목은 주요 이더리움 명세 설명된 형식의 트랜젝션입니다. 트랜젝션 메시지는 반드시 최소한 하나의 (새로운) 트랜젝션을 포함해야 합니다.
빈 트랜젝션 메시지는 권장되지 않으며 연결을 끊을 수 있습니다.

노드들은 반드시 같은 트랜젝션을 같은 세션에 있는 피어에게 재전송해야 하고 그 트랜젝션을 받은 피어에게는 전송하지 말아야합니다.
이는 피어마다 bloom filter를 유지하거나 이미 전송되어졌거나 발신된 트랜잭션 해쉬들의 집합 을 유지하기위해 실제로 이런식으로 자주 구현됩니다.

<!--
### GetBlockHeaders (0x03)

`[request-id: P, [startblock: {P, B_32}, limit: P, skip: P, reverse: {0, 1}]]`

Require peer to return a BlockHeaders message. 
The response must contain a number of block headers, 
of rising number when `reverse` is `0`, 
falling when `1`, 
`skip` blocks apart,
beginning at block `startblock` (denoted by either number or hash) in the canonical chain,
and with at most `limit` items.
-->

### GetBlockHeaders (0x03)

`[request-id: P, [startblock: {P, B_32}, limit: P, skip: P, reverse: {0, 1}]]`

BlockHeaders 메시지를 반환하도록 피어에게 요청합니다.
응답은 다음 조건을 만족하는 여러 개의 블록헤더를 포함해야 합니다.
- `reverse: {0, 1}` : `0`일때는 오름차순, `1`일때는 내림차순을.
- `skip` : 블록을 건너뛰는것을 의미, 지정된 숫자 만큼 블록을 건너뛰라는 의미입니다.
- `startblock` : 정규 체인에서 블록 `startblock`에서 시작하는 블록 헤더를 나타냅니다.
- `limit` : 최대 `limit` 항목을 포햄해야 합니다.

즉 `skip`의 수 만큼 블록을 건너 뛰고, `startblock` 에서 명시된 시작할 블록을 선택하고, `limit` 만큼의 블록 헤더를
`reverse`에 명시된 대로 오름차순 혹은 내림차순으로 반환해야 합니다.

<!--
### BlockHeaders (0x04)

`[request-id: P, [header₁, header₂, ...]]`

This is the response to GetBlockHeaders, containing the requested headers. The header list
may be empty if none of the requested block headers were found. The number of headers that
can be requested in a single message may be subject to implementation-defined limits.

The recommended soft limit for BlockHeaders responses is 2 MiB.
-->

### BlockHeaders (0x04)

`[request-id: P, [header₁, header₂, ...]]`

GetBlockHeaders의 응답이며, 요청된 헤더를 포합합니다.
요청된 헤더 리스트는 요청된 블록헤더가 없는경우 비어있을 수 있습니다.
단일 메시지에서 요철될 수 있는 헤더의 수는 구현할때 정의된 제한에 따라 달라질 수 있습니다.

권장되는 BlockHeaders 응답의 크기의 soft limit은 2 MiB 입니다.

<!--
### GetBlockBodies (0x05)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

This message requests block body data by hash. The number of blocks that can be requested
in a single message may be subject to implementation-defined limits.
-->

### GetBlockBodies (0x05)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

이 메시지는 해시로 블록 바디 데이터를 요청합니다. 단일 메시지에서 요청될 수 있는 블록의 수는 구현할때 정의된 제한에 따라 달라질 수 있습니다.

<!--
### BlockBodies (0x06)

`[request-id: P, [block-body₁, block-body₂, ...]]`

This is the response to GetBlockBodies. The items in the list contain the body data of the
requested blocks. The list may be empty if none of the requested blocks were available.

The recommended soft limit for BlockBodies responses is 2 MiB.
-->

### BlockBodies (0x06)

`[request-id: P, [block-body₁, block-body₂, ...]]`

GetBlockBodies에 대한 응답입니다. 리스트의 요소들은 요청된 블록의 바디 데이터를 포함합니다.
요청된 블록들이 모두 가용성이 없을때 리스트는 비어있을 수 있습니다.

BlockBodies 응답의 권장되는 soft limit은 2 MiB 입니다.


<!--
### NewBlock (0x07)

`[block, td: P]`

Specify a single complete block that the peer should know about. `td` is the total
difficulty of the block, i.e. the sum of all block difficulties up to and including this
block.
-->

### NewBlock (0x07)

`[block, td: P]`

피어가 반드시 알고 있어야할 단일의 완전한 블록을 지정합니다.
`td`는 블록의 총 난이도를 의미하며, 예를들어 이 블록을 포함한 모등 블록의 난이도 입니다.

<!--
### NewPooledTransactionHashes (0x08)

`[txtypes: B, [txsize₁: P, txsize₂: P, ...], [txhash₁: B_32, txhash₂: B_32, ...]]`

This message announces one or more transactions that have appeared in the network and
which have not yet been included in a block. The message payload describes a list of of
transactions, but note that it is encoded as three separate elements.

The `txtypes` element is a byte array containing the announced [transaction types]. The
other two payload elements refer to the sizes and hashes of the announced transactions.
All three payload elements must contain an equal number of items.

`txsizeₙ` refers to the length of the 'consensus encoding' of a typed transaction, i.e.
the byte size of `tx-type || tx-data` for typed transactions, and the size of the
RLP-encoded `legacy-tx` for non-typed legacy transactions.

The recommended soft limit for this message is 4096 items (~150 KiB).

To be maximally helpful, nodes should inform peers of all transactions that they may not
be aware of. However, nodes should only announce hashes of transactions that the remote
peer could reasonably be considered not to know, but it is better to return more
transactions than to have a nonce gap in the pool.
-->

### NewPooledTransactionHashes (0x08)

`[txtypes: B, [txsize₁: P, txsize₂: P, ...], [txhash₁: B_32, txhash₂: B_32, ...]]`

위 예시의 메시지는 네트워크상에 나타난 하나 이상의 트랜잭션과 블록에 아직 포함되지 않은 트랜잭션을 알립니다.
메시지의 페이로드는 트랜잭션의 리스트에 대하여 설명하지만 세개의 별도 요소로 인코딩되어 있음을 주목하세요.

`txtypes` 요소는 [transaction types]를 알려주는 바이트 배열입니다.
다른 두개의 페이로드 요소는 알려진 트랜젝션의 해시값과 사이즈를 참조합니다.
3개 페이로드 요소 모두 동일한 수의 항목을 포함해야 합니다.

`txsizeₙ`는
타입이 지정된 트랜잭션의 `tx-type || tx-data`의 바이트 크기나
그리고 타입이 지정되지 않은 legacy-transaction dml RKP-encoded `legacy-tx`의 크기 처럼
트랜잭션 타입의 'consensus encoding' 길이를 참조합니다.

<!--
### GetPooledTransactions (0x09)

`[request-id: P, [txhash₁: B_32, txhash₂: B_32, ...]]`

This message requests transactions from the recipient's transaction pool by hash.
The recommended soft limit for GetPooledTransactions requests is 256 hashes (8 KiB). The
recipient may enforce an arbitrary limit on the response (size or serving time), which
must not be considered a protocol violation.
-->


### GetPooledTransactions (0x09)

`[request-id: P, [txhash₁: B_32, txhash₂: B_32, ...]]`

이 메시지는 해시 값으로 수신자의 트랜잭션 풀로의 트랜잭션을 요청합니다.
권장되는 GetPooledTransactions 요청의 soft limit은 256 해시 (8 KiB) 입니다.
수신자는 프로토콜 위반으로 간주되어서는 안되는 응답에 임의의 제한(가령 가령 크기나 서비스 시간에)을 강제할 수 있습니다.


<!--
### PooledTransactions (0x0a)

`[request-id: P, [tx₁, tx₂...]]`

This is the response to GetPooledTransactions, returning the requested transactions from
the local pool. The items in the list are transactions in the format described in the main
Ethereum specification.

The transactions must be in same order as in the request, but it is OK to skip
transactions which are not available. This way, if the response size limit is reached,
requesters will know which hashes to request again (everything starting from the last
returned transaction) and which to assume unavailable (all gaps before the last returned
transaction).

It is permissible to first announce a transaction via NewPooledTransactionHashes, but then
to refuse serving it via PooledTransactions. This situation can arise when the transaction
is included in a block (and removed from the pool) in between the announcement and the
request.

A peer may respond with an empty list if none of the hashes match transactions in its
pool.
-->

### PooledTransactions (0x0a)

`[request-id: P, [tx₁, tx₂...]]`

GetPooledTransactions에 대한 응답메시지입니다.
로컬풀의 트랜잭션 요청을 반환합니다.
위 예시에서 볼 수 있는 메시지 리스트의 항목들은 주요 이더리움 명세에서 설명된 형식의 트랜잭션입니다.

명세에 설명된 트랜잭션은 요청 순서와 같아야 하지만 사용할 수 없는 트랜잭션을 건너뛰는건 괜찮습니다.
건너 뛸 경우에, 만약 응답 사이즈가 제한에 도달하면, 요청자들은 어떤 해쉬들(마지막으로 반환됐던 트랜젝션 이후의 모든 트랜잭션들)이 다시 요청하는지
그리고 어떤 해쉬들이 사용할 수 없는지(마지막으로 반환된 트랜잭션 이전의 모든 갭들)을 알 수 있습니다.

피어는 만약 본인의 풀에 잇는 트랜잭션과 매치되는 해쉬가 없다면 빈 리스트로 응답할 수 있습니다.


<!--
### GetReceipts (0x0f)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

Require peer to return a Receipts message containing the receipts of the given block
hashes. The number of receipts that can be requested in a single message may be subject to
implementation-defined limits.
-->

### GetReceipts (0x0f)

`[request-id: P, [blockhash₁: B_32, blockhash₂: B_32, ...]]`

피어에게 주어진 블록해쉬에 대한 Receipts을 포함하여 Receipts message 반환하도록 요청합니다.
단일메시지에서 요청될 수 있는 Receipts의 수는 구현할때 정의된 제한에 따라 달라질 수 있습니다.


<!--
### Receipts (0x10)

`[request-id: P, [[receipt₁, receipt₂], ...]]`

This is the response to GetReceipts, providing the requested block receipts. Each element
in the response list corresponds to a block hash of the GetReceipts request, and must
contain the complete list of receipts of the block.

The recommended soft limit for Receipts responses is 2 MiB.
-->

### Receipts (0x10)

`[request-id: P, [[receipt₁, receipt₂], ...]]`

GetReceipts 요청에 대한 응답입니다.
요청된 블록 receipt을 제공합니다.
응답 할때 리스트의 각각의 요소는 GetReceipts를 요청받을때 받은 블록해쉬에 대응되는 요소들이며, 블록에 대한 완전한 receipt 리스트 를 포함해야 합니다.

권장되는 Receipts 응답의 soft limit은 2 MiB 입니다.


## Change Log


<!--
### eth/68 ([EIP-5793], October 2022)

Version 68 changed the [NewPooledTransactionHashes] message to include types and sizes of
the announced transactions. Prior to this update, the message payload was simply a list of
hashes: `[txhash₁: B_32, txhash₂: B_32, ...]`.
-->

### eth/68 ([EIP-5793], October 2022)

Version 68에서 [NewPooledTransactionHashes] 메시지를 알려진 트랜젝션의 사이즈와 타입을 포함하도록 변경했습니다.
업데이트에서 중요한 점은, 메시지의 페이소드는 단순히 해시의 리스트였습니다: `[txhash₁: B_32, txhash₂: B_32, ...]`.


<!--
### eth/67 with withdrawals ([EIP-4895], March 2022)

PoS validator withdrawals were added by [EIP-4895], which changed the definition of block
headers to include a `withdrawals-root`, and block bodies to include the `withdrawals`
list. No new wire protocol version was created for this change, since it was only a
backwards-compatible addition to the block validity rules.
-->

### eth/67 with withdrawals ([EIP-4895], March 2022)
Pos validator withdrawals가 [EIP-4895]에 따라 추가되었으며, 블록 헤더의 정의가 `withdrawals-root`를 포함하도록 변경되었고,
block bodies는 `withdrawals`를 포함하도록 변경되었습니다. 오로지 블록의 유효성 검증 룰이 이전 버전들과 호활수 있도록 하는 추가 사항이었기 때문에
해당 변경에서 어떤 새로운 와이어 프로토콜 버전도 생성되지 않았습니다.

<!--
### eth/67 ([EIP-4938], March 2022)

Version 67 removed the GetNodeData and NodeData messages.

- GetNodeData (0x0d)
  `[request-id: P, [hash₁: B_32, hash₂: B_32, ...]]`
- NodeData (0x0e)
  `[request-id: P, [value₁: B, value₂: B, ...]]`
-->

### eth/67 ([EIP-4938], March 2022)

Version 67에서 GetNodeData 와 NodeData 메시지가 삭제되었습니다.

- GetNodeData (0x0d)
  `[request-id: P, [hash₁: B_32, hash₂: B_32, ...]]`
- NodeData (0x0e)
  `[request-id: P, [value₁: B, value₂: B, ...]]`


<!--
### eth/66 ([EIP-2481], April 2021)

Version 66 added the `request-id` element in messages [GetBlockHeaders], [BlockHeaders],
[GetBlockBodies], [BlockBodies], [GetPooledTransactions], [PooledTransactions],
GetNodeData, NodeData, [GetReceipts], [Receipts].
-->

### eth/66 ([EIP-2481], April 2021)

Verseion 66에서 `request-id`요소를 [GetBlockHeaders], [BlockHeaders],
[GetBlockBodies], [BlockBodies], [GetPooledTransactions], [PooledTransactions],
GetNodeData, NodeData, [GetReceipts], [Receipts] 에 추가하였습니다.

<!--
### eth/65 with typed transactions ([EIP-2976], April 2021)

When typed transactions were introduced by [EIP-2718], client implementers decided to
accept the new transaction and receipt formats in the wire protocol without increasing the
protocol version. This specification update also added definitions for the encoding of all
consensus objects instead of referring to the Yellow Paper.
-->

### eth/65 with typed transactions ([EIP-2976], April 2021)

타입이 지정된 트랜젝션이 [EIP-2718]에서 소개되었을때, 클라이언트 구현은  새로운 프로토콜 버전의 증가 없이 wire protocol에서 새로운 트랜젝션과 repeipt 형식을 수용하기로 결정했습니다.
본 명세의 업데이트는 또한 황서를 참조하는 대신 모든 합의 객체의 인코딩에 대한 정의를 추가했습니다.

<!--
### eth/65 ([EIP-2464], January 2020)

Version 65 improved transaction exchange, introducing three additional messages:
[NewPooledTransactionHashes], [GetPooledTransactions], and [PooledTransactions].

Prior to version 65, peers always exchanged complete transaction objects. As activity and
transaction sizes increased on the Ethereum mainnet, the network bandwidth used for
transaction exchange became a significant burden on node operators. The update reduced the
required bandwidth by adopting a two-tier transaction broadcast system similar to block
propagation.
-->

### eth/65 ([EIP-2464], January 2020)

Version 65는 트랜젝션 교환을 개선하고, 다음의 세가지의 메시지 형식을 추가로 소개하였습니다.
[NewPooledTransactionHashes], [GetPooledTransactions], 그리고 [PooledTransactions] 입니다.

version 65에서 주요사항은, 피어들은 항상 완전한 트랜젝션 객체들을 서로 교환해야 합니다.
이더리움 메인넷에서 활동과 트랜젝션 사이즈가 증가하면서, 트랜젝션 교환에서 사용하는 네트워크 대역폭(bandwitdh)이 노드 운영자에게 상당한 부하가 되었기 때문입니다.
본 업데이트는 블록전파와 유사한 두 단계(two-tire)의 트랜젝션 브로드캐스트 적용함으로써 요구되는 대역폭을 줄였습니다.

<!--
### eth/64 ([EIP-2364], November 2019)

Version 64 changed the [Status] message to include the [EIP-2124] ForkID. This allows
peers to determine mutual compatibility of chain execution rules without synchronizing the
blockchain.
-->

### eth/64 ([EIP-2364], November 2019)

Verseion 64에서 [Status] 메시지를 [EIP-2124] ForkID를 포함하도록 변경했습니다.
이 변경사항은 피어들이 블록페인 동기화 없이 체인 운용 규칙의 상호 호환성을 결정할 수 있도록 합니다.


<!--
### eth/63 (2016)

Version 63 added the GetNodeData, NodeData, [GetReceipts] and [Receipts] messages
which allow synchronizing transaction execution results.
-->

### eth/63 (2016)

Version 63에서 GetNodeData, NodeData, [GetReceipts] 그리고 [Receipts] messages를 추가하였습니다.
이 메시지들은 트랜젝션 실행 결과들을 동기화 할 수 있도록 합니다.

<!--
### eth/62 (2015)

In version 62, the [NewBlockHashes] message was extended to include block numbers
alongside the announced hashes. The block number in [Status] was removed. Messages
GetBlockHashes (0x03), BlockHashes (0x04), GetBlocks (0x05) and Blocks (0x06) were
replaced by messages that fetch block headers and bodies. The BlockHashesFromNumber (0x08)
message was removed.

Previous encodings of the reassigned/removed message codes were:

- GetBlockHashes (0x03): `[hash: B_32, max-blocks: P]`
- BlockHashes (0x04): `[hash₁: B_32, hash₂: B_32, ...]`
- GetBlocks (0x05): `[hash₁: B_32, hash₂: B_32, ...]`
- Blocks (0x06): `[[header, transactions, ommers], ...]`
- BlockHashesFromNumber (0x08): `[number: P, max-blocks: P]`
-->

### eth/62 (2015)

version 62에서 [NewBlockHashes] 메시지는 알려진 해쉬들과 동시에 블록의 번호를 포함하도록 확장 되었었습니다.
[Status]의 블록 번호는 제거되었습니다.
Messages
GetBlockHashes (0x03), BlockHashes (0x04), GetBlocks (0x05) 그리고 Blocks (0x06) 메시지들은
블록 헤더와 바디를 가져오는 메시지로 대체되었습니다.
BlockHashesFromNumber (0x08) 메시지는 제거되었습니다.

<!--
### eth/61 (2015)

Version 61 added the BlockHashesFromNumber (0x08) message which could be used to request
blocks in ascending order. It also added the latest block number to the [Status] message.
-->

### eth/61 (2015)

Version 61에서 BlockHashesFromNumber (0x08) 메시지를 추가하였습니다.
이 메시지는 블록들을 오름차순으로 요청하는데 사용할 수 있었습니다.
또한 [Status]메시지에 최신 블록 번호를 추가하였습니다.

<!--
### eth/60 and below

Version numbers below 60 were used during the Ethereum PoC development phase.

- `0x00` for PoC-1
- `0x01` for PoC-2
- `0x07` for PoC-3
- `0x09` for PoC-4
- `0x17` for PoC-5
- `0x1c` for PoC-6
-->

### eth/60 and below

60이하의 버전 번호는 이더리움 PoC 개발 단계에서 사용되었습니다.

- `0x00` for PoC-1
- `0x01` for PoC-2
- `0x07` for PoC-3
- `0x09` for PoC-4
- `0x17` for PoC-5
- `0x1c` for PoC-6

[block propagation]: #block-propagation
[state synchronization]: #state-synchronization-aka-fast-sync
[snap protocol]: ./snap.md
[Status]: #status-0x00
[NewBlockHashes]: #newblockhashes-0x01
[Transactions]: #transactions-0x02
[GetBlockHeaders]: #getblockheaders-0x03
[BlockHeaders]: #blockheaders-0x04
[GetBlockBodies]: #getblockbodies-0x05
[BlockBodies]: #blockbodies-0x06
[NewBlock]: #newblock-0x07
[NewPooledTransactionHashes]: #newpooledtransactionhashes-0x08
[GetPooledTransactions]: #getpooledtransactions-0x09
[PooledTransactions]: #pooledtransactions-0x0a
[GetReceipts]: #getreceipts-0x0f
[Receipts]: #receipts-0x10
[RLPx]: ../rlpx.md
[EIP-155]: https://eips.ethereum.org/EIPS/eip-155
[EIP-1559]: https://eips.ethereum.org/EIPS/eip-1559
[EIP-2124]: https://eips.ethereum.org/EIPS/eip-2124
[EIP-2364]: https://eips.ethereum.org/EIPS/eip-2364
[EIP-2464]: https://eips.ethereum.org/EIPS/eip-2464
[EIP-2481]: https://eips.ethereum.org/EIPS/eip-2481
[EIP-2718]: https://eips.ethereum.org/EIPS/eip-2718
[transaction types]: https://eips.ethereum.org/EIPS/eip-2718
[EIP-2976]: https://eips.ethereum.org/EIPS/eip-2976
[EIP-4895]: https://eips.ethereum.org/EIPS/eip-4895
[EIP-4938]: https://eips.ethereum.org/EIPS/eip-4938
[EIP-5793]: https://eips.ethereum.org/EIPS/eip-5793
[The Merge]: https://eips.ethereum.org/EIPS/eip-3675
[London hard fork]: https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/london.md
[Shanghai fork]: https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/shanghai.md
[Yellow Paper]: https://ethereum.github.io/yellowpaper/paper.pdf