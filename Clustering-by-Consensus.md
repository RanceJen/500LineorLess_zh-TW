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

最簡單的 Paxos 形式為一組服務器提供了一種方式來持續同意一個值，Multi-Paxos 則建立在這個基礎上，藉由「逐一的同意一系列的事實」，為了實作一個分散式狀態機，我們使用 Multi-Paxos 來同意每個狀態機的輸入，並以序列的形式執行他們。

> The protocol operates in a series of ballots, each led by a single member of the cluster, called the proposer. Each ballot has a unique ballot number based on an integer and the proposer's identity. The proposer's goal is to get a majority of cluster members, acting as acceptors, to accept its value, but only if another value has not already been decided.

該協議由一系列的選票進行操作，每個都由叢集中的一個叫做「提案者」的成員提出，每個選票都有獨有的整數型態選票編號並由 proposer 來認證，提案者的目標是讓大多數叢集內的成員作為接收者同意其提出的價值，但前提是這個提案還沒被決定過。