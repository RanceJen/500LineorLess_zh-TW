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

> 譯註：他應該是指缺分號時很多語言常常會吐出亂七八糟的 Error message

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

> 譯註：turtle graphics 指的是上方圖示中的放射狀圖形

### Goals and Structure (目標與架構)

> I want to accomplish a couple of things with this code. First and foremost, I want to implement a block language for turtle graphics, with which you can write code to create images through simple dragging-and-dropping of blocks, using as simple a structure of HTML, CSS, and JavaScript as possible. Second, but still important, I want to show how the blocks themselves can serve as a framework for other languages besides our mini turtle language.

我希望能用這些程式達成一些事，首先最重要的是我希望能為 turtle graphics 實作一個方塊式語言，這樣您只要透過簡單的拖拉就可以設計程式來創造圖片，用盡可能簡單的 HTML、CSS、以及 JavaScript，同樣重要的第二點是我希望能展現除了我們的迷你 turtle 語言之外，方塊式語言能如何作為一個其他語言的 framework。

> To do this, we encapsulate everything that is specific to the turtle language into one file (turtle.js) that we can easily swap with another file. Nothing else should be specific to the turtle language; the rest should just be about handling the blocks (blocks.js and menu.js) or be generally useful web utilities (util.js, drag.js, file.js). That is the goal, although to maintain the small size of the project, some of those utilities are less general-purpose and more specific to their use with the blocks.

要達成這個目標，我們要封裝所有 turtle language 特有的事物到一個我們可以輕易置換的檔案 (turtle.js)，並且不應有任何 turtle language 特有的被留下，剩下的都應該只是處理方塊或是通用 web 功能的(util.js, drag.js, file.js)，這樣就能達到我們的目標。不過為了保持專案的簡潔性，有一部份的工具是比較不通用而為方塊特製的。

> One thing that struck me when writing a block language was that the language is its own IDE. You can't just code up blocks in your favourite text editor; the IDE has to be designed and developed in parallel with the block language. This has some pros and cons. On the plus side, everyone will use a consistent environment and there is no room for religious wars about what editor to use. On the downside, it can be a huge distraction from building the block language itself.

在我設計方塊式語言時有一件困擾我的事就是它的 IDE ，人們沒辦法在他喜好的文字編輯器上撰寫方塊是語言，該語言的 IDE 必須是與方塊是語言同時設計和開發的。這其實有利有弊，在好的一面來看，每個人都用相同的環境開發避免了爭論該用哪個編輯器的宗教戰爭，在壞的一面就是他會大幅分散設計方塊式語言的設計者開發時的心力。

> 譯註：IDE 為 Integrated Development Environment ，中文稱「整合開發環境」，就是像 visual studio 之類的整合編輯器。

> 譯註2：所謂「設計方塊式語言的設計者」是指「實作出方塊式語言的人」，而非用方塊式語言來設計程式的使用者。

### The Nature of Scripts (指令的本質)

> A Blockcode script, like a script in any language (whether block- or text-based), is a sequence of operations to be followed. In the case of Blockcode the script consists of HTML elements which are iterated over, and which are each associated with a particular JavaScript function which will be run when that block's turn comes. Some blocks can contain (and are responsible for running) other blocks, and some blocks can contain numeric arguments which are passed to the functions.

Blockcode 的指令就像是任何（不管是方塊式還是文字式）語言的指令一樣，是一些可運算操作的組合，而在 Blockcode 之中的指令則是由迭代的 html 元素組成，他們與特定的 JavaScript 相關聯且在迭代到該方塊時執行，有些方塊可以包含其他的方塊（並且也負責執行他們），有些方塊可以把參數傳遞到實際執行的函式內。
 
> 譯註：這邊 script 的意指「一個可運算的指令」，像是 `rm -rf /` 。

