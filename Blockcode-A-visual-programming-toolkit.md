# Blockcode: 一個視覺化程式開發工具(Blockcode: A visual programming toolkit    by:Dethe Elza)

[原文出處](http://aosabook.org/en/500L/blockcode-a-visual-programming-toolkit.html)

translated by<`RanceJen`>

**OnGoing**

> Dethe is a geek dad, aesthetic programmer, mentor, and creator of the Waterbear visual programming tool. He co-hosts the Vancouver Maker Education Salons and wants to fill the world with robotic origami rabbits.

Dethe 是一個 geek 父親、藝術性的 programmer、導師以及程式設計工具 Waterbear 的創造者。他協辦了溫哥華 Maker 教育機構以希望能讓世界上充滿紙兔機器人。

> In block-based programming languages, you write programs by dragging and connecting blocks that represent parts of the program. Block-based languages differ from conventional programming languages, in which you type words and symbols.
>
> Learning a programming language can be difficult because they are extremely sensitive to even the slightest of typos. Most programming languages are case-sensitive, have obscure syntax, and will refuse to run if you get so much as a semicolon in the wrong place—or worse, leave one out. Further, most programming languages in use today are based on English and their syntax cannot be localized.

學習一個程式語言有時是一件困難的事，因為他們對於拼寫錯誤非常敏感。大部分的程式語言會有區分大小寫，模糊的語法，甚至連如果您在錯誤的地方寫上分號他們都會有拒絕運行，甚至在缺少分號時會更糟。更進一步來說，目前大部分的程式語言都是基於英文而且他們的語法並不能被本地化。

> 註：他應該是指缺分號時很多語言常常會吐出亂七八糟的 Error message

> In contrast, a well-done block language can eliminate syntax errors completely. You can still create a program which does the wrong thing, but you cannot create one with the wrong syntax: the blocks just won't fit that way. Block languages are more discoverable: you can see all the constructs and libraries of the language right in the list of blocks. Further, blocks can be localized into any human language without changing the meaning of the programming language.

相較起來，一個優質的方塊式語言可以完全消除語法錯誤，您依然可以創造一個「邏輯錯誤的程式」，但您沒辦法創造一個語法錯誤的程式，因為方塊之間就是無法那樣組合。方塊是的語言是更有可見性的。您可以在方塊列表中看見所有程式的結構跟函式庫。更甚之方塊式可以在不改變程式語言本身含意的情況下被融入任何人類語言。

![](http://aosabook.org/en/500L/blockcode-images/blockcode_ide.png)
Figure 1.1 - The Blockcode IDE in use

> Block-based languages have a long history, with some of the prominent ones being Lego Mindstorms, Alice3D, StarLogo, and especially Scratch. There are several tools for block-based programming on the web as well: Blockly, AppInventor, Tynker, and many more.
>
>The code in this chapter is loosely based on the open-source project Waterbear, which is not a language but a tool for wrapping existing languages with a block-based syntax. Advantages of such a wrapper include the ones noted above: eliminating syntax errors, visual display of available components, ease of localization. Additionally, visual code can sometimes be easier to read and debug, and blocks can be used by pre-typing children. (We could even go further and put icons on the blocks, either in conjunction with the text names or instead of them, to allow pre-literate children to write programs, but we don't go that far in this example.)

方塊式的程式語言已經有很長的歷史，其中特別突出的像是 Lego Mindstorms、Alice3D、StarLogo、尤其是 Scratch.也有一些在網路上提供的方塊式程式設計工具像 Blockly、AppInventor、Tynker 以及其他更多。

本章節中的程式大致上是基於開源專案 Waterbear，他並不是一個語言而是一個工具可以將現有的語言包裝成方塊式，這樣做的優點包含上面已提過的：視覺化顯示可用的元件，以及方塊可以被用於不會打字的孩童，(我們甚至可以更進一步將圖示放在方塊上，用來和文字一起使用或是替代文字，用來讓不識字的孩童嘗試程式設計，不過在這個範例中我們沒有發展到這麼遠)

> The choice of turtle graphics for this language goes back to the Logo language, which was created specifically to teach programming to children. Several of the block-based languages above include turtle graphics, and it is a small enough domain to be able to capture in a tightly constrained project such as this.
>
> If you would like to get a feel for what a block-based-language is like, you can experiment with the program that is built in this chapter from author's GitHub repository.

會為這個語言選擇 turtle graphics 的原因可以追朔到 Logo 語言，Logo 語言是專門為孩童教學程式語言所設計的，一部分的方塊式語言都有包含 turtle graphics ，它夠小以至於能被這種架構受限的專案實現。

如果您想嘗試一下何為方塊式語言，您可以實驗一下本章節的程式，它的專案建置在作者之 github repository 中。

> 註：turtle graphics 指的是上方圖示中的放射狀圖形

### Goals and Structure (目標與架構)

> I want to accomplish a couple of things with this code. First and foremost, I want to implement a block language for turtle graphics, with which you can write code to create images through simple dragging-and-dropping of blocks, using as simple a structure of HTML, CSS, and JavaScript as possible. Second, but still important, I want to show how the blocks themselves can serve as a framework for other languages besides our mini turtle language.

我希望能用這些程式達成一些事，首先最重要的是我希望能為 turtle graphics 實作一個方塊式語言，這樣您只要透過簡單的拖拉就可以設計程式來創造圖片，用盡可能簡單的 HTML、CSS、以及 JavaScript，同樣重要的第二點是我希望能展現除了我們的迷你 turtle 語言之外，方塊式語言能如何作為一個其他語言的 framework。

> To do this, we encapsulate everything that is specific to the turtle language into one file (turtle.js) that we can easily swap with another file. Nothing else should be specific to the turtle language; the rest should just be about handling the blocks (blocks.js and menu.js) or be generally useful web utilities (util.js, drag.js, file.js). That is the goal, although to maintain the small size of the project, some of those utilities are less general-purpose and more specific to their use with the blocks.

要達成這個目標，我們要封裝所有 turtle language 特有的事物到一個我們可以輕易置換的檔案 (turtle.js)，並且不應有任何 turtle language 特有的被留下，剩下的都應該只是處理方塊或是通用 web 功能的(util.js, drag.js, file.js)，這樣就能達到我們的目標。不過為了保持專案的簡潔性，有一部份的工具是比較不通用而為方塊特製的。

> One thing that struck me when writing a block language was that the language is its own IDE. You can't just code up blocks in your favourite text editor; the IDE has to be designed and developed in parallel with the block language. This has some pros and cons. On the plus side, everyone will use a consistent environment and there is no room for religious wars about what editor to use. On the downside, it can be a huge distraction from building the block language itself.

在我設計方塊式語言時有一件困擾我的事就是它的 IDE ，人們沒辦法在他喜好的文字編輯器上撰寫方塊是語言，該語言的 IDE 必須是與方塊是語言同時設計和開發的。這其實有利有弊，在好的一面來看，每個人都用相同的環境開發避免了爭論該用哪個編輯器的宗教戰爭，在壞的一面就是他會大幅分散設計方塊式語言的設計者開發時的心力。

> 註：IDE 為 Integrated Development Environment ，中文稱「整合開發環境」，就是像 visual studio 之類的整合編輯器。

> 註2：所謂「設計方塊式語言的設計者」是指「實作出方塊式語言的人」，而非用方塊式語言來設計程式的使用者。

### The Nature of Scripts (指令的本質)

> A Blockcode script, like a script in any language (whether block- or text-based), is a sequence of operations to be followed. In the case of Blockcode the script consists of HTML elements which are iterated over, and which are each associated with a particular JavaScript function which will be run when that block's turn comes. Some blocks can contain (and are responsible for running) other blocks, and some blocks can contain numeric arguments which are passed to the functions.

Blockcode 的指令就像是任何（不管是方塊式還是文字式）語言的指令一樣，是一些可運算操作的組合，而在 Blockcode 之中的指令則是由迭代的 html 元素組成，他們與特定的 JavaScript 相關聯且在迭代到該方塊時執行，有些方塊可以包含其他的方塊（並且也負責執行他們），有些方塊可以把參數傳遞到實際執行的函式內。
 
> 註：這邊 script 的意指「一個可運算的指令」，像是 `rm -rf /` 。

> 註2：可以包含其他方塊的方塊聽起來有點怪，但其實就是類似 for 這種東西，指令包指令，是不是很合理容易理解惹 ヽ(●´∀`●)ﾉ

> In most (text-based) languages, a script goes through several stages: a lexer converts the text into recognized tokens, a parser organizes the tokens into an abstract syntax tree, then depending on the language the program may be compiled into machine code or fed into an interpreter. That's a simplification; there can be more steps. For Blockcode, the layout of the blocks in the script area already represents our abstract syntax tree, so we don't have to go through the lexing and parsing stages. We use the Visitor pattern to iterate over those blocks and call predefined JavaScript functions associated with each block to run the program.

在大部份文字式語言，一個指令會經過幾種階段，一個語句分析器將文字轉換為已知的代碼，一個解析器再將代碼組合成一個抽象的語法樹，接著根據語言可能會編譯成機械碼或是再丟到其他的轉譯器內。這還只是簡化的說法，實際上可能還會有更多步驟要做。對於 Blockcode 來說，方塊在指令版面上的配置其實已經代表了抽象的語法樹，所以語句分析器跟解析器的步驟我們其實可以跳過。我們用訪問者模式的設計去迭代這些方塊並且呼叫每個方塊預先對應好的 JavaScript 函式來執行這個程式。

> There is nothing stopping us from adding additional stages to be more like a traditional language. Instead of simply calling associated JavaScript functions, we could replace turtle.js with a block language that emits byte codes for a different virtual machine, or even C++ code for a compiler. Block languages exist (as part of the Waterbear project) for generating Java robotics code, for programming Arduino, and for scripting Minecraft running on Raspberry Pi.

我們也可以輕易的追加不同的處理階段使其更像傳統語言，除了單純呼叫 JavaScript 對應的函式外，我們也可以替換 turtle.js 成產生對應不同虛擬機 byte code ，甚至是生成 C++ 的程式碼給編譯器。方塊式語言(在 Waterbear 有一部份)就是為了產生 Java 的機器人程式碼，來給 Arduino 的程式開發，或是在樹梅派上運行 Minecraft 的指令。

### Web Applications (網頁應用端)

> In order to make the tool available to the widest possible audience, it is web-native. It's written in HTML, CSS, and JavaScript, so it should work in most browsers and platforms.
>
> Modern web browsers are powerful platforms, with a rich set of tools for building great apps. If something about the implementation became too complex, I took that as a sign that I wasn't doing it "the web way" and, where possible, tried to re-think how to better use the browser tools.

為了最大化這個工具對任何潛在受眾的可用性，它會是純網頁原生的，用 HTML、CSS 和 JavaScript 寫成，所以應該在絕大部分的平台跟瀏覽器上都可以使用。

現代網頁瀏覽器是非常強大的平台，有豐富的工具可以用來建構優質的應用。如果今天實作的某些部份變得過於複雜，這就是個信號「我並不是用做網頁的方法在實作」。這種情況下如果可以的話，試著重新思考如何更好的使用瀏覽器工具。

> An important difference between web applications and traditional desktop or server applications is the lack of a main() or other entry point. There is no explicit run loop because that is already built into the browser and implicit on every web page. All our code will be parsed and executed on load, at which point we can register for events we are interested in for interacting with the user. After the first run, all further interaction with our code will be through callbacks we set up and register, whether we register those for events (like mouse movement), timeouts (fired with the periodicity we specify), or frame handlers (called for each screen redraw, generally 60 frames per second). The browser does not expose full-featured threads either (only shared-nothing web workers).

網頁應用程式和傳統桌面端或伺服器端應用之間的一個重要差別在於缺少 main() 或是其他的程式進入點，他並不會明確的「執行」，因為該流程已經被內建在瀏覽器並隱含在每個頁面中，在載入時就被解析並且執行。而在這個時間點我們就可以註冊我們希望能與使用者互動的事件，在第一輪執行過後，接下來的與我們設計之程式的互動都會用透過我們設定及註冊的 callback 來完成，不管是註冊像是滑鼠移動，計時器或畫格處理，瀏覽器處理時也不會完整的暴露該功能的 thread。（只使用不共享的網頁工作器）

### Stepping Through the Code(逐步了解程式碼)

> I've tried to follow some conventions and best practices throughout this project. Each JavaScript file is wrapped in a function to avoid leaking variables into the global environment. If it needs to expose variables to other files it will define a single global per file, based on the filename, with the exposed functions in it. This will be near the end of the file, followed by any event handlers set by that file, so you can always glance at the end of a file to see what events it handles and what functions it exposes.

在這個專案中我有盡量去遵循一些便利且優質的作法，每個 JavaScript 的檔案都封裝在一個函式中來避免洩漏任何變數在全域環境中，如果任何檔案有需要將某些數值曝露給其他檔案，則用檔案名稱定義一個 global，所有要曝露的函式都在其中。這些都放在接近檔案的結尾處，後面會接著所有設定之事件處理器。所以你都可以一看檔案結尾就知道他處理了哪些事件和公開了哪些函式。

> 註：假設 menu 需要給 itme 給別人則定義 `global.Menu[item]` 這種做法。

> The code style is procedural, not object-oriented or functional. We could do the same things in any of these paradigms, but that would require more setup code and wrappers to impose on what exists already for the DOM. Recent work on Custom Elements make it easier to work with the DOM in an OO way, and there has been a lot of great writing on Functional JavaScript, but either would require a bit of shoe-horning, so it felt simpler to keep it procedural.

這是程序式程式設計的 code style，而非物件導向或是函數式。這些的程式設計風格都可以做到一樣的事，不過可能會需要更多設定用程式碼和封裝來加在 DOM 已存在的東西之上，雖然最近有一個 Custom Elements 的功能可以輕易將 DOM 用物件導向的方式操作，在函數式 JavaScript 上也有很多優秀的寫法，但作者覺得這樣都有點卡腳，所以用程序式的方式去寫感覺是最簡單的。

> There are eight source files in this project, but index.html and blocks.css are basic structure and style for the app and won't be discussed. Two of the JavaScript files won't be discussed in any detail either: util.js contains some helpers and serves as a bridge between different browser implementations—similar to a library like jQuery but in less than 50 lines of code. file.js is a similar utility used for loading and saving files and serializing scripts.

在這個專案中總共有八個原始檔，其中 `index.html` 和 `blocks.css` 是這個應用的基本結構和形式，這我們就不做討論了。還有兩個 JavaScript 的檔案我們也不會討論細節，其一是 `util.js` ，它包含了一些橋接不同瀏覽器實作的輔助和服務，是一個類似 jQuery 的函式庫但只有不到 50 行程式碼。`file.js` 則是一個類似前者的工具，它用
來格式化指令、儲存或載入檔案。

> Script Serialization 的意義為將指令轉換為可儲存或是取用的格式，詳細參見 [wiki](https://zh.wikipedia.org/wiki/%E5%BA%8F%E5%88%97%E5%8C%96)

> These are the remaining files:
> * block.js is the abstract representation of a block-based language.
> * drag.js implements the key interaction of the language: allowing the user to drag blocks from a list of available blocks (the "menu") to assemble them into a program (the "script").
> * menu.js has some helper code and is also responsible for actually running the user's program.
> * turtle.js defines the specifics of our block language (turtle graphics) and initializes its specific blocks. This is the file that would be replaced in order to create a different block language.

然後這些是剩下的檔案
* block.js 是一個方塊式語言的抽象實現。
    * 類似創建方塊、方塊本身、方塊內容、方塊執行......等抽象化的實現。
* drag.js 實現這個語言的關鍵互動：允許使用者從可用方塊列表(就是選單)中拖拉方塊去組合他們成一個程式(拖拉的方塊就是指令)。
* menu.js 有一些輔助用工具(用來取得 menuItem)並負責實際執行使用者設計的程式。
* turtle.js 定義我們的方塊式語言的細節(turtle graphics)，並初始化對應的方塊，這就是那個可以被取代來創造其他方塊式語言的檔案。

>`blocks.js`
>
> Each block consists of a few HTML elements, styled with CSS, with some JavaScript event handlers for dragging-and-dropping and modifying the input argument. The blocks.js file helps to create and manage these groupings of elements as single objects. When a type of block is added to the block menu, it is associated with a JavaScript function to implement the language, so each block in the script has to be able to find its associated function and call it when the script runs.
![](http://aosabook.org/en/500L/blockcode-images/block.png)
Figure 1.2 - An example block

每個方塊由一些 HTML 元素及CSS 組成，此外還有一些 JavaScript 事件處理器來負責拖拉和輸入參數的部份，`blocks.js`這個檔案會協助處理創建跟將這一組一組的元素作為一個物件來管理，當一個類型為方塊的被加入方塊選單時，它會被關聯到一個 JavaScript 的函式來實作這個語言，所以在指令中的每個方塊都可以找到其關聯的函式並在運行時被呼叫。

> Blocks have two optional bits of structure. They can have a single numeric parameter (with a default value), and they can be a container for other blocks. These are hard limits to work with, but would be relaxed in a larger system. In Waterbear there are also expression blocks which can be passed in as parameters; multiple parameters of a variety of types are supported. Here in the land of tight constraints we'll see what we can do with just one type of parameter.