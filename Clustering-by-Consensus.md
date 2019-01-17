# 基於共識的叢集系統(Clustering by Consensus    by:Dustin J. Mitchell)

[原文出處](http://aosabook.org/en/500L/clustering-by-consensus.html)

translated by<`RanceJen`>

> Dustin is an open source software developer and release engineer at Mozilla. He has worked on projects as varied as a host configuration system in Puppet, a Flask-based web framework, unit tests for firewall configurations, and a continuous integration framework in Twisted Python. Find him as [@djmitche](http://github.com/djmitche) on GitHub or at dustin[@mozilla.com](dustin@mozilla.com).

Dustin 是一個開源軟體開發者和 Mozilla 的佈署工程師，他曾在開發過各式各樣在  Puppet 中的端口配置系統、以 Flask 為基礎的網頁框架，防火牆設置的單元測試以及一個用 Twisted Python 開發的持續集合框架

> 譯註：上面沒翻出來的都是框架名，至於 Mozilla 我相信大家都認識就沒什麼好翻的了。

### Introduction 引言
>
> In this chapter, we'll explore implementation of a network protocol designed to support reliable distributed computation. Network protocols can be difficult to implement correctly, so we'll look at some techniques for minimizing bugs and for catching and fixing the remaining few. Building reliable software, too, requires some special development and debugging techniques.

在這個章節中，我們會探索如何實現一個網路協議，並使其支援有可靠性的分散式運算。要正確的實作網路協議是有其難度的，所以我們會使用一些技巧來最小化、排查並修復 bug，以建構一個可靠的軟體，這同時也會需要一些特定的開發和除錯技巧。

### Motivating Example(啟發式案例) 
> The focus of this chapter is on the protocol implementation, but as a motivating example let's consider a simple bank account management service. In this service, each account has a current balance and is identified with an account number. Users access the accounts by requesting operations like "deposit", "transfer", or "get-balance". The "transfer" operation operates on two accounts at once -- the source and destination accounts -- and must be rejected if the source account's balance is too low.

本章節的目標是一個協議的實現，先讓我們由一個案例開始，以一個簡單的銀行帳戶服務為例，在這個服務中，每個帳戶會有其額度並以及帳戶編號，用戶可以用請求「存款」、「轉帳」、「取得額度」來操作帳戶，「轉帳」操作在兩個帳戶之間同時運作，在來源帳戶跟目標帳戶上，並且在來源帳戶額度過低時應該被拒絕。

> If the service is hosted on a single server, this is easy to implement: use a lock to make sure that transfer operations don't run in parallel, and verify the source account's balance in that method. However, a bank cannot rely on a single server for its critical account balances. Instead, the service is distributed over multiple servers, with each running a separate instance of exactly the same code. Users can then contact any server to perform an operation.

如果服務是建立在單一伺服器上，要實現的方式很簡單，用一個鎖來確保轉帳操作不會被平行的處理，並且在過程中驗證帳戶的額度。然而一個銀行重要的帳戶額度並不能只依賴單一伺服器處理。相對而言，服務應該要分散在多個伺服器中，每個都作為獨立個體執行相同的程式碼，使用者可以連接任何一個伺服器來處理操作。

> In a naive implementation of distributed processing, each server would keep a local copy of every account's balance. It would handle any operations it received, and send updates for account balances to other servers. But this approach introduces a serious failure mode: if two servers process operations for the same account at the same time, which new account balance is correct? Even if the servers share operations with one another instead of balances, two simultaneous transfers out of an account might overdraw the account.

在比較粗淺的分散處理實作中，每個伺服器在本地保存一份所有帳號的額度，並處理任何收到的操作，再傳送帳號額度的更新給其他伺服器。但這種方法存在一種嚴重的錯誤狀態，如果兩個伺服器同時處理對同一個帳戶的操作，那新的帳號額度會正確嗎？就算伺服器之間只共享操作而非額度，對帳戶同時轉帳兩次也有可能透支該帳戶。

> Fundamentally, these failures occur when servers use their local state to perform operations, without first ensuring that the local state matches the state on other servers. For example, imagine that server A receives a transfer operation from Account 101 to Account 202, when server B has already processed another transfer of Account 101's full balance to Account 202, but not yet informed server A. The local state on server A is different from that on server B, so server A incorrectly allows the transfer to complete, even though the result is an overdraft on Account 101.

從根本上來說，這個錯誤發生在當伺服器用它們本地的狀態來處理操作，卻沒先確認本地的狀態跟其他伺服器的狀態相符。舉例來說，想像當一個伺服器 A 收到一個轉帳操作從帳戶 101 轉帳到帳戶 202，此時伺服器 B 已經處理另外一個轉帳請求將帳戶 101 的全部額度都轉到帳戶 202 ，但還沒通知到伺服器 A 。這樣伺服器 A 跟 伺服器 B 的本地狀態各不相同。所以伺服器 A 就不正確的允許轉帳完成，即使這造成了帳戶 101 的透支。

### Distributed State Machines(分散式狀態機)
>
> The technique for avoiding such problems is called a "distributed state machine". The idea is that each server executes exactly the same deterministic state machine on exactly the same inputs. By the nature of state machines, then, each server will see exactly the same outputs. Operations such as "transfer" or "get-balance", together with their parameters (account numbers and amounts) represent the inputs to the state machine.
>
> The state machine for this application is simple:

用來避免上敘問題的技術就叫做「分散式狀態機」，這個主意是透過相同的輸入讓每個伺服器都形成相等的狀態機，藉由狀態機與生俱來的特性，如此伺服器就都會給出相同的輸出。而操作像是「轉帳」或「取得額度」就同時帶著它們的參數(像是帳戶編號與帳戶額度)作為狀態機的輸入。

這個應用的狀態機非常簡單

```
    def execute_operation(state, operation):
        if operation.name == 'deposit':
            if not verify_signature(operation.deposit_signature):
                return state, False
            state.accounts[operation.destination_account] += operation.amount
            return state, True
        elif operation.name == 'transfer':
            if state.accounts[operation.source_account] < operation.amount:
                return state, False
            state.accounts[operation.source_account] -= operation.amount
            state.accounts[operation.destination_account] += operation.amount
            return state, True
        elif operation.name == 'get-balance':
            return state, state.accounts[operation.account]
```

> Note that executing the "get-balance" operation does not modify the state, but is still implemented as a state transition. This guarantees that the returned balance is the latest information in the cluster of servers, and is not based on the (possibly stale) local state on a single server.

注意到執行「取得額度」操作並不會修改到狀態，但依然實作進狀態的轉換裡了，這保證回傳的額度一定是叢集中取出的最終額度，而非是某個伺服器的本地端備份。

> This may look different than the typical state machine you'd learn about in a computer science course. Rather than a finite set of named states with labeled transitions, this machine's state is the collection of account balances, so there are infinite possible states. Still, the usual rules of deterministic state machines apply: starting with the same state and processing the same operations will always produce the same output.

這可能看起來跟在資工系上課時聽到的傳統狀態機不同，它不是帶有狀態轉換的有限狀態機，這個機器的狀態是帳號額度的總集，所以有無限多種可能的狀態，但常見在狀態機上的定律依然適用於其：從相同狀態開始後，對於相同的操作流程狀態機永遠會給出相同的輸出。

> So, the distributed state machine technique ensures that the same operations occur on each host. But the problem remains of ensuring that every server agrees on the inputs to the state machine. This is a problem of consensus, and we'll address it with a derivative of the Paxos algorithm.

所以分散式狀態機的技術可以用來確保相同的操作發生在所有的端口上，但還是有疑慮就是要保證每個伺服器都要同意狀態機的輸入，這是一個共識的問題，我們會用 Paxos 演算法的延伸來解決它。


### Consensus by Paxos (基於 Paxos 的共識)

> Paxos was described by Leslie Lamport in a fanciful paper, first submitted in 1990 and eventually published in 1998, entitled "The Part-Time Parliament"1. Lamport's paper has a great deal more detail than we will get into here, and is a fun read. The references at the end of the chapter describe some extensions of the algorithm that we have adapted in this implementation.

Paxos 是由 Leslie Lamport 在其奇妙的論文中提出，最初在 1990 提交並在 1998 年公佈，題目為"The Part-Time Parliament"(兼職議會)，Lamport 的論文比我們這邊敘述的更詳細的多，讀起來也很有趣，在本文的最後有一些該演算法之延伸的參考文獻，我們會把它們應用到本次實作之中。

> The simplest form of Paxos provides a way for a set of servers to agree on one value, for all time. Multi-Paxos builds on this foundation by agreeing on a numbered sequence of facts, one at a time. To implement a distributed state machine, we use Multi-Paxos to agree on each state-machine input, and execute them in sequence.

最簡單的 Paxos 形式為一組服務器提供了一種方式來持續同意一個值，Muliti paxos 則是建立在「逐一同意一系列的事實」為基礎。為了實作一個分散式狀態機，我們使用 Multi-Paxos 來同意每個狀態機的輸入，並以序列的形式執行他們。

> The protocol operates in a series of ballots, each led by a single member of the cluster, called the proposer. Each ballot has a unique ballot number based on an integer and the proposer's identity. The proposer's goal is to get a majority of cluster members, acting as acceptors, to accept its value, but only if another value has not already been decided.

該協議由一系列的選票進行操作，每個都由叢集中的一個叫做「提案者」的成員提出，每個選票都有獨有的整數型態選票編號並由 proposer 來認證，提案者的目標是讓大多數叢集內的成員作為接收者同意其提出的價值，但前提是這個提案還沒被決定過。

![](https://i.imgur.com/iWTFzIw.png)
Figure 3.1 - A Ballot

> A ballot begins with the proposer sending a `Prepare` message with the ballot number N to the acceptors and waiting to hear from a majority (Figure 3.1.)
>
> The `Prepare` message is a request for the accepted value (if any) with the highest ballot number less than N. Acceptors respond with a `Promise` containing any value they have already accepted, and promising not to accept any ballot numbered less than N in the future. If the acceptor has already made a promise for a larger ballot number, it includes that number in the `Promise`, indicating that the proposer has been pre-empted. In this case, the ballot is over, but the proposer is free to try again in another ballot (and with a larger ballot number).

一次表決會從提案者發出 `Prepare` 訊息而開始，表決會帶有一個編號 N 來給接受者，並等待他們大多數的回應。

`Prepare` 的訊息是請求其接受小於 N 的最大表決編號的數據，接受者會回應 `Promise` 並帶有所有其已接受的數據，並承諾它之後不會再接受任何小於 N 的表決編號。而此時若接受者已經對更大的表決編號做過承諾，則會將該編號帶在 `Promise` 中，藉此向提案者說明已經被捷足先登了。在這種情況下，這次的表決就直接結束，但提案者可以自由使用(更大的)表決編號再嘗試提出另一次表決。

> When the proposer has heard back from a majority of the acceptors, it sends an `Accept` message, including the ballot number and value, to all acceptors. If the proposer did not receive any existing value from any acceptor, then it sends its own desired value. Otherwise, it sends the value from the highest-numbered promise.

當提案者收到大多數接受者的回應時，他會送出一個 `Accept` 訊息，包含表決編號和其數據給所有接受者。如果提案者沒有在接受者中收到任何現存的數據，則他送出自己預期的數據，否則他會送出來自最高表決編號之回應的數據。

> Unless it would violate a promise, each acceptor records the value from the `Accept` message as accepted and replies with an `Accepted` message. The ballot is complete and the value decided when the proposer has heard its ballot number from a majority of acceptors.

除非違反會先前的承諾，否則接受者就會紀錄下來自 `Accept` 的數據，並回應一個 `Accepted` 的訊息，當提案者收到大多數接受者回應這個提案編號時，就會定調其數據且完成此表決。


> Returning to the example, initially no other value has been accepted, so the acceptors all send back a `Promise` with no value, and the proposer sends an `Accept` containing its value, say:

回到範例上，再一開始沒有任何數據被同意過，所以接受者們都會先回傳一個不帶有任何數據的 `Promise` ，然後提案者再送出一個帶有自身數據的  `Accept` ，像是
```
 operation(name='deposit', amount=100.00, destination_account='Mike DiBernardo')
```

> If another proposer later initiates a ballot with a lower ballot number and a different operation (say, a transfer to acount `'Dustin J. Mitchell'`), the acceptors will simply not accept it. If that ballot has a larger ballot number, then the `Promise` from the acceptors will inform the proposer about Michael's $100.00 deposit operation, and the proposer will send that value in the `Accept` message instead of the transfer to Dustin. The new ballot will be accepted, but in favor of the same value as the first ballot.

如果有另外一個提案者後續才用較小的表決編號初始化的操作(像轉帳給 `'Dustin J. Mitchell'`)，接受者則單純的不同意它，而如果該表決有更高的表決編號，則在接受者傳的 `Promise` 中則會告知提案者有關先前存款 100 元的操作。然後提案者會在 `Accept` 中傳送這個值而非轉帳給 Dustin，這個表決會被接受。但其實是贊成了與第一個表決同樣的數據。

> In fact, the protocol will never allow two different values to be decided, even if the ballots overlap, messages are delayed, or a minority of acceptors fail.
>
> When multiple proposers make a ballot at the same time, it is easy for neither ballot to be accepted. Both proposers then re-propose, and hopefully one wins, but the deadlock can continue indefinitely if the timing works out just right.

事實上，這個協議永遠不會允許兩種不同的數據被定義，即使有重複的表決，訊息延遲，或是小部份的接受者故障。
當多個提案者同時提出表決時，很容易發生沒有任何表決被接受，提案者們接著又重新提案，然後期待看有沒有一者能獲勝。不過如果時機就是恰到好處則這個死鎖可以無限的持續下去。

> Consider the following sequence of events:
>
>* Proposer A performs the `Prepare`/`Promise` phase for ballot number 1.
>* Before Proposer A manages to get its proposal accepted, Proposer B performs a `Prepare`/`Promise` phase for ballot number 2.
>* When Proposer A finally sends its `Accept` with ballot number 1, the acceptors reject it because they have already promised ballot number 2.
>* Proposer A reacts by immediately sending a `Prepare` with a higher ballot number (3), before Proposer B can send its `Accept` message.
>* Proposer B's subsequent `Accept` is rejected, and the process repeats.
>
> With unlucky timing -- more common over long-distance connections where the time between sending a message and getting a response is long -- this deadlock can continue for many rounds.

考慮下列的事件順序
* 提案者 A 執行了表決 1 的 `Prepare`/`Promise` 步驟
* 在提案者 A 要接到 accepted 之前，提案者 B 執行了表決 2 的`Prepare`/`Promise` 步驟
* 在提案者 A 廣播表決 1 的 `Accept` 時，接受者拒絕了因為他們已經承諾了表決 2(不接受更小表決)
* 提案者 A 馬上傳送了一個新表決 3 的 `Prepare`，在提案者 B 可以廣播 `Accept` 之前。
* 提案者 B 隨後的 `Accept` 也遭到拒絕，然後流程不斷循環。

只要發生時的運氣不好(這常見於長距離通訊中，因為在發送到取得回應之間的時間差較長)，這個死鎖可以不斷重複很多次。

### Multi-Paxos

> Reaching consensus on a single static value is not particularly useful on its own. Clustered systems such as the bank account service want to agree on a particular state (account balances) that changes over time. We use Paxos to agree on each operation, treated as a state machine transition.

對於單個靜態值達成共識並不是特別有用，叢集系統像是銀行帳戶服務希望能不斷的同意特定的狀態轉換(像帳戶額度)，我們可以用 Paxos 來同意各個操作，就像狀態機轉換一樣處理他們。

> Multi-Paxos is, in effect, a sequence of simple Paxos instances (slots), each numbered sequentially. Each state transition is given a "slot number", and each member of the cluster executes transitions in strict numeric order. To change the cluster's state (to process a transfer operation, for example), we try to achieve consensus on that operation in the next slot. In concrete terms, this means adding a slot number to each message, with all of the protocol state tracked on a per-slot basis.

Multi-Paxos 事實上是一個簡單 Paxos 實體的序列，每個先按順序編號，對於狀態的轉換會再給予一個插槽編號，並且每個成員執行以嚴格的數字順序進行傳輸。舉例來說要轉換叢集的狀態時(就是完成一個傳輸操作)，我們會試著在下一個個插槽上達到共識，這代表對每個訊息加上插槽編號，並且每個協議都跟蹤插槽狀態
(待確認)

> Running Paxos for every slot, with its minimum of two round trips, would be too slow. Multi-Paxos optimizes by using the same set of ballot numbers for all slots, and performing the `Prepare`/`Promise` phase for all slots at once.

要在每個插槽上執行 Paxos，每次至少都必須執行兩次的來回也太慢了點，Multi-Paxos 藉由所有插槽使用同一組表決編號優化了這點，並一次執行所有插槽的 `Prepare`/`Promise` 。

### Paxos Made Pretty Hard (Paxos 是難以實現的)

> Implementing Multi-Paxos in practical software is notoriously difficult, spawning a number of papers mocking Lamport's "Paxos Made Simple" with titles like "Paxos Made Practical".

在實際的軟體中實現 Multi-Paxos 是惡名昭彰的困難，更產生了一些論文用像 "Paxos Made Practical" 的標題來嘲弄 Lamport 的 "Paxos Made Simple"。

> First, the multiple-proposers problem described above can become problematic in a busy environment, as each cluster member attempts to get its state machine operation decided in each slot. The fix is to elect a "leader" which is responsible for submitting ballots for each slot. All other cluster nodes then send new operations to the leader for execution. Thus, in normal operation with only one leader, ballot conflicts do not occur.

首先，前敘提到的多個提案者問題在繁忙的環境中越來越嚴重，因為每個叢集的成員都試圖讓其的狀態機操作在所有插槽中被決議。解決方法是選出一個 leader(領導者) 他負責提交所有插槽的表決。所有叢集中的其他結點都傳送它的新操作給領導者去執行，因此在只有單一領導者正常操作下，表決的衝突就不會再發生。

> The `Prepare`/`Promise` phase can function as a kind of leader election: whichever cluster member owns the most recently promised ballot number is considered the leader. The leader is then free to execute the `Accept`/`Accepted` phase directly without repeating the first phase. As we'll see below, leader elections are actually quite complex.

`Prepare` 和 `Promise` 的階段可以作為選舉 leader 的一部分，擁有最靠前表決編號的叢集成為就被作為領導者，接著領導者就可以放心執行 `Accept`/`Accepted` 的階段而不用重複前敘的部份。我們接著可以在下面看到，領導者的選舉也是很複雜的。

> Although simple Paxos guarantees that the cluster will not reach conflicting decisions, it cannot guarantee that any decision will be made. For example, if the initial `Prepare` message is lost and doesn't reach the acceptors, then the proposer will wait for a `Promise` message that will never arrive. Fixing this requires carefully orchestrated re-transmissions: enough to eventually make progress, but not so many that the cluster buries itself in a packet storm.

雖然 simple Paxos 保證了叢集內不會達成有衝突的決議，但它不保證會有決議被達成。舉個栗子，如果初始化的 `Prepare` 訊息丟失了沒有被傳給任何接收者，則提案者則會不斷等待永遠不會到達的 `Promise` 。要解決這個問題需要精心規劃的重送機制。要能重送到足夠繼續流程，但又不會讓整個叢集在封包風暴中炸掉。

> 瘋狂的重送給多個伺服器是真的可以炸掉整個叢集的，我們最近才炸了好幾次XDD

> Another problem is the dissemination of decisions. A simple broadcast of a Decision message can take care of this for the normal case. If the message is lost, though, a node can remain permanently ignorant of the decision and unable to apply state machine transitions for later slots. So an implementation needs some mechanism for sharing information about decided proposals.

另外一個問題是該如何傳播決議，簡單的廣播出決議訊息就可以解決一般的狀況，但如果訊息丟失則節點可能永遠不知道該決議，並且無法為後續的插槽應用狀態機轉換。所以在實作中需要一些用來分享決議提案的機制。

> Our use of a distributed state machine presents another interesting challenge: start-up. When a new node starts, it needs to catch up on the existing state of the cluster. Although it can do so by catching up on decisions for all slots since the first, in a mature cluster this may involve millions of slots. Furthermore, we need some way to initialize a new .
> 
> But enough talk of theory and algorithms -- let's have a look at the code.

我們使用的分散式狀態機還呈現了另外一個有趣的挑戰：「啟動」，當一個節點啟動時，他需要取得叢集中現存的狀態。雖然這可以用取得所有插槽所有從開始到現在的決議去解決，但這在一個足夠成熟的叢集中可能會涉及百萬個插槽，換句話說，我們需要一些用來初始化新叢集的方法。

討論了夠多理論跟演算法之後，讓我們開始看看程式碼吧。

### Introducing Cluster(認識叢集)

> The Cluster library in this chapter implements a simple form of Multi-Paxos. It is designed as a library to provide a consensus service to a larger application.

本章節中的叢集函式庫是 Multi-Paxos 的簡單實現，它被設計成一個函式庫來為大型應用提供取得共識的服務。

> Users of this library will depend on its correctness, so it's important to structure the code so that we can see -- and test -- its correspondence to the specification. Complex protocols can exhibit complex failures, so we will build support for reproducing and debugging rare failures.

該函式庫的用戶會依賴其提供的正確性，所以將如何結構化程式碼讓我們可以觀察並且測試其規格的正確性是很重要的。複雜的協議會呈現出複雜的故障，所以我們的函式庫將會支援複雜問題的重現跟除錯。

> The implementation in this chapter is proof-of-concept code: enough to demonstrate that the core concept is practical, but without all of the mundane equipment required for use in production. The code is structured so that such equipment can be added later with minimal changes to the core implementation.
>
> Let's get started.

在本章節中的實作是驗證概念用的程式碼，足夠展示我們核心的概念是很實際的，但卻不需要用到正常產品環境中的設備。而且程式碼都是結構化過的，所以這些正常產品環境中的設備也可以在稍微修改實作的核心後被加入。

讓我們開始吧！

### Types and Constants (型態與常數)

> Cluster's protocol uses fifteen different message types, each defined as a Python `namedtuple`.

叢集們會使用包含十五種不同訊息型態的協議，每一種各自用 Python 的 `namedtuple` 型態定義
```
    Accepted = namedtuple('Accepted', ['slot', 'ballot_num'])
    Accept = namedtuple('Accept', ['slot', 'ballot_num', 'proposal'])
    Decision = namedtuple('Decision', ['slot', 'proposal'])
    Invoked = namedtuple('Invoked', ['client_id', 'output'])
    Invoke = namedtuple('Invoke', ['caller', 'client_id', 'input_value'])
    Join = namedtuple('Join', [])
    Active = namedtuple('Active', [])
    Prepare = namedtuple('Prepare', ['ballot_num'])
    Promise = namedtuple('Promise', ['ballot_num', 'accepted_proposals'])
    Propose = namedtuple('Propose', ['slot', 'proposal'])
    Welcome = namedtuple('Welcome', ['state', 'slot', 'decisions'])
    Decided = namedtuple('Decided', ['slot'])
    Preempted = namedtuple('Preempted', ['slot', 'preempted_by'])
    Adopted = namedtuple('Adopted', ['ballot_num', 'accepted_proposals'])
    Accepting = namedtuple('Accepting', ['leader'])
```

> Using named tuples to describe each message type keeps the code clean and helps avoid some simple errors. The named tuple constructor will raise an exception if it is not given exactly the right attributes, making typos obvious. The tuples format themselves nicely in log messages, and as an added bonus don't use as much memory as a dictionary.
> 
> Creating a message reads naturally:

用 named tuples 去敘述各種訊息型態可以保持程式碼整潔並協助避免一些簡單的錯誤，named tuples 的建構子在沒有給定正確屬性的時候會丟出例外，讓手誤的部份變得相當明顯。 tuples 的格式在 log 中也很容易格式化，更加分的是它用的記憶體還沒有 dictionary 多。

可以很自然的閱讀創建 tuples 的訊息
```
  msg = Accepted(slot=10, ballot_num=30)
```

> And the fields of that message are accessible with a minimum of extra typing:

而且要存取該值域也很簡單

> 譯註：這句我意譯了，照字面翻很不直觀。
```
   got_ballot_num = msg.ballot_num
```

> We'll see what these messages mean in the sections that follow. The code also introduces a few constants, most of which define timeouts for various messages:

我們將在後續的部份看懂這些訊息的意思，這些程式碼引入了一些常數，它們大多是在定義各種訊息的超時。

```
    JOIN_RETRANSMIT = 0.7
    CATCHUP_INTERVAL = 0.6
    ACCEPT_RETRANSMIT = 1.0
    PREPARE_RETRANSMIT = 1.0
    INVOKE_RETRANSMIT = 0.5
    LEADER_TIMEOUT = 1.0
    NULL_BALLOT = Ballot(-1, -1)  # sorts before all real ballots
    NOOP_PROPOSAL = Proposal(None, None, None)  # no-op to fill otherwise empty slots
```

> Finally, Cluster uses two data types named to correspond to the protocol description:

最終，叢集命名下面兩種資料型態來對應協議的敘述

> 譯註：這句有點拗口，意思是「協議本身是敘述傳輸的方式」，那我們用資料型態 `namedtuple` 並命名為 `Proposal` / `Ballot` 來表達協議中的兩個敘述。
```
	Proposal = namedtuple('Proposal', ['caller', 'client_id', 'input'])
    Ballot = namedtuple('Ballot', ['n', 'leader'])
```

### Component Model(組件模型)

> Humans are limited by what we can hold in our active memory. We can't reason about the entire Cluster implementation at once -- it's just too much, so it's easy to miss details. For similar reasons, large monolithic codebases are hard to test: test cases must manipulate many moving pieces and are brittle, failing on almost any change to the code.

人類受到我們在活躍記憶中能夠擁有的東西的限制，我們沒辦法一次就推測出整個叢集的實現，因為信息量太大了，所以很容易錯失細節。相同的，一整個大型獨立架構的程式碼也是很難測試的，測資必須模擬很多環節並且測試結果也很脆弱，只要有任何一點程式改動就有可能失敗。

> To encourage testability and keep the code readable, we break Cluster down into a handful of classes corresponding to the roles described in the protocol. Each is a subclass of `Role`.

為了增強其可測試性並且保持程式可讀性，我們將叢集拆分成數個類別，分別對應協議敘述的規則，每一個都是整個規則本身的子類別。

```
class Role(object):

    def __init__(self, node):
        self.node = node
        self.node.register(self)
        self.running = True
        self.logger = node.logger.getChild(type(self).__name__)

    def set_timer(self, seconds, callback):
        return self.node.network.set_timer(self.node.address, seconds,
                                           lambda: self.running and callback())

    def stop(self):
        self.running = False
        self.node.unregister(self)
```

> The roles that a cluster node has are glued together by the `Node` class, which represents a single node on the network. Roles are added to and removed from the node as execution proceeds. Messages that arrive on the node are relayed to all active roles, calling a method named after the message type with a `do_` prefix. These `do_` methods receive the message's attributes as keyword arguments for easy access. The `Node` class also provides a `send` method as a convenience, using `functools.partial` to supply some arguments to the same methods of the `Network` class.

一個叢集所擁有的全部規則是被用 `Node` 類別連結在一起的，它代表整個網路中的單一節點，規則可以隨著運行流程被加入或從節點中移除，到達一個節點的訊息則全部依賴於運行不同的規則去調用有 `do_` 前綴的訊息類型。這些 `do_` 方法會接收訊息的屬性當作參數可以輕鬆的使用。 `Node` 的類別也提供 `send` 的方法，內部利用 `functools.partial` 來轉送一些參數給 `Network` 的 `send` 方法。

> 譯註：這段是在說明 code ，所以配合下面的程式碼觀看比較容易理解。

```

class Node(object):
    unique_ids = itertools.count()

    def __init__(self, network, address):
        self.network = network
        self.address = address or 'N%d' % self.unique_ids.next()
        self.logger = SimTimeLogger(
            logging.getLogger(self.address), {'network': self.network})
        self.logger.info('starting')
        self.roles = []
        self.send = functools.partial(self.network.send, self)

    def register(self, roles):
        self.roles.append(roles)

    def unregister(self, roles):
        self.roles.remove(roles)

    def receive(self, sender, message):
        handler_name = 'do_%s' % type(message).__name__

        for comp in self.roles[:]:
            if not hasattr(comp, handler_name):
                continue
            comp.logger.debug("received %s from %s", message, sender)
            fn = getattr(comp, handler_name)
            fn(sender=sender, **message._asdict())
```

### Application Interface(應用程序介面)

> The application creates and starts a `Member` object on each cluster member, providing an application-specific state machine and a list of peers. The member object adds a bootstrap role to the node if it is joining an existing cluster, or seed if it is creating a new cluster. It then runs the protocol (via `Network.run`) in a separate thread.

應用程式本身創建並為所有叢集成員各啟動一個 `Member` 物件，提供一個應用程式專用的狀態機及對照列表。如果節點本身是要加入一個現存的叢集則成員的物件會添加一個引導的規則給節點，或是當其實是要創造新的從即時添加 seed(一種 role) 給它，然後在不同的執行緒中執行該協議(透過 `Network.run`)

> The application interacts with the cluster through the `invoke` method, which kicks off a proposal for a state transition. Once that proposal is decided and the state machine runs, `invoke` returns the machine's output. The method uses a simple synchronized `Queue` to wait for the result from the protocol thread.

應用程式本身透過調用方法來跟叢集互動，同時也創造一個新的提案揭開狀態轉換的序幕，一旦這個提案被決議並則狀態機開始運行。 `invoke` 會回傳狀態機的輸出，這個方法用來和 `Queue` 做簡單的同步以等待協議的執行緒回傳結果。

> 譯註：這段一樣看 code 比較容易理解意思。
```
class Member(object):

    def __init__(self, state_machine, network, peers, seed=None,
                 seed_cls=Seed, bootstrap_cls=Bootstrap):
        self.network = network
        self.node = network.new_node()
        if seed is not None:
            self.startup_role = seed_cls(self.node, initial_state=seed, peers=peers,
                                      execute_fn=state_machine)
        else:
            self.startup_role = bootstrap_cls(self.node,
                                      execute_fn=state_machine, peers=peers)
        self.requester = None

    def start(self):
        self.startup_role.start()
        self.thread = threading.Thread(target=self.network.run)
        self.thread.start()

    def invoke(self, input_value, request_cls=Requester):
        assert self.requester is None
        q = Queue.Queue()
        self.requester = request_cls(self.node, input_value, q.put)
        self.requester.start()
        output = q.get()
        self.requester = None
        return output
```

### Role Classes(規則類別)

> Let's look at each of the role classes in the library one by one.
讓我們一一檢視函式庫內的規則類別。

#### Acceptor(接受者)

> The `Acceptor` implements the acceptor role in the protocol, so it must store the ballot number representing its most recent promise, along with the set of accepted proposals for each slot. It then responds to `Prepare` and `Accept` messages according to the protocol. The result is a short class that is easy to compare to the protocol.
> 
> For acceptors, Multi-Paxos looks a lot like Simple Paxos, with the addition of slot numbers to the messages.

`Acceptor` 實現了協議中接受者的規則，所以他必須保存能代表最新承諾的表決編號，以及每個插槽已接受的協議集合。然後他會根據協議回應 `Prepare` 和 `Accept` 的訊息，回應會是一個簡短的類別以至於可以輕易的跟協議本身做比較。

對於接受者來說 Multi-Paxos 跟簡單的 Paxos 看起來很相識，就是訊息會多了插槽的編號。

```
class Acceptor(Role):

    def __init__(self, node):
        super(Acceptor, self).__init__(node)
        self.ballot_num = NULL_BALLOT
        self.accepted_proposals = {}  # {slot: (ballot_num, proposal)}

    def do_Prepare(self, sender, ballot_num):
        if ballot_num > self.ballot_num:
            self.ballot_num = ballot_num
            # we've heard from a scout, so it might be the next leader
            self.node.send([self.node.address], Accepting(leader=sender))

        self.node.send([sender], Promise(
            ballot_num=self.ballot_num, 
            accepted_proposals=self.accepted_proposals
        ))

    def do_Accept(self, sender, ballot_num, slot, proposal):
        if ballot_num >= self.ballot_num:
            self.ballot_num = ballot_num
            acc = self.accepted_proposals
            if slot not in acc or acc[slot][0] < ballot_num:
                acc[slot] = (ballot_num, proposal)

        self.node.send([sender], Accepted(
            slot=slot, ballot_num=self.ballot_num))
```

#### Replica(仿製品)

> The `Replica` class is the most complicated role class, as it has a few closely related responsibilities:
> * Making new proposals;
> * Invoking the local state machine when proposals are decided;
> * Tracking the current leader; and
> * Adding newly started nodes to the cluster.

`Replica` 類別是最為複雜的規則類別，它有數個緊密相關的職責
* 發起一個新的提案
* 當提案被決議的時候調用本地的狀態機
* 追蹤現在的領導者是誰
* 為叢集加入新的節點

> The replica creates new proposals in response to `Invoke` messages from clients, selecting what it believes to be an unused slot and sending a `Propose` message to the current leader (Figure 3.2.) Furthermore, if the consensus for the selected slot is for a different proposal, the replica must re-propose with a new slot.

仿製品會為客戶端發起一個提案作為回應，選擇一個尚未使用的插槽並送訊息給當前的領導者。此外如果選擇插槽回應的共識是給其他提案的，則仿製品需要重送提案給新的插槽。