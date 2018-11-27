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

Blockcode 的指令就像是任何（不管是方塊式還是語言式）語言的指令一樣，是一些可運算操作的組合，而在 Blockcode 之中的指令則是由迭代的 html 元素組成，他們與特定的 JavaScript 相關聯且在迭代到該方塊時執行，有些方塊可以包含其他的方塊（並且也負責執行他們），有些方塊可以把參數傳遞到實際執行的函式內。
 
> 註：這邊 script 的意指「一個可運算的指令」，像是 `rm -rf /` 。

> 註2：可以包含其他方塊的方塊聽起來有點怪，但其實就是類似 for 這種東西，指令包指令，是不是很合理容易理解惹ヽ(●´∀`●)ﾉ