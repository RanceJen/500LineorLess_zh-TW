# Contingent: A Fully Dynamic Build System
（Contingent: 一個完全動態的建構系統）
## 作者： Brandon Rhodes and Daniel Rocco

translated by<`Hoching`>

[原文出處](http://aosabook.org/en/500L/contingent-a-fully-dynamic-build-system.html)
 
> Brandon Rhodes started using Python in the late 1990s, and for 17 years has maintained the PyEphem library for amateur astronomers. He works at Dropbox, has taught Python programming courses for corporate clients, consulted on projects like the New England Wildflower Society's “Go Botany” Django site, and will be the chair of the PyCon conference in 2016 and 2017. Brandon believes that well-written code is a form of literature, that beautifully formatted code is a work of graphic design, and that correct code is one of the most transparent forms of thought.

Brandon Rhodes 在 1990 年代末期開始使用 Python，17 年來一直為業餘天文學家維護 PyEphem 函式庫。他在 Dropbox 工作，曾經教授企業客戶 Python 編程的課程，在一些專案中也曾被諮詢，像是新英格蘭野花協會的 “Go Botany” Django 網站，並將在 2016 年和 2017 年擔任 PyCon 會議的主席。Brandon 認為寫得很好的程式碼是一種文學形式，格式精美的程式碼是圖形設計的工作，正確的代碼則是最透明的思想形式之一。

> Daniel Rocco loves Python, coffee, craft, stout, object and system design, bourbon, teaching, trees, and Latin guitar. Thrilled that he gets to write Python for a living, he is always on the lookout for opportunities to learn from others in the community, and to contribute by sharing knowledge. He is a frequent speaker at PyAtl on introductory topics, testing, design, and shiny things; he loves seeing the spark of wonder and delight in people's eyes when someone shares a novel, surprising, or beautiful idea. Daniel lives in Atlanta with a microbiologist and four aspiring rocketeers.

Daniel Rocco 喜歡 Python、咖啡、工藝品、司陶特啤酒、物件和系統設計、波本威士忌、教學、樹木和拉丁吉他。他以寫 Python 維生，並為此感到非常興奮。他總是在尋找機會向社區中的其他人學習，並透過分享知識做出貢獻。他經常在 PyAtl 上發表演講，關於介紹性的主題、測試、設計和閃亮的事物。他喜歡看到當有人分享一個新穎、令人驚訝或美麗的想法時，人們眼中驚奇和喜悅的火花。丹尼爾和一名微生物學家及四名有抱負的火箭專家住在亞特蘭大。

## Introduction （介紹）

> Build systems have long been a standard tool within computer programming.

Build 系統一直以來都是電腦程式設計中重要的標準工具。

> The standard `make` build system, for which its author won the ACM Software System Award, was first developed in&nbsp;1976. It not only lets you declare that an output file depends upon one (or more) inputs, but lets you do this recursively. A&nbsp;program, for example, might depend upon an object file which itself depends upon the corresponding source code:

標準的 `make` build 系統一開始在 1976 被開發，它的作者還贏得了 ACM 軟體系統獎。它不只讓你可以根據一個以上的輸入宣告一個輸出的檔案，還可以讓你遞迴地去做。舉例來說，一個程式可能依賴一個 object file，而 object file 又依賴於對應的程式碼。

```
    prog: main.o
            cc -o prog main.o

    main.o: main.c
            cc -C -o main.o main.c
```

> Should `make` discover, upon its next invocation, that the `main.c` source code file now has a more recent modify time than `main.o`, then it will not only rebuild the `main.o` object file but will also rebuild `prog` itself.

如果 `make` 在下一次調用時，發現到 `main.c` 程式碼最近的修改時間比 `main.o` 還要晚，那它就不只會 rebuild `main.o` object file，它也會 rebuild `prog` 自己。

> Build systems are a common semester project assigned to undergraduate computer science students — not only because build systems are used in nearly all software projects, but because their construction involves fundamental data structures and algorithms involving directed graphs (which this chapter will later discuss in more detail).

Build systems 一般來說是指派給資工系大學生，一個學期的專案。不只因為 build systems 幾乎被用在所有軟體專案，也是因為它們的建造牽涉到基礎的資料結構和關於 directed graphs （在待會會提到更多細節） 的演算法。

> With decades of use and practice behind build systems, one might expect them to have become completely general-purpose and ready for even the most extravagant demands. But, in fact, one kind of common interaction between build artifacts — the problem of dynamic cross-referencing — is handled so poorly by most build systems that in this chapter we are inspired to not only rehearse the classic solution and data structures used to solve the `make` problem, but to extend that solution dramatically, to a far more demanding domain.

build systems 在經過數十年的使用和實行後，我們可能會期待它們在一般用途下已經很完善，甚至對於過分的要求也都有所準備。但事實上，在 build artifacts 中一種常見的交互作用 —— 動態交叉引用的問題，在大部分的 build systems 中都處理得很糟糕，受到此啟發，我們在本章節中不只會詳述這個經典問題和用來解決此 `make` 問題的資料結構，我們也會大幅延伸此解決方案到更耗費心力的領域。

> The problem, again, is cross-referencing. Where do cross-references tend to emerge? In text documents, documentation, and printed books!

問題還是出在交叉引用上。而交叉引用會在哪裡出現呢？在教科書文檔、文件和印刷書籍裡。


## The Problem: Building Document Systems（問題：建構文檔系統）

> Systems to rebuild formatted documents from source always seem to do too much work, or too little.

系統去從原始檔 rebuild 格式化的文檔總是會做太多或太少工。

> They do too much work when they respond to a minor edit by making you wait for unrelated chapters to be re-parsed and re-formatted. But they can also rebuild too little, leaving you with an inconsistent final product.

系統會做太多工，當你只是做了一個小小的編輯，它們卻會重新解析和重新格式化不相關的章節。但它們也會做太少工，讓你拿到一個不一致的最終結果。

> Consider [Sphinx](http://sphinx-doc.org/), the document builder that is used for both the official Python language documentation and many other projects in the Python community. A Sphinx project’s `index.rst` will usually include a table of contents:

以 [Sphinx](http://sphinx-doc.org/) 來說，這個檔案 builder 會被用在官方的 Python 說明書和其他很多 Python 社群的專案。一個 Sphinx 專案的 `index.rst` 通常會包含一個目錄：

```
   Table of Contents
   =================

   .. toctree::

      install.rst
      tutorial.rst
      api.rst
```

> This list of chapter filenames tells Sphinx to include a link to each of the three named chapters when it builds the `index.html` output file. It will also include links to any sections within each chapter. Stripped of its markup, the text that results from the above title and `toctree` command might be:

這個章節檔案名稱的列表告訴 Sphinx 在 builds `index.html` 檔案時，要去 include 一個連結到
這三個已被命名的章節。它也會 include 連結到章節裡的任一個段落。省略掉標記的話，以上的標題和 `toctree` 指令的文字結果會是：

```
  Table of Contents

  • Installation

  • Newcomers Tutorial
      • Hello, World
      • Adding Logging

  • API Reference
      • Handy Functions
      • Obscure Classes
```

> This table of contents, as you can see, is a mash-up of information from four different files. While its basic order and structure come from `index.rst`, the actual titles of each chapter and section are pulled from the three chapter source files themselves.

這個表格的內容是由四個不同檔案的資訊混搭而成。它的基本順序和架構是由 `index.rst` 來的，實際上每個章節和段落的標題則是從三個章節的原始檔案來的。

> If you later reconsider the tutorial’s chapter title — after all, the word “newcomer” sounds so quaint, as if your users are settlers who have just arrived in pioneer Wyoming — then you would edit the first line of `tutorial.rst` and write something better:

假如你重新思考 tutorial 那章的標題 —— 畢竟 “newcomer” 這個字聽起來太過時了，好像你的使用者們是最開始抵達懷俄明州開墾的居民。你應該會編輯 `tutorial.rst` 的第一行，改寫得好一點：

```
  -Newcomers Tutorial
  +Beginners Tutorial
   ==================

   Welcome to the tutorial!
   This text will take you through the basics of...
```

> When you are ready to rebuild, Sphinx will do exactly the right thing! It will rebuild both the tutorial chapter itself, and the index. (Piping the output into `cat` makes Sphinx announce each rebuilt file on a separate line, instead of using bare carriage returns to repeatedly overwrite a single line with these progress updates.)

當你準備好要 rebuild，Sphinx 會完全正確地執行！它將會 rebuild tutorial 章節和 index。（將輸出導入 `cat` 會讓 Sphinx 在獨立一行宣佈每一個 rebuilt 的檔案，而不是用最基本的回車字元去用這些進度的更新來重覆覆寫單一行）

```
   $ make html | cat
   writing output... [ 50%] index
   writing output... [100%] tutorial
```

> Because Sphinx chose to rebuild both documents, not only will `tutorial.html` now feature its new title up at the top, but the output `index.html` will display the updated chapter title in the table of contents. Sphinx has rebuilt everything so that the output is consistent.

因為 Sphinx 選擇去重 build 了兩個文檔，不只 `tutorial.html` 在頂部有了新的標題， `index.html` 也會在內容的表格中顯示更新過後的章節標題。Sphinx 重 build 了所有東西所以最後輸出是一致的。

> What if your edit to `tutorial.rst` is more minor?

如果你在 `tutorial.rst` 修改的地方更細微呢？

```
   Beginners Tutorial
   ==================

  -Welcome to the tutorial!
  +Welcome to our project tutorial!
   This text will take you through the basics of...
```

> In this case there is no need to rebuild `index.html` because this minor edit to the interior of a paragraph does not change any of the information in the table of contents. But it turns out that Sphinx is not quite as clever as it might have at first appeared! It will go ahead and perform the redundant work of rebuilding `index.html` even though the resulting contents will be exactly the same.
 
在這個情況下不需要重 build `index.html`，因為這個對於段落內部細微的修改，並不會對目錄的資訊有任何改變。但結果 Sphinx 並沒有它一開始看起來那麼聰明，它會去執行冗餘的工作，也就是重 build `index.html`，即使產生的內容會是完全一樣的。

```
   writing output... [ 50%] index
   writing output... [100%] tutorial`</pre>
```

> You can run `diff` on the “before” and “after” versions of `index.html` to confirm that your small edit has had no effect on the front page — yet Sphinx made you wait while it was rebuilt anyway.

你可以在 `index.html` 前後的版本執行 `diff`，來確認說你微小的改動並不會影響到首頁，然而 Sphinx 卻會讓你等待它重 build。

> You might not even notice the extra rebuild effort for small documents that are easy to compile. But the delay to your workflow can become significant when you are making frequent tweaks and edits to documents that are long, complex, or that involve the generation of multimedia like plots or animations. While Sphinx is at least making an effort not to rebuild every chapter when you make a single change — it has not, for example, rebuilt `install.html` or `api.html` in response to your `tutorial.rst` edit — it is doing more than is necessary.

對這種很容易編譯的小文檔來說，你可能甚至不會注意到重 rebuild 需要多耗費什麼力氣。然而當你需要頻繁的調整和編輯的文檔很長、很複雜或是牽涉到多媒體的產生，像是圖片或是動畫，就有可能對你的工作流程造成顯著的延遲。當你做了單一的改動，即使 Sphinx 已經努力不去重 build 每個章節，例如說，對於你在 `tutorial.rst` 的編輯，它已經沒有重 build `install.html` 或是 `api.html`，但它還是做了一些不必要的事。

> But it turns out that Sphinx does something even worse: it sometimes does too little, leaving you with inconsistent output that could be noticed by users.

但 Sphinx 甚至會做出更糟的事：它有時候會做太少，以至於讓你獲得不一致的輸出，那可能會被使用者發現。

> To see one of its simplest failures, first add a cross reference to the top of your API documentation:

要看到它其中一種最簡單的錯誤，首先在你的 API 說明文件頂端加上交叉引用。

```
   API Reference
   =============

  +Before reading this, try reading our :doc:`tutorial`!
  +
   The sections below list every function
   and every single class and method offered...
```

> With its usual caution as regards the table of contents, Sphinx will dutifully rebuild both this API reference document as well as the `index.html` home page of your project:

由於對於目錄的謹慎，Sphinx 會盡職地重 build 這個 API 參考文檔和你專案的首頁 `index.html`：

```
   writing output... [ 50%] api
   writing output... [100%] index
```

> In the `api.html` output file you can confirm that Sphinx has included the attractive human-readable title of the tutorial chapter into the cross reference’s anchor tag:

在 `api.html` 的輸出檔案裡，你可以確認 Sphinx 已經包含了 tutorial 章節裡吸引人且人類可讀的標題，就放在交叉引用的 anchor tag 裡： 

```html
   <p>Before reading this, try reading our
   <a class="reference internal" href="tutorial.html">
     <em>Beginners Tutorial</em>
   </a>!</p>
```

> What if you now make another edit to the title at the top of the `tutorial.rst` file? You will have invalidated **three** output files:
1. The title at the top of `tutorial.html` is now out of date, so the file needs to be rebuilt.
2. The table of contents in `index.html` still has the old title, so that document needs to be rebuilt.
3. The embedded cross reference in the first paragraph of `api.html` still has the old chapter title, and also needs to be rebuilt.

要是你現在對 `tutorial.rst` 檔案頂部的標題做另一個改動？你會讓**三個**輸出檔案變得無效：
1. `tutorial.html` 檔案頂部的標題變得過時，所以檔案需要被重 build。
2. `index.html` 檔案裡的目錄還是舊的標題，所以該檔案需要被重 build。
3. `api.html` 檔案裡第一的段落嵌入式的交叉引用還是舊的章節標題，所以也需要重 build。

> What does Sphinx do?

Sphinx 會做些什麼呢？

```
   writing output... [ 50%] index
   writing output... [100%] tutorial
```   

> Whoops.

糟了。

> Only two files were rebuilt, not three. Sphinx has failed to correctly rebuild your documentation.

只有兩個檔案被重 build，而不是三個。Sphinx 沒有正確地重 build 你的文件。

> If you now push your HTML to the web, users will see the old title in the cross reference at the top of `api.html` but then a different title — the new one — once the link has carried them to `tutorial.html` itself. This can happen for many kinds of cross reference that Sphinx supports: chapter titles, section titles, paragraphs, classes, methods, and functions.

如果你這時將你的 HTML 推到網路上，使用者們在 `api.html` 檔案頂部會看到交叉引用的舊標題，但一旦他們被連結帶到 `tutorial.html` ，他們會接著看到不同的標題，也就是新的標題。這可能會發生在很多種 Sphinx 支援的交叉引用：章標題、節標題、段落、類別、方法和函式們。

## Build Systems and Consistency

> The problem outlined above is not specific to Sphinx. Not only does it haunt other document systems, like LaTeX, but it can even plague projects that are simply trying to direct compilation steps with the venerable `make` utility, if their assets happen to cross-reference in interesting ways.

以上列出的問題並不是 Sphinx 特有的。它不止會出現在其他像是 LaTeX 的文檔系統上，它也困擾很多只是嘗試用崇高神聖的 `make` 來管理彙編步驟的專案們，如果它們剛好用有趣的方式去做交叉引用的話。

> As the problem is ancient and universal, its solution is of equally long lineage:

由於這個問題是古老且普遍的，它的解法也同樣沿襲下來：

```bash
$ rm -r _build/
$ make html
```

> If you remove all of the output, you are guaranteed a complete rebuild! Some projects even alias `rm` `-r` to a target named `clean` so that only a quick `make` `clean` is necessary to wipe the slate.

如果你移除掉全部的輸出，就可以保證會做一次完整的重新建置。一些專案甚至會把 `clean` 當作 `rm` `-r` 的別名，這樣就只需要用 `make` `clean` 就可以快速地將過去一筆勾銷。  

> By eliminating every copy of every intermediate or output asset, a hefty `rm` `-r` is able to force the build to start over again with nothing cached — with no memory of its earlier state that could possibly lead to a stale product.

藉由清除每一個中間產物或是輸出的副本，沈重的 `rm` `-r` 就可以強迫這個建置重新開始，不會有任何東西被快取住，也就不會有任何存有之前狀態的記憶體去導到舊的產物。 

> But could we develop a better approach?

但我們可以開發出更好的方法嗎？

> What if your build system were a persistent process that noticed every chapter title, every section title, and every cross-referenced phrase as it passed from the source code of one document into the text of another? Its decisions about whether to rebuild other documents after a change to a single source file could be precise, instead of mere guesses, and correct, instead of leaving the output in an inconsistent state.

如果你的建置系統是一個一貫的程序，會注意到每個章標題、每個節標題和每個交叉引用的片語

> The result would be a system like the old static `make` tool, but which learned the dependencies between files as they were built — that added and removed dependencies dynamically as cross references were added, updated, and deleted.

結果將會是一個類似於舊的靜態 `make` 工具的系統，但是它在建構文件時學習了文件之間的依賴關係 —— 在添加、更新和刪除交叉引用時，依賴性會動態地添加和刪除。

> In the sections that follow we will construct such a tool, named Contingent, in Python. Contingent guarantees correctness in the presence of dynamic dependencies while performing the fewest possible rebuild steps. While it can be applied to any problem domain, we will run it against a small version of the problem outlined above.

在接下來的部分中，我們將在 Python 中構建一個名為 Contingent 的工具。在執行盡可能少的重建步驟的同時，Contingent 保證存在動態依賴性時的正確性。雖然它可以應用於任何問題域，但我們將針對上述問題的小版本來執行它。


## Linking Tasks to Make a Graph（連結任務們來做圖）

> Any build system needs a way to link inputs and outputs. The three markup texts in our discussion above, for example, each produce a corresponding HTML output file. The most natural way to express these relationships is as a collection of boxes and arrows — or, in mathematical terminology, <em>nodes</em> and <em>edges</em> — to form a <em>graph</em>.

任何一個建置系統都需要有方法來連結輸入和輸出。舉例來說，在我們前面的討論中有三個標記文本，每一個都會製造出相對應的 HTML 輸出檔。表示這些關係最自然的方式就是用一些方塊和箭頭，以數學名詞來說就是點和線，來組成一張圖。

![figure1](http://aosabook.org/en/500L/contingent-images/figure1.png)

<small>Figure 4.1 - Three files generated by parsing three input texts.</small><a id="figure1"></a>
<small>圖4.1 —— 藉由解析三個輸入文檔來產生三個檔案.</small>

> Each language in which a programmer might tackle writing a build system will offer various data structures with which such a graph of nodes and edges might be represented.

每一種程式設計師可能會拿來寫建置系統的語言都提供了各式各樣的資料結構，以此就應該可以表現出一張圖中的點和線。

> How could we represent such a graph in Python?

我們要如何用 Python 表現出那樣的一張圖呢？

> The Python language gives priority to four generic data structures by giving them direct support in the language syntax. You can create new instances of these big-four data structures by simply typing their literal representation into your source code, and their four type objects are available as built-in symbols that can be used without being imported.

Python 語言會優先考慮四種通用數據結構，在語言語法中提供直接支持。你可以藉由在原始碼中鍵入它們文字上的表示來創建這些四大數據結構的新實例，並且它們的四個類型對象可作為內置符號使用，無需導入即可使用。

> The **tuple** is a read-only sequence used to hold heterogeneous data — each slot in a tuple typically means something different. Here, a tuple holds together a hostname and port number, and would lose its meaning if the elements were re-ordered:

**tuple** 是一個唯讀的序列，被用來存異質性的資料：一般來說，每一個 tuple 中的位置都代表不一樣的東西。此處的範例來說，一個 tuple 把一個主機名稱和 port 號存在一起

```python
('dropbox.com', 443)
```

> The **list** is a mutable sequence used to hold homogenous data — each item usually has the same structure and meaning as its peers. Lists can be used either to preserve data’s original input order, or can be rearranged or sorted to establish a new and more useful order.

**list** 是一個可變的序列，被用來存同質性的資料：每一個項目通常和它的同伴們有一樣的結構和意思。 Lists 可以被用來保存資料原始的輸入順序，也可以重新安排或排序以建立一個新的且更有用的順序。

```python
['C', 'Awk', 'TCL', 'Python', 'JavaScript']
```

> The **set** does not preserve order. Sets remember only whether a given value has been added, not how many times, and are therefore the go-to data structure for removing duplicates from a data stream. For example, the following two sets will each have three elements:

**set** 沒辦法保存順序。 set 只會記得一個數值是否被加入過，不會知道被加了幾次，因此該資料結構會被用來移除數據流之中重複的資料。例如，以下的兩個 set 各會有三個元素：

```python
{3, 4, 5}
{3, 4, 5, 4, 4, 3, 5, 4, 5, 3, 4, 5}
```

> The **dict** is an associative data structure for storing values accessible by a key. Dicts let the programmer chose the key by which each value is indexed, instead of using automatic integer indexing as the tuple and list do. The lookup is backed by a hash table, which means that dict key lookup runs at the same speed whether the dict has a dozen or a million keys.

**dict** 是一種關連性的資料結構，用於儲存可用 key 訪問的值。Dicts 讓程式設計師選擇索引每個值的 key，而不是像 tuple 和 list 使用自動整數索引。查找由哈希表支持，這意味著無論 dict 有十幾個還是一百萬個 key，dict 用 key 查找都以相同的速度運行。

```python
{'ssh': 22, 'telnet': 23, 'domain': 53, 'http': 80}
```

> A key to Python’s flexibility is that these four data structures are composable. The programmer can arbitrarily nest them inside each other to produce more complex data stores whose rules and syntax remain the simple ones of the underlying tuples, lists, sets, and dicts.

Python 靈活性的關鍵是這四種數據結構是可組合的。程式設計師可以任意地將它們嵌套在一起，以生成更複雜的數據存儲，其規則和語法仍然是簡單的底層 tuple, list, set, 和 dict。

> Given that each of our graph edges needs to know at least its origin node and its destination node, the simplest possible representation would be a tuple. The top edge in [Figure 4.1](#figure1) might look like:

鑑於我們每個圖形的邊緣至少需要知道其原始節點及其目標節點，最簡單的表示形式將​​是 tuple。[圖 4.1]（＃figure1）中的上邊緣可能如下所示：

```python
    ('tutorial.rst', 'tutorial.html')
```

> How can we store several edges? While our initial impulse might be to simply throw all of our edge tuples into a list, that would have disadvantages. A list is careful to maintain order, but it is not meaningful to talk about an absolute order for the edges in a graph. And a list would be perfectly happy to hold several copies of exactly the same edge, even though we only want it to be possible to draw a single arrow between `tutorial.rst` and `tutorial.html`. The correct choice is thus the set, which would have us represent [Figure 4.1](#figure1) as:

我們如何儲存多個邊呢？雖然我們最初的衝動可能只是簡單地將所有邊緣 tuple 都放入 list 中，但這會有缺點。列表會小心維護順序，但談論圖中邊的絕對順序沒有意義。並且列表會非常樂意保存同一個邊緣的多個副本，即使我們只希望它可以在 `tutorial.rst` 和 `tutorial.html` 之間繪製單個箭頭。因此正確的選擇是 set，我們會將[圖 4.1](#figure1)表示為：

```python
    {('tutorial.rst', 'tutorial.html'),
     ('index.rst', 'index.html'),
     ('api.rst', 'api.html')}
```

> This would allow quick iteration across all of our edges, fast insert and delete operations for a single edge, and a quick way to check whether a particular edge was present.

這會讓我們能夠快速地遍歷所有的邊，迅速地執行單一邊緣的插入和刪除，而且是一個可以很快確認一個特定的邊是否存在的方式。

> Unfortunately, those are not the only operations we need.

很不幸的，我們不只需要執行那些操作。

> A build system like Contingent needs to understand the relationship between a given node and all the nodes connected to it. For example, when `api.rst` changes, Contingent needs to know which assets, if any, are affected by that change in order to minimize the work performed while also ensuring a complete build. To answer this question — “what nodes are downstream from `api.rst`?” — we need to examine the <em>outgoing</em> edges from `api.rst`.

像 Contingent 這樣的建構系統需要了解給定節點和連接到它的所有節點之間的關係。例如，當 `api.rst` 發生變化時，Contingent 需要知道哪些資產（如果有的話）會受到該變化的影響，以便最小化所需執行的工作，同時確保能夠完整構建。要回答這個問題 —— “什麼節點是 `api.rst` 的下游？” —— 我們需要檢查從 `api.rst` *傳出* 的邊緣。

> But building the dependency graph requires that Contingent be concerned with a node's <em>inputs</em> as well. What inputs were used, for example, when the build system assembled the output document `tutorial.html`? It is by watching the input to each node that Contingent can know that `api.html` depends on `api.rst` but that `tutorial.html` does not. As sources change and rebuilds occur, Contingent rebuilds the incoming edges of each changed node to remove potentially stale edges and re-learn which resources a task uses this time around.

但是構建相依關係的圖需要 Contingent 也關注節點的*輸入*。什麼輸入被用到了？例如，當建構系統組裝輸出文檔 `tutorial.html` 時用了什麼輸入？藉由觀察每個節點的輸入，Contingent 可以知道 `api.html` 依賴於 `api.rst` 而 `tutorial.html` 沒有。當來源檔案更改而重建發生時，Contingent 將重建每個已更改節點的傳入邊緣以刪除可能過時的邊緣，並重新了解任務此時使用的資源。

> Our set-of-tuples does not make answering either of these questions easy. If we needed to know the relationship between `api.html` and the rest of the graph, we would need to traverse the entire set looking for edges that start or end at the `api.html` node.

我們的 set-of-tuples 不能輕鬆回答這些問題。如果我們需要知道 `api.html` 和圖中其餘部分之間的關係，我們需要遍歷整個集合，尋找從 `api.html` 節點開始或結束的邊。

> An associative data structure like Python's dict would make these chores easier by allowing direct lookup of all the edges from a particular node:

像 Python 的 dict 這樣的關聯性數據結構可以透過直接查找特定節點的所有邊緣來簡化這些雜務：

```python
    {'tutorial.rst': {('tutorial.rst', 'tutorial.html')},
     'tutorial.html': {('tutorial.rst', 'tutorial.html')},
     'index.rst': {('index.rst', 'index.html')},
     'index.html': {('index.rst', 'index.html')},
     'api.rst': {('api.rst', 'api.html')},
     'api.html': {('api.rst', 'api.html')}}
```

> Looking up the edges of a particular node would now be blazingly fast, at the cost of having to store every edge twice: once in a set of incoming edges, and once in a set of outgoing edges. But the edges in each set would have to be examined manually to see which are incoming and which are outgoing. It is also slightly redundant to keep naming the node over and over again in its set of edges.

查看特定節點的邊緣會變得非常快，代價是必須將每個邊緣存儲兩次：一次在一個輸入邊緣的 set，一次在一個輸出邊緣的 set。但是必須手動檢查每個 set 中的邊緣，以查看哪些是傳入的，哪些是傳出的。在其邊緣的 set 中反復命名節點也有點冗餘。

> The solution to both of these objections is to place incoming and outgoing edges in their own separate data structures, which will also absolve us of having to mention the node over and over again for every one of the edges in which it is involved.

這兩個異議的解決方案是將輸入和輸出邊緣放置在它們自己獨立的數據結構中，這也使我們不必因為涉及某節點的每個邊緣而反覆提及該節點。

```python
    incoming = {
        'tutorial.html': {'tutorial.rst'},
        'index.html': {'index.rst'},
        'api.html': {'api.rst'},
        }

    outgoing = {
        'tutorial.rst': {'tutorial.html'},
        'index.rst': {'index.html'},
        'api.rst': {'api.html'},
        }
```

> Notice that `outgoing` represents, directly in Python syntax, exactly what we drew in [Figure 4.1](#figure1) earlier: the source documents on the left will be transformed by the build system into the output documents on the right. For this simple example each source points to only one output — all the output sets have only one element — but we will see examples shortly where a single input node has multiple downstream consequences.

請注意，`outgoing` 直接用 Python 語法表示我們之前在[圖 4.1](#figure1)中提到的內容：左側的原始文檔將由建構系統轉換為右側的輸出文檔。對於這個簡單的範例，每個來源只指向一個輸出 —— 所有輸出的 set 只有一個元素 —— 但我們很快將會看到單個輸入節點擁有多個下游結果的例子。

> Every edge in this dictionary-of-sets data structure does get represented twice, once as an outgoing edge from one node (`tutorial.rst` → `tutorial.html`) and again as an incoming edge to the other (`tutorial.html` ← `tutorial.rst`). These two representations capture precisely the same relationship, just from the opposite perspectives of the two nodes at either end of the edge. But in return for this redundancy, the data structure supports the fast lookup that Contingent needs.

在這個 dictionary-of-sets 的資料結構裡，每一個邊都被表示了兩次，一次是作為從某節點出發的邊 (`tutorial.rst` → `tutorial.html`) ，再一次是作為另一節點進來的邊 (`tutorial.html` ← `tutorial.rst`)。這兩種表現方式都準確地描述了同樣的關係，只是分別從一個邊兩端頂點的相反角度來看。但因為這有點冗餘，此資料結構提供了 Contingent 所需的快速查找方式。


## The Proper Use of Classes（類別的妥善使用）

> You may have been surprised by the absence of classes in the above discussion of Python data structures. After all, classes are a frequent mechanism for structuring applications and a hardly less-frequent subject of heated debate among their adherents and detractors. Classes were once thought important enough that entire educational curricula were designed around them, and the majority of popular programming languages include dedicated syntax for defining and using them.

你可能對於類別沒有出現在以上 Python 資料結構的討論而感到驚訝，畢竟，類別對於建構應用程式來說是很常用的機制，也是他們的追隨者和批評者激烈爭論中不那麼頻繁出現的主題。類別曾經被覺得重要到整個教育課程都圍繞著它做設計，且大多數的程式語言也都包含類別定義和使用的專門語法。

> But it turns out that classes are often orthogonal to the question of data structure design. Rather than offering us an entirely alternative data modeling paradigm, classes simply repeat data structures that we have already seen:

但結果類別通常和資料結構設計的問題正交。類別並沒有為我們提供完全替代的資料建模範例，而只是單純地重複我們已經看到的資料結構：

> * A class instance is <em>implemented</em> as a dict.
> * A class instance is <em>used</em> like a mutable tuple.

* 一個類別實例被實作成 dict
* 一個類別實例被拿來當作 mutable tuple 使用 

> The class offers key lookup through a prettier syntax, where you get to say `graph.incoming` instead of `graph["incoming"]`. But, in practice, class instances are almost never used as generic key-value stores. Instead, they are used to organize related but heterogeneous data by attribute name, with implementation details encapsulated behind a consistent and memorable interface.

這個類別對於鍵的查詢提供了更漂亮的語法，你會需要用 `graph.incoming` 而不是 `graph["incoming"]`。但實際上，類別實例幾乎從來不會用於一般的鍵值對儲存。它們會被用來以屬性名稱去組織相關但異質性的資料，實作的內容都被封裝在一個一致且可記憶的介面。

> So instead of putting a hostname and a port number together in a tuple and having to remember which came first and which came second, you create an `Address` class whose instances each have a `host` and a `port` attribute. You can then pass `Address` objects around where otherwise you would have had anonymous tuples. Code becomes easier to read and easier to write. But using a class instance does not really change any of the questions we faced above when doing data design; it just provides a prettier and less anonymous container.

原本需要把主機名稱和埠號一起放在一個 tuple，且必須記得哪個是第一哪個是第二，取而代之地，你只需要創建一個 `Address` 類別，它的每一個實例都會有 `host` 和 `port` 的屬性。你就可以四處傳遞 `Address` 物件，而不是用匿名的 tuples，程式碼會變得簡單讀和寫。但使用類別實例其實並沒有改變我們以上在資料設計上遇到的任何問題，它只是提供了一個比較美觀且比較不匿名的容器。

> The true value of classes, then, is not that they change the science of data design. The value of classes is that they let you <em>hide</em> your data design from the rest of a program!

類別實際上的價值不是他改變了資料設計的科學，而是它讓你*隱藏*了你的資料設計，不被程式的其他部分看到。

> Successful application design hinges upon our ability to exploit the powerful built-in data structures Python offers us while minimizing the volume of details we are required to keep in our heads at any one time. Classes provide the mechanism for resolving this apparent quandary: used effectively, a class provides a facade around some small subset of the system's overall design. When working within one subset — a `Graph`, for example — we can forget the implementation details of other subsets as long as we can remember their interfaces. In this way, programmers often find themselves navigating among several levels of abstraction in the course of writing a system, now working with the specific data model and implementation details for a particular subsystem, now connecting higher-level concepts through their interfaces.

成功的應用程序設計取決於我們利用 Python 提供給我們的強大內建數據結構的能力，同時最大限度地減少我們在任何時候需要維持在腦中的細節量。類別提供了解決這個明顯困境的機制：有效率的使用下，一個類別可以提供圍繞系統整體設計的子集合的外觀。當在處理其中的一個子集合時，舉例來說：一個 `Graph`，我們可以不去記得其他子集合的實作細節，只要我們可以記住它們的操作介面。這樣一來，程式設計師常常會覺得他們在編寫系統的過程中是在好幾個抽象層之間航行，一下為了做一個特定的子系統，需要特定的資料模型和實作細節；一下需要透過他們的介面來連結更高層級的概念。

> For example, from the outside, code can simply ask for a new `Graph` instance:

舉例來說，從外層，程式碼只可以要求一個新的 `Graph` 實例：

```python
>>> from contingent import graphlib
>>> g = graphlib.Graph()
```

> without needing to understand the details of how `Graph` works. Code that is simply using the graph sees only interface verbs — the method calls — when manipulating a graph, as when an edge is added or some other operation performed:

程式碼只是要用圖（graph）的話，不需要了解 `Graph` 運作的細節，只會看到介面的動詞 —— 方法的名稱。像是當你操作一張圖，要加上一個邊或是做其他的操作：

```python
>>> g.add_edge('index.rst', 'index.html')
>>> g.add_edge('tutorial.rst', 'tutorial.html')
>>> g.add_edge('api.rst', 'api.html')
```

> Careful readers will have noticed that we added edges to our graph without explicitly creating “node” and “edge” objects, and that the nodes themselves in these early examples are simply strings. Coming from other languages and traditions, one might have expected to see user-defined classes and interfaces for everything in the system:

仔細的讀者會發現我們在圖上新增邊的時候，並沒有明確地創造一個“節點”和“邊”的物件，而且在之前的範例中，這些節點都只是單純的字串。由於其他的語言和傳統，我們可能會預期看到每一個系統中的東西都有使用者定義的類別和介面：

```python
    Graph g = new ConcreteGraph();
    Node indexRstNode = new StringNode("index.rst");
    Node indexHtmlNode = new StringNode("index.html");
    Edge indexEdge = new DirectedEdge(indexRstNode, indexHtmlNode);
    g.addEdge(indexEdge);
```

> The Python language and community explicitly and intentionally emphasize using simple, generic data structures to solve problems, instead of creating custom classes for every minute detail of the problem we want to tackle. This is one facet of the notion of “Pythonic” solutions: Pythonic solutions try to minimize syntactic overhead and leverage Python's powerful built-in tools and extensive standard library.

Python 語言和社群都明確且刻意地強調去使用簡單而通用的資料結構來解決問題，而不是對於每個我們處理的微小問題都去創造客製化的類別。這個 “Pythonic” 解法的概念，其中一個方面就是： Pythonic 解法會試著去最小化語法上的開銷，還有利用 Python 強大的內建工具及涵蓋許多範圍的標準函式庫。

> With these considerations in mind, let’s return to the `Graph` class, examining its design and implementation to see the interplay between data structures and class interfaces. When a new `Graph` instance is constructed, a pair of dictionaries has already been built to store edges using the logic we outlined in the previous section:

考慮到以上這些，讓我們回到 `Graph` 類別來檢驗它的設計和實作，當一個新的 `Graph` 實例被建構出來之後，就也會建出一組字典，用前面段落列出來的邏輯來儲存邊：

```python
class Graph:
    """A directed graph of the relationships among build tasks."""

    def __init__(self):
        self._inputs_of = defaultdict(set)
        self._consequences_of = defaultdict(set)
```

> The leading underscore in front of the attribute names `_inputs_of` and `_consequences_of` is a common convention in the Python community to signal that an attribute is private. This convention is one way the community suggests that programmers pass messages and warnings through space and time to each other. Recognizing the need to signal differences between public and internal object attributes, the community adopted the single leading underscore as a concise and fairly consistent indicator to other programmers, including our future selves, that the attribute is best treated as part of the invisible internal machinery of the class.

在屬性名稱之前的前下底線，像是 `_inputs_of` 和 `_consequences_of`，在 Python 社群中普遍習慣用來標示一個私有屬性。這個習慣是社群建議程式設計者們傳遞訊息和警告給彼此的其中一種方式。Python 社群認知到有必要去標示公開和內部物件屬性的不同，因此採用了前下底線作為一個簡潔且一致的指標來提醒其他的程式設計者，包含未來的自己，該屬性最好被當作類別中看不到的內部系統。

> Why are we using a `defaultdict` instead of a standard dict? A common problem when composing dicts with other data structures is handling missing keys. With a normal dict, retrieving a key that does not exist raises a `KeyError`:

為什麼我們會用 `defaultdict` 而不是標準 dict 呢？當在用其他的資料結構組成 dict 的時候，常見的問題就是要處理消失的鍵值。使用一般 dict 的時候，如果去存取一個不存在的鍵值會觸發一個 `KeyError`：

```python
>>> consequences_of = {}
>>> consequences_of['index.rst'].add('index.html')
Traceback (most recent call last):
     ...
KeyError: 'index.rst'
```

> Using a normal dict requires special checks throughout the code to handle this specific case, for example when adding a new edge:

使用一般 dict 的話，在整個程式碼都會需要特別的確認以處理這個特別的情況，舉例來說，當增加一個新的邊時：

```python
    # Special case to handle “we have not seen this task yet”:

    if input_task not in self._consequences_of:
        self._consequences_of[input_task] = set()

    self._consequences_of[input_task].add(consequence_task)
```

> This need is so common that Python includes a special utility, the `defaultdict`, which lets you provide a function that returns a value for absent keys. When we ask about an edge that the `Graph` hasn't yet seen, we will get back an empty `set` instead of an exception:

這個需求太普遍以致於 Python 包含了一個特別的工具程式： `defaultdict`，它讓你可以提供一個對於不存在的鍵值會回傳數值的函式。當我們去存取一個 `Graph` 還沒有出現的邊時，我們會拿到一個空的 `set` ，而不是一個例外：

```python
>>> from collections import defaultdict
>>> consequences_of = defaultdict(set)
>>> consequences_of['api.rst']
set()
```

> Structuring our implementation this way means that each key’s first use can look identical to second and subsequent times that a particular key is used:

用這樣的方式來建構我們的實作內容代表說每一個鍵值在第一次、第二次和接下來的每一次使用時，都會看起來相同：

```python
>>> consequences_of['index.rst'].add('index.html')
>>> 'index.html' in consequences_of['index.rst']
True
```

> Given these techniques, let’s examine the implementation of `add_edge`, which we earlier used to build the graph for [Figure 4.1](#figure1).

有了這些技巧，讓我們來檢驗 `add_edge` 的實作內容，我們之前有用來建構[圖 4.1](#figure1) 的圖。

```python
    def add_edge(self, input_task, consequence_task):
        """Add an edge: `consequence_task` uses the output of `input_task`."""
        self._consequences_of[input_task].add(consequence_task)
        self._inputs_of[consequence_task].add(input_task)
```

> This method hides the fact that two, not one, storage steps are required for each new edge so that we know about it in both directions. And notice how `add_edge()` does not know or care whether either node has been seen before. Because the inputs and consequences data structures are each a `defaultdict(set)`, the `add_edge()` method remains blissfully ignorant as to the novelty of a node — the `defaultdict` takes care of the difference by creating a new `set` object on the fly. As we saw above, `add_edge()` would be three times longer had we not used `defaultdict`. More importantly, it would be more difficult to understand and reason about the resulting code. This implementation demonstrates a Pythonic approach to problems: simple, direct, and concise.

這個方法隱藏了事實上對於每個新的邊，是有兩個而不是一個儲存步驟，這樣我們才可以從兩個方向都知道這個邊。而且會發現 `add_edge()` 並不知道也不在意節點是不是已經被看到過了。因為輸入和輸出的資料結構都是一個 `defaultdict(set)`，這個 `add_edge()` 方法對於一個節點是不是新的可以一無所知，`defaultdict` 會藉由創造一個新的 `set` 物件來動態地處理其中的差異。就像我們前面看到的，如果我們沒有使用 `add_edge()` 的話，程式碼會變長三倍，更重要的是，會變得更難理解和解釋產生出來的程式碼。這樣的實作展示了一個處理問題的 Pythonic 的方法：簡單、直接、明確。


> Callers should also be given a simple way to visit every edge without having to learn how to traverse our data structure:

呼叫者也應該有一個簡單的方法去訪問每個邊，而不是必須要學習如何去遍歷我們的資料結構：

```python
    def edges(self):
        """Return all edges as ``(input_task, consequence_task)`` tuples."""
        return [(a, b) for a in self.sorted(self._consequences_of)
                       for b in self.sorted(self._consequences_of[a])]
```

> The `Graph.sorted()` method makes an attempt to sort the nodes in a natural sort order (such as alphabetical) that can provide a stable output order for the user.

這個 `Graph.sorted()` 方法是嘗試以自然順序（像是依字母順序）來排序，以提供一個穩定的輸出順序給使用者。

> By using this traversal method we can see that, following our three “add” method calls earlier, `g` now represents the same graph that we saw in [Figure 4.1](#figure1).

藉由使用這樣的遍歷方法可以看出在我們先前呼叫了三次 “add” 方法之後，`g` 現在就跟 [Figure 4.1](#figure1) 一樣了。

```python
>>> from pprint import pprint
>>> pprint(g.edges())
[('api.rst', 'api.html'),
 ('index.rst', 'index.html'),
 ('tutorial.rst', 'tutorial.html')]
```

> Since we now have a real live Python object, and not just a figure, we can ask it interesting questions! For example, when Contingent is building a blog from source files, it will need to know things like “What depends on `api.rst`?” when the content of `api.rst` changes:

由於我們擁有了一個真實的 Python 物件，而不只是一張圖而已，我們可以問它一些令人感興趣的問題！例如，當 Contingent 正在從原始檔案建構一個部落格時，它將會需要知道一些事情，像是“什麼是依賴於 `api.rst`？” 當 `api.rst` 的內容改變的時候：

```python
>>> g.immediate_consequences_of('api.rst')
['api.html']
```

> This `Graph` is telling Contingent that, when `api.rst` changes, `api.html` is now stale and must be rebuilt.

`Graph` 就是在告訴 Contingent 說，當 `api.rst` 有更動的時候， `api.html` 就過時了，必須被重新建構。

> How about `index.html`?

那 `index.html` 呢？

```python
>>> g.immediate_consequences_of('index.html')
[]
```

> An empty list has been returned, signalling that `index.html` is at the right edge of the graph and so nothing further needs to be rebuilt if it changes. This query can be expressed very simply thanks to the work that has already gone in to laying out our data:

回傳了一個空的 list，代表說 `index.html` 是在圖的右側邊緣，因此當它改變時，沒有東西需要被重新建構。由於之前在安排資料時所下的功夫，這個查詢可以很簡單地被表示：

```python
    def immediate_consequences_of(self, task):
        """Return the tasks that use `task` as an input."""
        return self.sorted(self._consequences_of[task])
```

```python
>>> from contingent.rendering import as_graphviz
>>> open('figure1.dot', 'w').write(as_graphviz(g)) and None
```

> [Figure 4.1](#figure1) ignored one of the most important relationships that we discovered in the opening section of our chapter: the way that document titles appear in the table of contents. Let’s fill in this detail. We will create a node for each title string that needs to be generated by parsing an input file and then passed to one of our other routines:

[圖4.1](#figure1)忽略了其中一個我們在本章節一開始的段落中發現最重要的關係：文檔的標題會出現在目錄。讓我們補上這個細節，對於每個標題的字串，我們會創造一個節點，它要在解析一個輸入檔後才被產生，然後會被傳給我們其他的工作：

```python
>>> g.add_edge('api.rst', 'api-title')
>>> g.add_edge('api-title', 'index.html')
>>> g.add_edge('tutorial.rst', 'tutorial-title')
>>> g.add_edge('tutorial-title', 'index.html')
```

> The result is a graph ([Figure 4.2](#figure2)) that could properly handle rebuilding the table of contents that we discussed in the opening of this chapter.

結果會產生一張圖（[圖4.2](#figure2)），它可以適當地處理目錄的重新建構，如同我們在本章節一開始談到的那樣。

![figure2](http://www.aosabook.org/en/500L/contingent-images/figure3.png)

> <small>Figure 4.2 - Being prepared to rebuild `index.html` whenever any title that it mentions gets changed.</small><a id="figure2"></a>

<small>圖4.2 —— 當 `index.html` 中提到的任何標題被改動時，準備好重建它 </small>

> This manual walk-through illustrates what we will eventually have Contingent do for us: the graph `g` captures the inputs and consequences for the various artifacts in our project's documentation.

這個手動的演練展示了我們最終希望 Contingent 幫我們做的事情：圖 `g` 描繪了我們專案文件中各種工件的輸入和輸出。

## Learning Connections（學習連結）

> We now have a way for Contingent to keep track of tasks and the relationships between them. If we look more closely at [Figure 4.2](#figure2), however, we see that it is actually a little hand-wavy and vague: <em>how</em> is `api.html` produced from `api.rst`? How do we know that `index.html` needs the title from the tutorial? And how is this dependency resolved?

我們現在已經有了一個方法讓 Contingent 可以追蹤任務（task），還有它們之間的關係。然而，如果我們更仔細地審視 [圖4.2](#figure2)，我們會發現它其實有一點模糊：`api.html` 是**如何**由 `api.rst` 製造出來的呢？我們怎麼知道 `index.html` 會需要 tutorial 的標題？而這些依存關係是怎麼解決的？

> Our intuitive notion of these ideas served when we were constructing consequences graphs by hand, but unfortunately computers are not terribly intuitive, so we will need to be more precise about what we want.

當我們在手工建造結果圖時，我們對這些想法的直覺概念起了作用，不幸的是電腦並沒有這麼直覺，所以我們需要更精確來達到我們想要的結果。

> What are the steps required to produce output from sources? How are these steps defined and executed? And how can Contingent know the connections between them?

從原始檔案製造成輸出需要哪些步驟呢？這些步驟要如何定義和執行呢？而 Contingent 又要如何知道它們之間的關係？

> In Contingent, build tasks are modeled as functions plus arguments. The functions define actions that a particular project understands how to perform. The arguments provide the specifics: <em>which</em> source document should be read, <em>which</em> blog title is needed. As they are running, these functions may in turn invoke <em>other</em> task functions, passing whatever arguments they need answers for.

在 Contingent 中，建構的任務們會被建模為函式加上參數，這些函式定義了一些動作，特定的專案會知道該怎麼做。參數們提供了具體的細節：哪一個原始文檔應該被讀取？會需要哪一個部落格標題？當這些函式在執行時，它們可能會觸發其他的任務函式，以解答它們需要知道的參數。

> To see how this works, we will actually now implement the documentation builder described at the beginning of the chapter. In order to prevent ourselves from wallowing around in a bog of details, for this illustration we will work with simplified input and output document formats. Our input documents will consist of a title on the first line, with the remainder of the text forming the body. Cross references will simply be source file names enclosed in backticks, which on output are replaced with the title from the corresponding document in the output.

為了看看這是如何運作的，我們現在會實做一個在本章節一開始提到的文檔建構器。為了防止我們在細節的泥沼中打滾，在這個例子中我們會使用簡化的輸入和輸出文檔格式，我們的輸入文檔會包含第一行的標題和剩下文字所組成的正文。交叉引用將只是一個被反引號包起來的來源檔案名稱，輸出中的來源檔案名稱將替換為輸出中相應文檔的標題。

> Here is the content of our example `index.txt`, `api.txt`, and `tutorial.txt`, illustrating titles, document bodies, and cross-references from our little document format:

這裡是我們範例中 `index.txt`、`api.txt` 和 `tutorial.txt` 的內容，從我們的小文檔格式說明標題、文檔正文和交叉引用：

```python
>>> index = """
... Table of Contents
... -----------------
... * `tutorial.txt`
... * `api.txt`
... """

>>> tutorial = """
... Beginners Tutorial
... ------------------
... Welcome to the tutorial!
... We hope you enjoy it.
... """

>>> api = """
... API Reference
... -------------
... You might want to read
... the `tutorial.txt` first.
... """
```

> Now that we have some source material to work with, what functions would a Contingent-based blog builder need?

現在我們有了要處理的來源材料，對一個以 Contingent 為基礎的部落格建構器來說，會需要什麼函式呢？

> In the simple examples above, the HTML output files proceed directly from the source, but in a realistic system, turning source into markup involves several steps: reading the raw text from disk, parsing the text to a convenient internal representation, processing any directives the author may have specified, resolving cross-references or other external dependencies (such as include files), and applying one or more view transformations to convert the internal representation to its output form.

在以上簡單的範例中，HTML 的輸出文件們是直接由原始檔案而來，但在實際的系統中，由原始檔案轉換成 markup 會牽涉到幾個步驟：由磁碟讀取原始文本、將文本解析為方便的內部表示、解析交叉引用或其他外部的依賴（例如 include 檔案），還有應用一個或多個視圖轉換，來將內部的表示轉換成其輸出形式。

> Contingent manages tasks by grouping them into a `Project`, a sort of build system busybody that injects itself into the middle of the build process, noting every time one task talks to another to construct a graph of the relationships between all the tasks.

Contingent 藉由把任務們分組成 `Project` 來管理它們。`Project` 某種程度上是建構系統中的好事者，它會把自己注入到建構程序其中，且注意每次一個任務與另一個任務交談，以構建所有任務之間關係的圖形。

```python
>>> from contingent.projectlib import Project, Task
>>> project = Project()
>>> task = project.task
```

> A build system for the example given at the beginning of the chapter might involve a few tasks.

本章節一開始提到的範例建構系統可能會牽涉到幾個任務。

> Our `read()` task will pretend to read the files from disk. Since we really defined the source text in variables, all it needs to do is convert from a filename to the corresponding text.

我們的 `read()` 任務會假裝去從磁碟中讀取檔案。由於我們其實是把原始文本定義在變數中，所需要做的只是從檔案名稱轉換成相對應的文本。

```python
  >>> filesystem = {'index.txt': index,
  ...               'tutorial.txt': tutorial,
  ...               'api.txt': api}
  ...
  >>> @task
  ... def read(filename):
  ...     return filesystem[filename]
```

> The `parse()` task interprets the raw text of the file contents according to the specification of our document format. Our format is very simple: the title of the document appears on the first line, and the rest of the content is considered the document's body.

`parse()` 任務會根據文檔格式的規範去解釋文件內容的原始文本。我們的格式非常簡單：文檔的標題顯示在第一行，其餘內容被視為文檔的正文。

```python
  >>> @task
  ... def parse(filename):
  ...     lines = read(filename).strip().splitlines()
  ...     title = lines[0]
  ...     body = '\n'.join(lines[2:])
  ...     return title, body
```

> Because the format is so simple, the parser is a little silly, but it illustrates the interpretive responsibilities that parsers are required to carry out. (Parsing in general is a very interesting subject and many books have been written either partially or completely about it.) In a system like Sphinx, the parser must understand the many markup tokens, directives, and commands defined by the system, transforming the input text into something the rest of the system can work with.

因為格式實在太簡單了，解析器顯得有點愚蠢，然而它說明了解析器需要執行的解釋責任。（解析通常是一個非常有趣的主題，很多書都是部分或完全以它為主題寫成的。）在像 Sphinx 這樣的系統中，解析器必須理解系統定義的許多標記符號、指令和命令，將輸入的文本轉換為系統其餘部分可以使用的內容。

> Notice the connection point between `parse()` and `read()` — the first task in parsing is to pass the filename it has been given to `read()`, which finds and returns the contents of that file.

注意 `parse()` 和 `read()` 的連結點 —— 解析的第一個任務就是將取得的檔案名稱傳給 `read()`，它會找到且回傳該檔案的內容。

> The `title_of()` task, given a source file name, returns the document's title:

`title_of()` 任務會被給予一個原始檔案名稱，它會回傳該文檔的標題：

```python
  >>> @task
  ... def title_of(filename):
  ...     title, body = parse(filename)
  ...     return title
```

> This task nicely illustrates the separation of responsibilities between the parts of a document processing system. The `title_of()` function works directly from an in-memory representation of a document — in this case, a tuple — instead of taking it upon itself to re-parse the entire document again just to find the title. The `parse()` function alone produces the in-memory representation, in accordance with the contract of the system specification, and the rest of the blog builder processing functions like `title_of()` simply use its output as their authority.

此任務很好地說明了文檔處理系統各部分之間的職責分離。`title_of()` 函數直接從文檔的內存表示——在這種情況下是一個 tuple——而不是自己重新解析整個文檔只是為了找到標題。根據系統規範的契約，`parse（）` 函數會單獨產生內存中的表示，而其他部落格建構器的處理函數如 `title_of()` 只是使用它的輸出作為其權限。

> If you are coming from an orthodox object-oriented tradition, this function-oriented design may look a little weird. In an OO solution, `parse()` would return some sort of `Document` object that has `title_of()` as a method or property. In fact, Sphinx works exactly this way: its `Parser` subsystem produces a “Docutils document tree” object for the other parts of the system to use.

如果你來自一個正統物件導向的傳統，這種功能導向的設計可能看起來有點奇怪。在物件導向的解決方案中，`parse()` 將返回某種 `Document` 物件，該物件會有 `title_of()` 的方法或屬性。事實上，Sphinx 就是這樣運作的：它的 `Parser` 子系統為系統的其他部份生成了一個 “Docutils document tree” 物件。

> Contingent is not opinionated with regard to these differing design paradigms and supports either approach equally well. For this chapter we are keeping things simple.

對於這些不同的設計範例，Contingent 並不是自以為是，並且同樣支持這兩種方法。在本章中，我們保持簡單。

> The final task, `render()`, turns the in-memory representation of a document into an output form. It is, in effect, the inverse of `parse()`. Whereas `parse()` takes an input document conforming to a specification and converts it to an in-memory representation, `render()` takes an in-memory representation and produces an output document conforming to some specification.

最後一個任務 `render()` 將文檔的內存表示形式轉換為輸出形式。它實際上是 `parse()` 的反轉。`parse()` 接受符合規範的輸入文檔並將其轉換為內存中的表示，`render()` 接受內存中的表示並生成符合某些規範的輸出文檔。

```python
  >>> import re
  >>>
  >>> LINK = '<a href="{}">{}</a>'
  >>> PAGE = '<h1>{}</h1>\n<p>\n{}\n<p>'
  >>>
  >>> def make_link(match):
  ...     filename = match.group(1)
  ...     return LINK.format(filename, title_of(filename))
  ...
  >>> @task
  ... def render(filename):
  ...     title, body = parse(filename)
  ...     body = re.sub(r'`([^`]+)`', make_link, body)
  ...     return PAGE.format(title, body)
```

> Here is an example run that will invoke every stage of the above logic — rendering `tutorial.txt` to produce its output:

這裏示範一次執行，它會觸發以上邏輯的每一個階段 —— render `tutorial.txt` 來製造它的輸出：

```python
>>> print(render('tutorial.txt'))
<h1>Beginners Tutorial</h1>
<p>
Welcome to the tutorial!
We hope you enjoy it.
<p>
```

> [Figure 4.3](#figure3) illustrates the task graph that transitively connects all the tasks required to produce the output, from reading the input file, to parsing and transforming the document, and rendering it:

[圖4.3](#figure3)說明連接了生成輸出所需的全部任務的任務圖，從讀取輸入文件到解析和轉換文檔，以及呈現它：

![figure3](http://www.aosabook.org/en/500L/contingent-images/figure3.png)

<small>Figure 4.3 - A task graph.</small><a id="figure3"></a>
<small>圖4.3 —— 任務圖.</small>

> It turns out that [Figure 4.3](#figure3) was not hand-drawn for this chapter, but has been generated directly from Contingent! Building this graph is possible for the `Project` object because it maintains its own call stack, similar to the stack of live execution frames that Python maintains to remember which function to continue running when the current one returns.

[圖4.3](#figure3)不是為了本章節的手繪圖，而是直接由 Contingent 生成的！`Project` 物件可以構建此圖，因為它會維護自己的調用堆棧，類似於 Python 維護的即時執行幀堆疊，用來記住當前返回時要繼續運行的函數。

> Every time a new task is invoked, Contingent can assume that it has been called — and that its output will be used — by the task currently at the top of the stack. Maintaining the stack will require that several extra steps surround the invocation of a task <em>T</em>:

每次調用新任務時，Contingent 都可以假定它已被當前位於堆疊頂部的任務調用 —— 且其輸出將被使用。維護堆棧將需要幾個圍繞調用任務*T*的額外步驟：

><li>Push <em>T</em> onto the stack.</li>
><li>Execute <em>T</em>, letting it call any other tasks it needs.</li>
><li>Pop <em>T</em> off the stack.</li>
><li>Return its result.</li>

* 把*T*推上堆疊
* 執行*T*，讓它呼叫它需要的其他任何任務
* 從推疊取出*T*
* 返回它的結果

> To intercept task calls, the `Project` leverages a key Python feature: <em>function decorators</em>. A&nbsp;decorator is allowed to process or transform a function at the moment that it is being defined. The `Project.task` decorator uses this opportunity to package every task inside another function, a <em>wrapper</em>, which allows a clean separation of responsibilities between the wrapper — which will worry about graph and stack management on behalf of the Project — and our task functions that focus on document processing. Here is what the `task` decorator boilerplate looks like:

為了攔截任務的調用，`Project` 利用了一個關鍵的 Python 特性：*函數裝飾器*。裝飾器被允許在定義函數時處理或轉換函數。 `Project.task` 裝飾器利用這個機會將每個任務打包到另一個函數中，也就是一個*包裝器*，它允許在包裝器之間清楚地分離責任——這將代表專案中圖形和堆棧管理的擔憂——以及專注於文檔處理的任務功能。這是 `task` 裝飾器樣板的樣子：

```python
        from functools import wraps

        def task(function):
            @wraps(function)
            def wrapper(*args):
                # wrapper body, that will call function()
            return wrapper
```

> This is an entirely typical Python decorator declaration. It can then be applied to a function by naming it after an `@` character atop the `def` that creates the function:

這是一個完全典型的 Python 裝飾器聲明。接著可以透過在創建函數的 `def` 上面加上 `@` 字符及裝飾器的名稱來應用它：

```python
    @task
    def title_of(filename):
        title, body = parse(filename)
        return title
```

> When this definition is complete, the name `title_of` will refer to the wrapped version of the function. The wrapper can access the original version of the function via the name `function`, calling it at the appropriate time. The body of the Contingent wrapper runs something like this:

當定義完成時，`title_of` 這個名稱將引用該函數的包裝版本。包裝器可以通過名稱 `function` 訪問函數的原始版本，並在適當的時候調用它。 Contingent 包裝器的主體運行如下：

```python
    def task(function):
        @wraps(function)
        def wrapper(*args):
            task = Task(wrapper, args)
            if self.task_stack:
                self._graph.add_edge(task, self.task_stack[-1])
            self._graph.clear_inputs_of(task)
            self._task_stack.append(task)
            try:
                value = function(*args)
            finally:
                self._task_stack.pop()

            return value
        return wrapper
```

> This wrapper performs several crucial maintenance steps:

這個包裝器執行了幾個關鍵的維護步驟：

> 1. Packages the task — a function plus its arguments — into a small object for convenience. The `wrapper` here names the wrapped version of the task function.
> 2. If this task has been invoked by a current task that is already underway, add an edge capturing the fact that this task is an input to the already-running task.
> 3. Forget whatever we might have learned last time about the task, since it might make new decisions this time — if the source text of the API guide no longer mentions the Tutorial, for example, then its `render()` will no longer ask for the `title_of()` the Tutorial document.
> 4. Push this task onto the top of the task stack in case it decides, in its turn, to invoke further tasks in the course of doing its work.
> 5. Invoke the task inside of a `try...finally` block that ensures we correctly remove the finished task from the stack, even if it dies by raising an exception.
> 6. Return the task’s return value, so that callers of this wrapper will not be able to tell that they have not simply invoked the plain task function itself.

1.為方便起見，將任務——函數及其參數——打包到一個小物件中。這裡以 `wrapper` 命名任務函數的包裝版本。
2.如果此任務已由當前正在進行的任務調用，請添加一個邊緣，來獲取此任務是某個已經在運行的任務之輸入這一事實。
3.忘記我們上次可能學到的有關任務的內容，因為這次可能會做出新的決定——例如，如果API指南的原始文本不再提及 Tutorial，那麼它的 `render()` 將不會再要求 Tutorial 文檔 的 `title_of()`。
4.將此任務推送到任務堆疊的頂部，以防它決定在執行其工作的過程中調用其他任務。
5.在 `try ... finally` 區塊中調用任務，確保我們正確地從堆疊中移除已完成的任務，即使它因為丟出例外而死亡。
6.返回任務的返回值，這樣這個包裝器的調用者就無法說他們不只是單純地調用普通任務函數本身。


> Steps 4 and 5 maintain the task stack itself, which is then used by step 2 to perform the consequences tracking that is our whole reason for building a task stack in the first place.

步驟4和5是維護任務堆疊本身，然後在步驟2中被用來執行後結果追蹤，這是我們一開始建構任務堆疊的全部原因。

> Since each task gets surrounded by its own copy of the wrapper function, the mere invocation and execution of the normal stack of tasks will produce a graph of relationships as an invisible side effect. That is why we were careful to use the wrapper around each processing step that we defined:

由於每個任務都被自己的包裝函數副本包圍，僅僅調用和執行正常的任務堆疊將產生一個關係圖，這是一種無形的副作用。這就是為什麼我們要小心地使用我們定義的每個處理步驟的包裝器：

```python
    @task
    def read(filename):
        # body of read

    @task
    def parse(filename):
        # body of parse

    @task
    def title_of(filename):
        # body of title_of

    @task
    def render(filename):
        # body of render
```

> Thanks to these wrappers, when we called `parse('tutorial.txt')` the decorator learned the connection between `parse` and `read`. We can ask about the relationship by building another `Task` tuple and asking what the consequences would be if its output value changed:

幸好有了這些包裝器，當我們調用 `parse('tutorial.txt')` 時，裝飾器會知道 `parse` 和 `read` 之間的連結。我們可以藉由建構另一個 `Task` tuple 來詢問關係，還有詢問如果輸出值發生變化會產生什麼後果：

```python
>>> task = Task(read, ('tutorial.txt',))
>>> print(task)
read('tutorial.txt')
>>> project._graph.immediate_consequences_of(task)
[parse('tutorial.txt')]
```

> The consequence of re-reading the `tutorial.txt` file and finding that its contents have changed is that we need to re-execute the `parse()` routine for that document. What happens if we render the entire set of documents? Will Contingent be able to learn the entire build process?

重新讀取 `tutorial.txt` 檔案且發現它的內容有更動的話，我們會需要對該文檔重新執行 `parse()` 的動作。如果我們 render 整套文件會怎樣？ Contingent 可以知道整個建構過程嗎？

```python
>>> for filename in 'index.txt', 'tutorial.txt', 'api.txt':
...     print(render(filename))
...     print('=' * 30)
...
<h1>Table of Contents</h1>
<p>
* <a href="tutorial.txt">Beginners Tutorial</a>
* <a href="api.txt">API Reference</a>
<p>
==============================
<h1>Beginners Tutorial</h1>
<p>
Welcome to the tutorial!
We hope you enjoy it.
<p>
==============================
<h1>API Reference</h1>
<p>
You might want to read
the <a href="tutorial.txt">Beginners Tutorial</a> first.
<p>
==============================
```

> It worked! From the output, we can see that our transform substituted the document titles for the directives in our source documents, indicating that Contingent was able to discover the connections between the various tasks needed to build our documents.

真的有效！從輸出中，我們可以看到我們的轉換取代了原始文檔中指令的文檔標題，表明 Contingent 能夠發現建構文檔所需的各種任務之間的聯繫。

![figure4](http://www.aosabook.org/en/500L/contingent-images/figure4.png)

<small>Figure 4.4 - The complete set of relationships between our input files and our HTML outputs.</small>
<a id="figure4"></a>
<small>圖4.4 —— 我們的輸入檔案及 HTML 輸出的完整關係</small>


> By watching one task invoke another through the `task` wrapper machinery, `Project` has automatically learned the graph of inputs and consequences. Since it has a complete consequences graph at its disposal, Contingent knows all the things to rebuild if the inputs to any tasks change.

藉由觀察一個任務透過 `task` 包裝機器調用另一個任務，`Project` 自動習得了輸入和結果圖。由於它有一個完整的結果圖，如果有任何任務的輸入發生變化，Contingent 會知道所有要重建的事情。

## Chasing Consequences（追逐結果）

> Once the initial build has run to completion, Contingent needs to monitor the input files for changes. When the user finishes a new edit and runs “Save”, both the `read()` method and its consequences need to be invoked.

一旦初始的建構完成了，Contingent 會需要監控輸入的檔案是否有變化。當使用者完成一次新的編輯且執行 ”儲存“，`read()` 方法和它的輸出結果們都必須被觸發。

> This will require us to walk the graph in the opposite order from the one in which it was created. It was built, you will recall, by calling `render()` for the API Reference and having that call `parse()` which finally invoked the `read()` task. Now we go in the other direction: we know that `read()` will now return new content, and we need to figure out what consequences lie downstream.

這會需要我們以創建它的相反順序來遍歷圖形。你應該記得它是透過 API Reference 調用 `render()`，再由 `render()` 調用 `parse()`，`parse()` 最終調用 `read()` 任務而建構的。現在我們從另一個方向走：我們知道 `read（）`現在將返回新的內容，而我們需要弄清楚下游的結果。

> The process of compiling consequences is a recursive one, as each consequence can itself have further tasks that depended on it. We could perform this recursion manually through repeated calls to the graph. (Note that we are here taking advantage of the fact that the Python prompt saves the last value displayed under the name `_` for use in the subsequent expression.)

編譯出結果的過程是遞歸的，因為每個結果本身都有依賴於它的進一步任務。我們可以通過重複調用圖來手動執行此遞歸。 （請注意，我們在這裡利用了 Python 會把前一個顯示的值儲存名為 `_`， 以便在後續表達式中使用。）

```python
>>> task = Task(read, ('api.txt',))
>>> project._graph.immediate_consequences_of(task)
[parse('api.txt')]
>>> t1, = _
>>> project._graph.immediate_consequences_of(t1)
[render('api.txt'), title_of('api.txt')]
>>> t2, t3 = _
>>> project._graph.immediate_consequences_of(t2)
[]
>>> project._graph.immediate_consequences_of(t3)
[render('index.txt')]
>>> t4, = _
>>> project._graph.immediate_consequences_of(t4)
[]
```

> This recursive task of looking repeatedly for immediate consequences and only stopping when we arrive at tasks with no further consequences is a basic enough graph operation that it is supported directly by a method on the `Graph` class:

這個遞歸的任務是反複查看直接的結果（immediate consequences），並且只在我們走到沒有進一步結果的任務時停止，這是一個基本的圖形操作，它直接由 `Graph` 類別中的方法支持：

```python
>>> # Secretly adjust pprint to a narrower-than-usual width:
>>> _pprint = pprint
>>> pprint = lambda x: _pprint(x, width=40)
>>> pprint(project._graph.recursive_consequences_of([task]))
[parse('api.txt'),
 render('api.txt'),
 title_of('api.txt'),
 render('index.txt')]
```

> In fact, `recursive_consequences_of()` tries to be a bit clever. If a particular task appears repeatedly as a downstream consequence of several other tasks, then it is careful to only mention it once in the output list, and to move it close to the end so that it appears only after the tasks that are its inputs. This intelligence is powered by the classic depth-first implementation of a topological sort, an algorithm which winds up being fairly easy to write in Python through a hidden recursive helper function. Check out the `graphlib.py` source code for the details.

事實上，`recursive_consequences_of()` 有試著更聰明一些。如果某個特定任務重複出現於其他任務的下游結果，那麼它只需要在輸出列表中提及一次，並將其移動到接近末端，以便它只在它的輸入任務之後出現。這個聰明的想法是來自於拓撲排序中的經典深度優先算法，這種算法透過隱藏的遞歸輔助函數，很容易在 Python 中編寫。詳細信息可以查看 `graphlib.py` 原始碼。

> If, upon detecting a change, we are careful to re-run every task in the recursive consequences, then Contingent will be able to avoid rebuilding too little. Our second challenge, however, was to avoid rebuilding too much. Refer again to [Figure 4.4](#figure4). We want to avoid rebuilding all three documents every time that `tutorial.txt` is changed, since most edits will probably not affect its title but only its body. How can this be accomplished?

如果在檢測到更改後，我們會小心地在遞歸結果中重新運行每個任務，那麼 Contingent 也就能夠避免重建太少的問題。然而，我們的第二個挑戰是避免重建太多。再次參考 [圖4.4](#figure4)。我們希望每次更改 `tutorial.txt` 時避免都要重建所有三個文檔，因為大多數編輯可能不會影響它的標題，只會影響它的正文。如何實現這一目標呢？

> The solution is to make graph recomputation dependent on caching. When stepping forward through the recursive consequences of a change, we will only invoke tasks whose inputs are different than last time.

解決方案是利用緩存來重新計算圖表。當逐步推進改動的遞歸結果時，我們只會調用輸入與上次不同的任務。

> This optimization will involve a final data structure. We will give the `Project` a `_todo` set with which to remember every task for which at least one input value has changed, and which therefore requires re-execution. Because only tasks in `_todo` are out-of-date, the build process can skip running any tasks unless they appear there.

此優化會牽涉到最後的一個數據結構。我們將給 `project` 一個 `_todo` 的 set，用來記住至少一個輸入值已經改變的每個任務，因此它們會需要重新執行。因為只有 `_todo` 中的任務是過時的，所以建構過程可以跳過執行任何任務，除非它們出現在 `_todo` 裡面。

> Again, Python’s convenient and unified design makes these features very easy to code. Because task objects are hashable, `_todo` can simply be a set that remembers task items by identity — guaranteeing that a task never appears twice — and the `_cache` of return values from previous runs can be a dict with tasks as keys.

再一次的，Python 的方便且統一的設計使這些功能非常容易編碼。因為任務對像是可清除的，所以 `_todo` 可以簡單地設計成一個通過身份記住任務項的 set，set 保證任務永遠不會出現兩次，而先前運行的返回值的 `_cache` 可以是以任務作為鍵值的 dict。

> More precisely, the rebuild step must keep looping as long as `_todo` is non-empty. During each loop, it should:

更精確來說，只要 `_todo` 不為空，重新建構的步驟必須一直迴圈。在每一次迴圈中，它應該要：

> * Call `recursive_consequences_of()` and pass in every task listed in `_todo`. The return value will be a list of not only the `_todo` tasks themselves, but also every task downstream of them — every task, in other words, that could possibly need re-execution if the outputs come out different this time.
> * For each task in the list, check whether it is listed in `_todo`. If not, then we can skip running it, because none of the tasks that we have re-invoked upstream of it has produced a new return value that would require the task’s recomputation.
> * But for any task that is indeed listed in `_todo` by the time we reach it, we need to ask it to re-run and re-compute its return value. If the task wrapper function detects that this return value does not match the old cached value, then its downstream tasks will be automatically added to `_todo` before we reach them in the list of recursive consequences.

* 調用 `recursive_consequences_of()`，並傳入 `_todo` 中列出的每個任務。返回值不僅是 `_todo` 任務本身的列表，而且還列出了它們下游的每個任務——每個任務，換句話說，如果這次輸出不同，可能需要重新執行。
* 對於列表中的每個任務，檢查它是否列在 `_todo` 中。如果沒有，那麼我們可以跳過它，因為我們在其上游重新調用的任務都沒有產生一個新的返回值，會需要該任務重新計算。
* 但是對於任何在我們到達時確實列在 `_todo` 中的任務，我們需要讓它重新運行並重新計算其返回值。如果任務包裝器函數檢測到此返回值與舊的緩存值不匹配，那在我們於遞歸結果列表中到達它們之前，其下游任務將自動添加到  `_todo`。


> By the time we reach the end of the list, every task that could possibly need to be re-run should in fact have been re-run. But just in case, we will check `_todo` and try again if it is not yet empty. Even for very rapidly changing dependency trees, this should quickly settle out. Only a cycle — where, for example, task <em>A</em> needs the output of task <em>B</em> which itself needs the output of task <em>A</em> — could keep the builder in an infinite loop, and only if their return values never stabilize. Fortunately, real-world build tasks are typically without cycles.

> 當我們到達列表的末端時，所有可能需要重新執行的每個任務實際上都應該重新執行。但為了以防萬一，如果 `_todo` 不是空的，我們將檢查 `_todo` 並再次嘗試。即使對於非常快速改變依賴的樹，這也應該很快就會解決。只有一個循環時，例如，任務*A*需要任務*B*的輸出，而任務*B*本身需要任務*A*的輸出，會讓建構器陷入無限循環，而且只有當它們的返回值永遠不會穩定時才會發生。幸運的是，真實世界的建構任務通常沒有周期。

> Let us trace the behavior of this system through an example.

讓我們透過一個例子來追踪這個系統的行為。

> Suppose you edit `tutorial.txt` and change both the title and the body content. We can simulate this by modifying the value in our `filesystem` dict:

假設你編輯了 `tutorial.txt` 並更改標題和正文的內容。我們可以通過修改 `filesystem` dict 中的值來模擬這個情況：

```python
>>> filesystem['tutorial.txt'] = """
... The Coder Tutorial
... ------------------
... This is a new and improved
... introductory paragraph.
... """
```

> Now that the contents have changed, we can ask the Project to re-run the `read()` task by using its `cache_off()` context manager that temporarily disables its willingness to return its old cached result for a given task and argument:

現在文件內容已經改變了，我們可以讓 Project 重新執行 `read()` 任務，藉由使用它的 `cache_off()` 上下文管理器，`cache_off()` 會暫時不回傳給定的任務和參數的舊緩存結果：

```python
>>> with project.cache_off():
...     text = read('tutorial.txt')
```

> The new tutorial text has now been read into the cache. How many downstream tasks will need to be re-executed?

現在新的 tutorial 文件已經讀入緩存中。多少個下游任務會需要重新被執行？

> To help us answer this question, the `Project` class supports a simple tracing facility that will tell us which tasks are executed in the course of a rebuild. Since the above change to `tutorial.txt` affects both its body and its title, everything downstream will need to be re-computed:

為了幫助我們回答這個問題，`Project` 類別支持一個簡單的追踪工具，它將告訴我們在重建過程中執行哪些任務。由於上面對 `tutorial.txt` 的更改會影響其內文和標題，因此需要重新計算下游的所有內容：

```python
>>> project.start_tracing()
>>> project.rebuild()
>>> print(project.stop_tracing())
calling parse('tutorial.txt')
calling render('tutorial.txt')
calling title_of('tutorial.txt')
calling render('api.txt')
calling render('index.txt')
```

> Looking back at [Figure 4.4](#figure4), you can see that, as expected, this is every task that is an immediate or downstream consequence of `read('tutorial.txt')`.

回過頭去看 [圖4.4](#figure4)，你可以發現，正如預期的那樣，這是 `read('tutorial.txt')` 直接或下游的結果。

> But what if we edit it again, but this time leave the title the same?

但如果我們再次編輯它，不過這一次讓標題保持不變，會怎麼樣？

```python
>>> filesystem['tutorial.txt'] = """
... The Coder Tutorial
... ------------------
... Welcome to the coder tutorial!
... It should be read top to bottom.
... """
>>> with project.cache_off():
...     text = read('tutorial.txt')
```

> This small, limited change should have no effect on the other documents.

這種有限的小改變應該對其他文件沒有影響。

```python
>>> project.start_tracing()
>>> project.rebuild()
>>> print(project.stop_tracing())
calling parse('tutorial.txt')
calling render('tutorial.txt')
calling title_of('tutorial.txt')
```

> Success! Only one document got rebuilt. The fact that `title_of()`, given a new input document, nevertheless returned the same value, means that all further downstream tasks were insulated from the change and did not get re-invoked.

成功！只重建了一個文檔。給定一個新的輸入文檔，然而 `title_of()` 是返回相同的值，這意味著所有進一步的下游任務都與更動無關，並且沒有被重新調用。


## Conclusion（結論）

> There exist languages and programming methodologies under which Contingent would be a suffocating forest of tiny classes, with verbose names given to every concept in the problem domain.

在許多語言和編程方法之中，Contingent 會是一個令人窒息的微小類別森林，它在問題域中給出了每個概念的詳細名稱。

> When programming Contingent in Python, however, we skipped the creation of a dozen possible classes like `TaskArgument` and `CachedResult` and `ConsequenceList`. We instead drew upon Python’s strong tradition of solving generic problems with generic data structures, resulting in code that repeatedly uses a small set of ideas from the core data structures tuple, list, set, and dict.

然而，在 Python 中編寫 Contingent 時，我們跳過了創建十幾個可能的類，如 `TaskArgument` 和 `CachedResult` 以及 `ConsequenceList`。我們反而利用了 Python 通用數據結構解決泛型問題的強大傳統，導致代碼重複使用來自核心數據結構元組，列表，集合和字典的一小部分想法。

> But does this not cause a problem?

但這不會導致問題嗎？

> Generic data structures are also, by their nature, anonymous. Our `project._cache` is a set. So is every collection of upstream and downstream nodes inside the `Graph`. Are we in danger of seeing generic `set` error messages and not knowing whether to look in the project or the graph implementation for the error?

通用數據結構本質上也是匿名的。我們的 `project._cache` 是一個集合。因此 `Graph` 中的每個上游和下游節點集合也是如此。我們是否有看到通用 `set` 錯誤消息的危險，而不知道是否要查看項目或圖表實現中的錯誤？

> In fact, we are not in danger!

事實上，我們沒有危險！

> Thanks to the careful discipline of encapsulation — of only allowing `Graph` code to touch the graph’s sets, and `Project` code to touch the project’s set — there will never be ambiguity if a set operation returns an error during a later phase of the project. The name of the innermost executing method at the moment of the error will necessarily direct us to exactly the class, and set, involved in the mistake. There is no need to create a subclass of `set` for every possible application of the data type, so long as we put that conventional underscore in front of data structure attributes and then are careful not to touch them from code outside of the class.

由於封裝的謹慎規則 - 只允許 `Graph` 代碼觸摸圖形集，而 `Project` 代碼觸摸項目集 - 如果 set 操作在後續階段返回錯誤，則永遠不會有歧義。項目。錯誤時刻最內層執行方法的名稱必然會將我們引導到完全類，並設置，參與錯誤。沒有必要為數據類型的每個可能的應用程序創建 `set` 的子類，只要我們將傳統的下劃線放在數據結構屬性的前面，然後小心不要從類外的代碼中觸摸它們。

> Contingent demonstrates how crucial the Facade pattern, from the epochal <em>Design Patterns</em> book, is for a well-designed Python program. Not every data structure and fragment of data in a Python program gets to be its own class. Instead, classes are used sparingly, at conceptual pivots in the code where a big idea — like the idea of a dependency graph — can be wrapped up into a Facade that hides the details of the simple generic data structures that lie beneath it.

Contingent 展示了來自 epochal **Design Patterns** 一書的 Facade 模式對於精心設計的 Python 程序的重要性。並非 Python 程序中的每個數據結構和數據片段都是它自己的類。相反，在代碼中的概念樞紐中謹慎使用類，其中一個重要的想法 - 如依賴圖的概念 - 可以被包含在 Facade 中，該 Facade 隱藏了位於其下的簡單通用數據結構的細節。

> Code outside of the Facade names the big concepts that it needs and the operations that it wants to perform. Inside of the Facade, the programmer manipulates the small and convenient moving parts of the Python programming language to make the operations happen.

Facade 之外的代碼命名了它需要的大概念以及它想要執行的操作。在 Facade 內部，程序員操縱 Python 編程語言的小而方便的移動部分以使操作發生。