> 譯註2：可以包含其他方塊的方塊聽起來有點怪，但其實就是類似 for 這種東西，指令包指令，是不是很合理容易理解惹 ヽ(●´∀`●)ﾉ

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

> 譯註： callback 指事件對應的特定函數調用，中國大陸一般翻譯為「回調」，我總覺得這樣翻很怪故保持原文。

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

方塊的結構中有兩個選填的部份，他們可以有一個數字的參數(並有帶有預設值)，以及他們可以變成另外其他方塊的容器，這些是我們運作上的硬性限制，但這在大型的系統上會被解除掉，在 Waterbear 中就有表達式的方塊可以被當作參數傳入，並支援多種的參數型態，在這種較為緊縮的架構下，我們可以看到有什麼事是只要一種型態的參數就可以做到的。

```
<!-- The HTML structure of a block -->
<div class="block" draggable="true" data-name="Right">
    Right
    <input type="number" value="5">
    degrees
</div>
```

> It's important to note that there is no real distinction between blocks in the menu and blocks in the script. Dragging treats them slightly differently based on where they are being dragged from, and when we run a script it only looks at the blocks in the script area, but they are fundamentally the same structures, which means we can clone the blocks when dragging from the menu into the script.
>
> The createBlock(name, value, contents) function returns a block as a DOM element populated with all internal elements, ready to insert into the document. This can be used to create blocks for the menu, or for restoring script blocks saved in files or localStorage. While it is flexible this way, it is built specifically for the Blockcode "language" and makes assumptions about it, so if there is a value it assumes the value represents a numeric argument and creates an input of type "number". Since this is a limitation of the Blockcode, this is fine, but if we were to extend the blocks to support other types of arguments, or more than one argument, the code would have to change.

要提到有一點很重要的就是，其實選單上的方塊跟指令區的方塊實際上並沒有差別，拖曳會讓他們有一點不同在「從哪裡來」這點上，而當你運行指令時則只看指令區內的方塊，但從根本上來看他們都是一樣的結構，這代表當你從選單拖曳方塊時到指令區時可以複製方塊。

`createBlock(name,value,contents)` 這個函式回傳一個包含所有內部要素的 DOM 元素，並準備被插入這個文件，這可以用來為選單創造一個方塊，或是恢復儲存在檔案或是本地空間的指令方塊，雖然這種做法很彈性，但這是特別為 blockcode 這個語言假設並且建構的，所以這裡假設了值一定是由數字的形式作為參數，並且創建一個 "number" 型態的輸入。這就是 Blockcode 的限制，這是可以接受的，但如果要有擴展的方塊可支援其他型態的參數，或是多於一個的參數，則程式碼就要做改動。

```
    function createBlock(name, value, contents){
        var item = elem('div',
            {'class': 'block', draggable: true, 'data-name': name},
            [name]
        );
        if (value !== undefined && value !== null){
            item.appendChild(elem('input', {type: 'number', value: value}));
        }
        if (Array.isArray(contents)){
            item.appendChild(
                elem('div', {'class': 'container'}, contents.map(function(block){
                return createBlock.apply(null, block);
            })));
        }else if (typeof contents === 'string'){
            // Add units (degrees, etc.) specifier
            item.appendChild(document.createTextNode(' ' + contents));
        }
        return item;
    }
```

> We have some utilities for handling blocks as DOM elements:
> * `blockContents(block)` retrieves the child blocks of a container block. It always returns a list if called on a container block, and always returns null on a simple block
> * `blockValue(block)` returns the numerical value of the input on a block if the block has an input field of type number, or null if there is no input element for the block
> * `blockScript(block)` will return a structure suitable for serializing with JSON, to save blocks in a form they can easily be restored from
> * `runBlocks(blocks)` is a handler that runs each block in an array of blocks

我們有一些工具可將方塊作為 DOM 元素處理：
* `blockContents(block)` 檢索一個容器式方塊的子方塊，如果輸入的是一個容器式方塊則它回傳一個子方塊的列表，反之則回傳 null 。
* `blockValue(block)` 如果方塊有輸入區塊的話，以數字型態回傳輸入方塊內的值，反之則回傳 null 。
* `blockScript(block)` 回傳一個方塊結構用以序列化成 JSON ，用來格式化儲存方塊，同時也可以輕鬆的被還原。
* `runBlocks(blocks)` 是一個處理器用來執行 blocks 這個陣列中的各個方塊
```
function blockContents(block){
        var container = block.querySelector('.container');
        return container ? [].slice.call(container.children) : null;
    }

    function blockValue(block){
        var input = block.querySelector('input');
        return input ? Number(input.value) : null;
    }

    function blockUnits(block){
        if (block.children.length > 1 &&
            block.lastChild.nodeType === Node.TEXT_NODE &&
            block.lastChild.textContent){
            return block.lastChild.textContent.slice(1);
        }
    }

    function blockScript(block){
        var script = [block.dataset.name];
        var value = blockValue(block);
        if (value !== null){
            script.push(blockValue(block));
        }
        var contents = blockContents(block);
        var units = blockUnits(block);
        if (contents){script.push(contents.map(blockScript));}
        if (units){script.push(units);}
        return script.filter(function(notNull){ return notNull !== null; });
    }

    function runBlocks(blocks){
        blocks.forEach(function(block){ trigger('run', block); });
    }
```

> `drag.js`
> 
> The purpose of `drag.js` is to turn static blocks of HTML into a dynamic programming language by implementing interactions between the menu section of the view and the script section. The user builds their program by dragging blocks from the menu into the script, and the system runs the blocks in the script area.

`drag.js` 的目的在於將靜態的 HTML 方塊轉換成動態的程式設計語言，藉由實作選單區塊和指令區塊的視覺互動，使用者就可以從選單區拖曳方塊到指令區來建構他們的程式，更讓系統可以運行指令區的方塊。

> We're using HTML5 drag-and-drop; the specific JavaScript event handlers it requires are defined here. (For more information on using HTML5 drag-and-drop, see Eric Bidleman's article.) While it is nice to have built-in support for drag-and-drop, it does have some oddities and some pretty major limitations, like not being implemented in any mobile browser at the time of this writing.
>
> We define some variables at the top of the file. When we're dragging, we'll need to reference these from different stages of the dragging callback dance.

我們使用的是 HTML5 的拖放功能，指定的 JavaScript 事件處理器需要被定義在這(更多的 HTML5 拖放功能可見[Eric Bidleman 的主題說明](http://www.html5rocks.com/en/tutorials/dnd/basics/))。
雖然有內建的拖拉功能是一件好事，但這它同時也有一些奇怪且重要的限制，像是這功能「目前」並沒有在任何手機瀏覽器上被實作。(「目前」指 2016 年該作者寫文章的當下)

我們在檔案的頂部定義了一些數值，我們在拖曳的不同階段呼叫 callback 時會需要引用他們。
```
    var dragTarget = null; // Block we're dragging
    var dragType = null; // Are we dragging from the menu or from the script?
    var scriptBlocks = []; // Blocks in the script, sorted by position
```

> Depending on where the drag starts and ends, drop will have different effects:
>
> * If dragging from script to menu, delete `dragTarget` (remove block from script).
> * If dragging from script to script, move `dragTarget` (move an existing script block).
> * If dragging from menu to script, copy `dragTarget` (insert new block in script).
> * If dragging from menu to menu, do nothing.
>
> During the `dragStart(evt)` handler we start tracking whether the block is being copied from the menu or moved from (or within) the script. We also grab a list of all the blocks in the script which are not being dragged, to use later. The evt.dataTransfer.setData call is used for dragging between the browser and other applications (or the desktop), which we're not using, but have to call anyway to work around a bug.

有鑑於拖曳的起點終點不同，最終放下方塊時會有以下不同的效果。
* 如果從指令區拖曳到選單區，刪除 `dragTarget` (從指令區刪除該方塊)
* 如果從指令區拖曳到指令區，移動 `dragTarget` (移動已存在的方塊)
* 如果從選單區拖曳到指令區，複製 `dragTarget` (插入新方塊到指令區)
* 如果從選單區拖曳到選單區，啥都不做。

在 `dragStart(evt)` 這個處理器之中我們會追蹤不管是從選單區被複製或是從指令區被移動的方塊。我們同時也會抓取一個在指令區所有沒被拖曳之方塊的列表留待後續使用， `evt.dataTransfer.setData` 這個呼叫是用在瀏覽器與其他應用交互拖曳物件時，這部份我們沒有使用，但不管怎樣都要呼叫它來避免一個 bug。

> 譯註：bug 指程式漏洞或問題。
```
    function dragStart(evt){
        if (!matches(evt.target, '.block')) return;
        if (matches(evt.target, '.menu .block')){
            dragType = 'menu';
        }else{
            dragType = 'script';
        }
        evt.target.classList.add('dragging');
        dragTarget = evt.target;
        scriptBlocks = [].slice.call(
            document.querySelectorAll('.script .block:not(.dragging)'));
        // For dragging to take place in Firefox, we have to set this, even if
        // we don't use it
        evt.dataTransfer.setData('text/html', evt.target.outerHTML);
        if (matches(evt.target, '.menu .block')){
            evt.dataTransfer.effectAllowed = 'copy';
        }else{
            evt.dataTransfer.effectAllowed = 'move';
        }
    }
```

> While we are dragging, the `dragenter`, `dragover`, and `dragout` events give us opportunities to add visual cues by highlighting valid drop targets, etc. Of these, we only make use of `dragover`.

當我們在拖曳時，`dragenter`、`dragover` 和 `dragout` 事件讓我們有辦法做到「用視覺上的提示來突顯出我們放置的目標」......之類的功能，其中這我們只用到 `dragover` 的部份。
```
    function dragOver(evt){
        if (!matches(evt.target, '.menu, .menu *, .script, .script *, .content')) {
            return;
        }
        // Necessary. Allows us to drop.
        if (evt.preventDefault) { evt.preventDefault(); }
        if (dragType === 'menu'){
            // See the section on the DataTransfer object.
            evt.dataTransfer.dropEffect = 'copy';  
        }else{
            evt.dataTransfer.dropEffect = 'move';
        }
        return false;
    }
```

> When we release the mouse, we get a `drop` event. This is where the magic happens. We have to check where we dragged from (set back in `dragStart`) and where we have dragged to. Then we either copy the block, move the block, or delete the block as needed. We fire off some custom events using `trigger()` (defined in `util.js`) for our own use in the block logic, so we can refresh the script when it changes.

當我們放開拖曳的滑鼠按鍵時會取得一個放下事件，這就是魔法發生的時機點。我們要先去檢查方塊是從何處被拖曳過來的，然後放置在什麼區域。然後視需求看要複製、移動還是刪除這個方塊。藉由`trigger()`(定義在 `util.js`)來為了我們的方塊邏輯啟動一些客製化的事件，所以我們就可以當它改動時刷新指令的部份。
```
    function drop(evt){
        if (!matches(evt.target, '.menu, .menu *, .script, .script *')) return;
        var dropTarget = closest(
            evt.target, '.script .container, .script .block, .menu, .script');
        var dropType = 'script';
        if (matches(dropTarget, '.menu')){ dropType = 'menu'; }
        // stops the browser from redirecting.
        if (evt.stopPropagation) { evt.stopPropagation(); }
        if (dragType === 'script' && dropType === 'menu'){
            trigger('blockRemoved', dragTarget.parentElement, dragTarget);
            dragTarget.parentElement.removeChild(dragTarget);
        }else if (dragType ==='script' && dropType === 'script'){
            if (matches(dropTarget, '.block')){
                dropTarget.parentElement.insertBefore(
                    dragTarget, dropTarget.nextSibling);
            }else{
                dropTarget.insertBefore(dragTarget, dropTarget.firstChildElement);
            }
            trigger('blockMoved', dropTarget, dragTarget);
        }else if (dragType === 'menu' && dropType === 'script'){
            var newNode = dragTarget.cloneNode(true);
            newNode.classList.remove('dragging');
            if (matches(dropTarget, '.block')){
                dropTarget.parentElement.insertBefore(
                    newNode, dropTarget.nextSibling);
            }else{
                dropTarget.insertBefore(newNode, dropTarget.firstChildElement);
            }
            trigger('blockAdded', dropTarget, newNode);
        }
    }
```

> The `dragEnd(evt)` is called when we mouse up, but after we handle the `drop` event. This is where we can clean up, remove classes from elements, and reset things for the next drag.

而 `dragEnd(evt)` 會在滑鼠按鍵被放開的時候呼叫，但會在我們處理前敘的 `drop` 之後，這是我們用來清理的部份，將相關的類別移除(像是 'dragging' )，並重設各項東西來準備下一次的拖曳。
```
    function _findAndRemoveClass(klass){
        var elem = document.querySelector('.' + klass);
        if (elem){ elem.classList.remove(klass); }
    }

    function dragEnd(evt){
        _findAndRemoveClass('dragging');
        _findAndRemoveClass('over');
        _findAndRemoveClass('next');
    }
```

> `menu.js`
>
> The file `menu.js` is where blocks are associated with the functions that are called when they run, and contains the code for actually running the script as the user builds it up. Every time the script is modified, it is re-run automatically.
> 
> "Menu" in this context is not a drop-down (or pop-up) menu, like in most applications, but is the list of blocks you can choose for your script. This file sets that up, and starts the menu off with a looping block that is generally useful (and thus not part of the turtle language itself). This is kind of an odds-and-ends file, for things that may not fit anywhere else.

`menu.js` 這個檔案用來關聯方塊與其運行時呼叫函式的，並有著使用者實際建構時會運行的程式碼，每一次指令有更動時，它都會自動重新運行。

此上下文中的 "Menu" 並不是像大多數應用中戳一下就會彈出來的下拉式選單，而是一個用來讓使用者選擇指令的方塊列表。這個檔案用來設置好選單區並使它以一個通用的迴圈方塊作為開頭(由於迴圈的概念是通用的，所以他不是 turtle 語言的一部分)，這是一個比較東拼西湊的文件，用來整理一些放那都不太對的東西。

> Having a single file to gather random functions in is useful, especially when an architecture is under development. My theory of keeping a clean house is to have designated places for clutter, and that applies to building a program architecture too. One file or module becomes the catch-all for things that don't have a clear place to fit in yet. As this file grows it is important to watch for emerging patterns: several related functions can be spun off into a separate module (or joined together into a more general function). You don't want the catch-all to grow indefinitely, but only to be a temporary holding place until you figure out the right way to organize the code.

有一個檔案可以收集一些散亂的函式是很實用的，尤其是在開發中的架構上，我維持房子乾淨的理論就是要有一個特定的地點放雜物，這同時也可以應用在程式架構上。一個檔案或是模組當收集者來管理全部沒有明確歸屬的東西。當檔案不斷變大時，切記要注意他成長的方向：像是一些有相關性的函式可以被分離成一個模組(或是組合成一個更為通用的函式)，你並不會想要這個收集者無限增大，而應該只是在你組織好程式之前的一個臨時放置處。

> We keep around references to `menu` and `script` because we use them a lot; no point hunting through the DOM for them over and over. We'll also use `scriptRegistry`, where we store the scripts of blocks in the menu. We use a very simple name-to-script mapping which does not support either multiple menu blocks with the same name or renaming blocks. A more complex scripting environment would need something more robust.
> 
> We use `scriptDirty` to keep track of whether the script has been modified since the last time it was run, so we don't keep trying to run it constantly.

我們會保留 `menu` 和 `script` 兩者的引用是因為他們常常被使用，沒必要一次又一次的重新透過 DOM 搜索它們，我們還使用 `scriptRegistry` 來保存選單上方塊的實際指令，其會做簡單的名稱對應指令。而這樣做就不支援多個選單上的方塊擁有同一名稱或是重新命名方塊，在指令更為複雜的情況下我們會需要更具穩健性的處理方式。

我們使用 `scriptDirty` 來追蹤看至指令區在上次運行後有沒有改變，這樣我們就不會嘗試重複運行它。

>譯註：BlockCode 在拉方塊上去後會自動執行並秀出新的運行結果，每次改變時都會自動運行，所以才會有上面那句解釋。
```
    var menu = document.querySelector('.menu');
    var script = document.querySelector('.script');
    var scriptRegistry = {};
    var scriptDirty = false;
```

> When we want to notify the system to run the script during the next frame handler, we call `runSoon()` which sets the `scriptDirty ` flag to `true`. The system calls `run()` on every frame, but returns immediately unless `scriptDirty` is set. When `scriptDirty` is set, it runs all the script blocks, and also triggers events to let the specific language handle any tasks it needs before and after the script is run. This decouples the blocks-as-toolkit from the turtle language to make the blocks re-usable (or the language pluggable, depending how you look at it).

當我們想要通知系統在下一幀開始運行指令，會呼叫 `runSoon()` 來設置 `scriptDirty` 的標誌為 `true` 。 系統每一幀都會自動呼叫 `run()`，但在 `scriptDirty` 非 `true` 時會立即回傳，而當 `scriptDirty` 為 `true` 時，就執行所有的指令方塊，並觸發對應事件來讓特定的語言處理「指令運行前後」要做的事(像是編譯之類的)。這就將方塊本身從 turtle 語言中解耦出來成一種工具包，使得「方塊本身是可以被再利用」的(看你要從什麼角度來看，也可以說成是「可以被擴充在語言之上」的)。

> As part of running the script, we iterate over each block, calling `runEach(evt)` on it, which sets a class on the block, then finds and executes its associated function. If we slow things down, you should be able to watch the code execute as each block highlights to show when it is running.
>
> The `requestAnimationFrame` method below is provided by the browser for animation. It takes a function which will be called for the next frame to be rendered by the browser (at 60 frames per second) after the call is made. How many frames we actually get depends on how fast we can get work done in that call.

在運行指令的部份，我們會迭代所有方塊，呼叫其之 `runEach(evt)` ，它會在方塊上設置一個類別，然後找到並運行其對應的函式，如果我們把整件事的過程放慢，你就可以看到程式碼運行時會突顯該運行方塊。

下面的 `requestAnimationFrame` 這個是瀏覽器為動畫提供的方法，它呼叫後接收一個渲染下一幀時要被呼叫的函式給瀏覽器(通常是一秒 60 幀)，而實際幀數取決於我們的工作可以被執行的多快。
```
    function runSoon(){ scriptDirty = true; }

    function run(){
        if (scriptDirty){
            scriptDirty = false;
            Block.trigger('beforeRun', script);
            var blocks = [].slice.call(
                document.querySelectorAll('.script > .block'));
            Block.run(blocks);
            Block.trigger('afterRun', script);
        }else{
            Block.trigger('everyFrame', script);
        }
        requestAnimationFrame(run);
    }
    requestAnimationFrame(run);

    function runEach(evt){
        var elem = evt.target;
        if (!matches(elem, '.script .block')) return;
        if (elem.dataset.name === 'Define block') return;
        elem.classList.add('running');
        scriptRegistry[elem.dataset.name](elem);
        elem.classList.remove('running');
    }
```

> We add blocks to the menu using `menuItem(name, fn, value, contents)` which takes a normal block, associates it with a function, and puts in the menu column.

我們將方塊增加到選單時使用 `menuItem(name, fn, value, contents)` ，它會接收一個普通的方塊，將其與對應函式關聯起來，並放到選單區的欄位上。
```
    function menuItem(name, fn, value, units){
        var item = Block.create(name, value, units);
        scriptRegistry[name] = fn;
        menu.appendChild(item);
        return item;
    }
```

> We define `repeat(block)` here, outside of the turtle language, because it is generally useful in different languages. If we had blocks for conditionals and reading and writing variables they could also go here, or into a separate trans-language module, but Right now we only have one of these general-purpose blocks defined.

我們也將 `repeat(block)` 定義在這裡，而非 turtle 語言之中，因為 `repeat(block)` 是非常通用於所有語言的。如果我們有一些條件語句、讀寫值之類的也可以放在這，或是分離成一個轉換語言用的模組，但目前為止我們只有這一個通用於方塊上的方法要定義。
```
    function repeat(block){
        var count = Block.value(block);
        var children = Block.contents(block);
        for (var i = 0; i < count; i++){
            Block.run(children);
        }
    }
    menuItem('Repeat', repeat, 10, []);
```

> `turtle.js`
>
> `turtle.js` is the implementation of the turtle block language. It exposes no functions to the rest of the code, so nothing else can depend on it. This way we can swap out the one file to create a new block language and know nothing in the core will break.

`turtle.js` 是 turtle 方塊語言的實現，它本身並沒有向其他任何程式碼公開自己的函數，所以也不會有任何東西依賴於其，這樣我們才能替換掉這個檔案來創造新的方塊式語言而不會使核心的部份無法運作。

![](http://aosabook.org/en/500L/blockcode-images/turtle_example.png)
Figure 1.3 - Example of Turtle code running

> Turtle programming is a style of graphics programming, first popularized by Logo, where you have an imaginary turtle carrying a pen walking on the screen. You can tell the turtle to pick up the pen (stop drawing, but still move), put the pen down (leaving a line everywhere it goes), move forward a number of steps, or turn a number of degrees. Just those commands, combined with looping, can create amazingly intricate images.

Turtle programming 是一種圖形程式設計的方法，由 Logo 這個語言而開始被人們了解，你可以想像成一隻烏龜咬著筆在螢幕上行走。你可以告訴這個烏龜收起筆(停止劃線，但依然移動)，或是拿出筆(在行走得路徑上留下軌跡線)、要向前走幾步路、或是轉向幾度。只需要上敘指令配上迴圈，就可以畫出非常精美且錯綜複雜的圖形。

> In this version of turtle graphics we have a few extra blocks. Technically we don't need both `turn right` and `turn left` because you can have one and get the other with negative numbers. Likewise `move back` can be done with `move forward` and negative numbers. In this case it felt more balanced to have both.

在這個版本中我們其實有幾個多餘的方塊，像是技術上是不需要同時存在 `turn right` 和 `turn left` 的，因為對應的另外一個可以用負值來做到相同效果，相同的 `move back` 也可以用負值的 `move forward` 來做到。但我覺得在這邊這幾個種同時存在感覺比較平衡。

> The image above was formed by putting two loops inside another loop and adding a move forward and turn right to each loop, then playing with the parameters interactively until I liked the image that resulted.

上方的圖片是由一個迴圈內部在加上兩個向前並右轉的迴圈組成，並且調整參數運行到我覺得滿意為止所得到的圖案。
```
    var PIXEL_RATIO = window.devicePixelRatio || 1;
    var canvasPlaceholder = document.querySelector('.canvas-placeholder');
    var canvas = document.querySelector('.canvas');
    var script = document.querySelector('.script');
    var ctx = canvas.getContext('2d');
    var cos = Math.cos, sin = Math.sin, sqrt = Math.sqrt, PI = Math.PI;
    var DEGREE = PI / 180;
    var WIDTH, HEIGHT, position, direction, visible, pen, color;
```
> The `reset()` function clears all the state variables to their defaults. If we were to support multiple turtles, these variables would be encapsulated in an object. We also have a utility, `deg2rad(deg)`, because we work in degrees in the UI, but we draw in radians. Finally, `drawTurtle()` draws the turtle itself. The default turtle is simply a triangle, but you could override this to draw a more aesthetically-pleasing turtle.

`reset()` 函式會清除所有的狀態設定值為預設值，如果我們支援存在多個畫圖烏龜，則這些設定值會被封裝成一個物件。我們同時也有一個工具，`deg2rad(deg)` 能將角度轉換為弧度，因為我們畫圖時實際上用的是弧度。最後 `drawTurtle()` 繪畫出烏龜本身，預設的烏龜是個簡單的三角形，不過你可以嘗試覆寫它來畫出一個更美觀的烏龜。

> Note that `drawTurtle` uses the same primitive operations that we define to implement the turtle drawing. Sometimes you don't want to reuse code at different abstraction layers, but when the meaning is clear it can be a big win for code size and performance.

注意到 `drawTurtle` 其實是用我們先前定義的基本操作來實現，雖然有時候你並不想在不同的抽象層中混用程式碼，但當行為非常明確時這樣做對程式的大小跟效能都會有獲益。

```
    function reset(){
        recenter();
        direction = deg2rad(90); // facing "up"
        visible = true;
        pen = true; // when pen is true we draw, otherwise we move without drawing
        color = 'black';
    }

    function deg2rad(degrees){ return DEGREE * degrees; }

    function drawTurtle(){
        var userPen = pen; // save pen state
        if (visible){
            penUp(); _moveForward(5); penDown();
            _turn(-150); _moveForward(12);
            _turn(-120); _moveForward(12);
            _turn(-120); _moveForward(12);
            _turn(30);
            penUp(); _moveForward(-5);
            if (userPen){
                penDown(); // restore pen state
            }
        }
    }
```

> We have a special block to draw a circle with a given radius at the current mouse position. We special-case `drawCircle` because, while you can certainly draw a circle by repeating `MOVE 1 RIGHT 1` 360 times, controlling the size of the circle is very difficult that way.

我們有一個特殊的方塊來畫一個給定半徑的圓圈在當前指標的位置上，我們將 `drawCircle` 當成一個特殊案例是因為如果用  `MOVE 1 RIGHT 1` 重複 360 次的方式來畫圓，他的大小會非常難掌握。
```
    function drawCircle(radius){
        // Math for this is from http://www.mathopenref.com/polygonradius.html
        var userPen = pen; // save pen state
        if (visible){
            penUp(); _moveForward(-radius); penDown();
            _turn(-90);
            var steps = Math.min(Math.max(6, Math.floor(radius / 2)), 360);
            var theta = 360 / steps;
            var side = radius * 2 * Math.sin(Math.PI / steps);
            _moveForward(side / 2);
            for (var i = 1; i < steps; i++){
                _turn(theta); _moveForward(side);
            }
            _turn(theta); _moveForward(side / 2);
            _turn(90);
            penUp(); _moveForward(radius); penDown();
            if (userPen){
                penDown(); // restore pen state
            }
        }
    }
```

> Our main primitive is `moveForward`, which has to handle some elementary trigonometry and check whether the pen is up or down.

我們的主要的基礎操作是 `moveForward`，它的實現必須處理一些基礎的三角函數，並確認筆是提起還是放下的。

```
    function _moveForward(distance){
        var start = position;
        position = {
            x: cos(direction) * distance * PIXEL_RATIO + start.x,
            y: -sin(direction) * distance * PIXEL_RATIO + start.y
        };
        if (pen){
            ctx.lineStyle = color;
            ctx.beginPath();
            ctx.moveTo(start.x, start.y);
            ctx.lineTo(position.x, position.y);
            ctx.stroke();
        }
    }
```

> Most of the rest of the turtle commands can be easily defined in terms of what we've built above.

大部分剩下的是對烏龜的操作，可以根據我們上面建構的內容來輕鬆的定義。

```
    function penUp(){ pen = false; }
    function penDown(){ pen = true; }
    function hideTurtle(){ visible = false; }
    function showTurtle(){ visible = true; }
    function forward(block){ _moveForward(Block.value(block)); }
    function back(block){ _moveForward(-Block.value(block)); }
    function circle(block){ drawCircle(Block.value(block)); }
    function _turn(degrees){ direction += deg2rad(degrees); }
    function left(block){ _turn(Block.value(block)); }
    function right(block){ _turn(-Block.value(block)); }
    function recenter(){ position = {x: WIDTH/2, y: HEIGHT/2}; }
```

> When we want a fresh slate, the `clear` function restores everything back to where we started.

當我們想刷新畫板，`clear` 會還原所有東西到起始狀態。

```
    function clear(){
        ctx.save();
        ctx.fillStyle = 'white';
        ctx.fillRect(0,0,WIDTH,HEIGHT);
        ctx.restore();
        reset();
        ctx.moveTo(position.x, position.y);
    }
```

> When this script first loads and runs, we use our `reset` and `clear` to initialize everything and draw the turtle.

當這個指令被第一次載入跟執行的時候，我們使用 `reset` 和 `clear` 來初始化所有事物並畫出烏龜。

```
    onResize();
    clear();
    drawTurtle();
```

> Now we can use the functions above, with the `Menu.item` function from `menu.js`, to make blocks for the user to build scripts from. These are dragged into place to make the user's programs.

現在我們可以用這些函式，搭配來自 `menu.js` 的 `Menu.item` 來製作出方塊給使用者拖曳以建構程式用的指令方塊。

```
    Menu.item('Left', left, 5, 'degrees');
    Menu.item('Right', right, 5, 'degrees');
    Menu.item('Forward', forward, 10, 'steps');
    Menu.item('Back', back, 10, 'steps');
    Menu.item('Circle', circle, 20, 'radius');
    Menu.item('Pen up', penUp);
    Menu.item('Pen down', penDown);
    Menu.item('Back to center', recenter);
    Menu.item('Hide turtle', hideTurtle);
    Menu.item('Show turtle', showTurtle);
```

## Lessons Learned(學習課題)

### Why Not Use MVC?(為和不用 MVC 架構？)

> Model-View-Controller (MVC) was a good design choice for Smalltalk programs in the '80s and it can work in some variation or other for web apps, but it isn't the right tool for every problem. All the state (the "model" in MVC) is captured by the block elements in a block language anyway, so replicating it into Javascript has little benefit unless there is some other need for the model (if we were editing shared, distributed code, for instance).

模組-視圖-控制器(MVC) 是 80 年代用於短暫交互用應用程式的優質設計選擇，而其也有一些不同的改良為網頁應用之運用方式，但他並不適用於所有的程式上。在方塊式語言中所有的狀態 (也就是 MVC 中的 model) 都是由方塊元素組成，所以將 MVC 模式帶進來的效益很低，除非你還有其他模組的需求(像是如果要共同編輯同一份分散式的程式碼，那會需要同步的模組之類的)。

> An early version of Waterbear went to great lengths to keep the model in JavaScript and sync it with the DOM, until I noticed that more than half the code and 90% of the bugs were due to keeping the model in sync with the DOM. Eliminating the duplication allowed the code to be simpler and more robust, and with all the state on the DOM elements, many bugs could be found simply by looking at the DOM in the developer tools. So in this case there is little benefit to building further separation of MVC than we already have in HTML/CSS/JavaScript.

在早期的 Waterbear 架構成長的很龐大，就為了保持 JavaScript 中的模組與 DOM k
同步，直到我理解到超過一半的程式碼跟 90％ 以上的 bug 都來自模組跟 DOM 的同步上。消除這種複製並同步的機制後，反而讓程式碼更加的簡潔和強健。且在所有狀態都保持在 DOM 元素後，很多的 bug 看開發者工具中的 DOM 就可以找到。所以在這個案例中在HTML/CSS/JavaScript 的基礎上再把架構分割成 MVC 的效益不大。

### Toy Changes Can Lead to Real Changes(在玩具上的改變可以引導到實際上)

> Building a small, tightly scoped version of the larger system I work on has been an interesting exercise. Sometimes in a large system there are things you are hesitant to change because they affect too many other things. In a tiny, toy version you can experiment freely and learn things which you can then take back to the larger system. For me, the larger system is Waterbear and this project has had a huge impact on the way Waterbear is structured.

建構一個現有大型系統之架構小並且緊湊的版本是一個很有趣的實驗，有時後在大型系統中有些事物會讓你猶豫不決到底要不要改動，因為它們會一下影響到太多事物。但在一個小型的玩具版本中，你可以自由實驗並學習到可以帶回大型系統的經驗。之於我就像是 Waterbear 和本專案一樣，對於 Waterbear 目前建構的方向有非常大的影響。

#### Small Experiments Make Failure OK(對小實驗失敗是 OK 的)

> Some of the experiments I was able to do with this stripped-down block language were:
> * using HTML5 drag-and-drop,
> * running blocks directly by iterating through the DOM calling associated functions,
> * separating the code that runs cleanly from the HTML DOM,
> * simplified hit testing while dragging,
> * building our own tiny vector and sprite libraries (for the game blocks), and
> * "live coding" where the results are shown whenever you change the block script.

在這個小型的方塊式語言中，我希望實驗的是：
* 使用 HTML5 的拖放功能。
* 依據直接迭代 DOM 並呼叫關聯函式來運行方塊。
* 將運行的程式碼乾淨的從  HTML DOM 分離出來。
* 嘗試簡潔化拖拉時的命中判斷。
* 建構自有的小型向量(vector)及精靈函式庫(sprite)(為了遊戲方塊)
> 譯註：裡面的 example 有三個檔案但都是空的=_=
* 實作 "live coding" 讓結果能夠在改變指令方塊時即時展現
> 譯註："live coding" 就是方塊剛放上去，不用案 run 之類的按鈕就會即時呈現新的結果。

> The thing about experiments is that they do not have to succeed. We tend to gloss over failures and dead ends in our work, where failures are punished instead of treated as important vehicles for learning, but failures are essential if you are going to push forward. While I did get the HTML5 drag-and-drop working, the fact that it isn't supported at all on any mobile browser means it is a non-starter for Waterbear. Separating the code out and running code by iterating through the blocks worked so well that I've already begun bringing those ideas to Waterbear, with excellent improvements in testing and debugging. The simplified hit testing, with some modifications, is also coming back to Waterbear, as are the tiny vector and sprite libraries. Live coding hasn't made it to Waterbear yet, but once the current round of changes stabilizes I may introduce it.

用來實驗的事物並不一定要成功，人們常常傾向於掩飾行不通或是失敗的實作，把失敗當成一種懲罰非學習的過程。但其實失敗為成功之母，像我就嘗試 HTML 的拖放功能，而事實上它在手機瀏覽器根本不支援，也代表這功能不會是 Waterbear 的首選。然而分離程式碼並透過迭代方塊運行方塊則運作極佳的提昇了測試跟除錯的效果，優秀到我已經開始將這個主意帶回 Waterbear，簡潔化的拖拉命中判斷在稍作修改後也會回饋到 Waterbear。至於遊戲方塊函式庫及 "live coding" 就還沒有這部份的計畫，但當現階段的改進穩定之後我可能就會引入他們。

#### What Are We Trying to Build, Really?(到底想透過這個專案來建構什麼東西？)

> Building a small version of a bigger system puts a sharp focus on what the important parts really are. Are there bits left in for historical reasons that serve no purpose (or worse, distract from the purpose)? Are there features no-one uses but you have to pay to maintain? Could the user interface be streamlined? All these are great questions to ask while making a tiny version. Drastic changes, like re-organizing the layout, can be made without worrying about the ramifications cascading through a more complex system, and can even guide refactoring the complex system.

建構一個大型系統的小型版本讓我們可以專注於真正重要的部份。思考是否有一些因為歷史因素留下來的無用資料？(更甚是偏離目的的資料)，是否有一些根本沒人用的功能但卻要花心力維護的部份？是否使用者界面可以再做改進？這些都是重做一個小型版本時可以捫心自問的好問題。有一些重大的改變像是重新編排布局這種事就可以直接嘗試看看，不用擔心在複雜系統上會有的一連串影響。甚至後續可以當做重構一個複雜系統的實作指南。

#### A Program is a Process, Not a Thing(這個程式仍在進行式)

> There are things I wasn't able to experiment with in the scope of this project that I may use the blockcode codebase to test out in the future. It would be interesting to create "function" blocks which create new blocks out of existing blocks. Implementing undo/redo would be simpler in a constrained environment. Making blocks accept multiple arguments without radically expanding the complexity would be useful. And finding various ways to share block scripts online would bring the webbiness of the tool full circle.

其實依然有一些我在這個專案的範圍中我沒能實驗的事物，我後續可能用 blockcode 作為基礎來測試。像是如果能實作一個「創造方塊的函式方塊」，讓它可以在現有的方塊之外生成方塊，以及在仍處於建構中的環境引入「上一步/下一步」的功能也會比較簡單，又或是讓方塊可以接受多個參數，而非塞入更多方塊擴展其複雜度也會是很實用的，還有可以找幾種不同的方式讓指令方塊可以透過網路分享，讓其開發工具的整個生命週期都在網頁上，我認為都會是很有趣的。