# 簡介(Introduction    by:Michael DiBernardo)

[原文出處](http://aosabook.org/en/500L/introduction.html)

translated by<`RanceJen`>

> This is the fourth volume in the Architecture of Open Source Applications series, and the first to not feature the words "open source applications" anywhere in the title.
>
> The first three volumes in the series were about big problems that big programs have to solve. For an engineer who is early in their career, it may be a challenge to understand and build upon programs that are much bigger than a few thousand lines of code, so, while big problems can be interesting to read about, they can also be challenging to learn from.
>
> 500 Lines or Less focuses on the design decisions that programmers make in the small when they are building something new. The programs you will read about in this book were all written from scratch for this purpose (although several of them were inspired by larger projects that the authors had worked on previously).
>
> Before reading each chapter, we encourage you to first think about how you might solve the problem. What design considerations or constraints do you think the author is going to consider important? What abstractions do you expect to see? How do you think the problem is going to be decomposed? Then, when reading the chapter, try to identify what surprised you. It is our hope that you will learn more by doing this than by simply reading through each chapter from beginning to end.

這是 the Architecture of Open Source Applications 系列的第四集，也是第一個沒有把 open source applications 放在標題內的。

系統中的頭三集是指在較大型的程式中要解決的重要問題，或許對於一些經驗較淺的工程師來說要理解及建構超過幾千行 code 的程式會有點挑戰性。雖然閱讀大型的程式會是一件很有趣的事，但同時可能也會使人難以學習。

500 line or less 專住在 programmer 建構較小的事物時在設計上的抉擇，在這本書內會看到的程式都是從頭到尾以此為目的的。(雖然他們有一部份是由作者群曾做過得大型專案所啟發的)

在閱讀每一個章節之前，我們鼓勵先思考「這個問題您會如何解決，您覺得作者在設計時會認為什麼注意事項或是限制是重要的？您期望看到什麼樣的概念？您認為這是個怎樣的問題。」在閱讀每個章節時，試著去找出有哪些是讓您訝異的，比起只是單純把每個章節從頭看到尾，我們希望您會從上敘的方法中學習到更多。

> Writing a useful program in fewer than 500 lines of source code---without resorting to cheap tricks---is a challenging exercise in itself; writing one to be read for pedagogical purposes when neatly rendered in a printed book is even tougher. As such, the editors have occasionally taken liberties with some of the source formatting when porting it into the book. The original source for each chapter can be found in the code subdirectory of its project folder.
>
> We hope that the experiences of the authors in this book will help you grow out of your comfort zone in your own programming practice.
> ---Michael DiBernardo

要在不使用一些投機技巧的情況下寫出一個小於五百行 code 的實用程式本身就是一個挑戰了，還要是能被以教學目的印刷在書上的就更難了。所以在要移植到書上時作者們偶爾會將一些格式做修改。每個章節的原始資源都可以在該章節專案的 code 子資料夾找到。
---Michael DiBernardo

> 註：以下 Contributors,Acknowledgments,Contributing,Colophon 維持原文

Contributors 

Michael DiBernardo (editorial): Michael DiBernardo is an engineer and director of delivery at Wave, and a past PyCon Canada chair. He writes at mikedebo.ca.

Amy Brown (editorial): Amy Brown is a freelance editor based in Toronto. She specializes in science and academic editing, and working with self-publishing authors. She co-edited the Architecture of Open Source Applications books with Greg Wilson.

Dethe Elza (Blockcode): Dethe is a geek dad, aesthetic programmer, mentor, and creator of the Waterbear visual programming tool. He co-hosts the Vancouver Maker Education Salons and wants to fill the world with robotic origami rabbits.

Malini Das (CI): Malini is a software engineer who is passionate about developing quickly (but safely!), and solving cross-functional problems. She has worked at Mozilla as a tools engineer and is currently honing her skills at Twitch.

Dustin J. Mitchell (Cluster): Dustin is an open source software developer and release engineer at Mozilla. He has worked on projects as varied as a host configuration system in Puppet, a Flask-based web framework, unit tests for firewall configurations, and a continuous integration framework in Twisted Python.

Daniel Rocco (Contingent): Daniel loves Python, coffee, craft, stout, object and system design, bourbon, teaching, trees, and Latin guitar. Thrilled that he gets to write Python for a living, he is always on the lookout for opportunities to learn from others in the community, and to contribute by sharing knowledge. He is a frequent speaker at PyAtl on introductory topics, testing, design, and shiny things; he loves seeing the spark of delight in people's eyes when someone shares a surprising or beautiful idea. Daniel lives in Atlanta with a microbiologist and four aspiring rocketeers.

Brandon Rhodes (Contingent): Brandon Rhodes started using Python in the late 1990s, and for 17 years has maintained the PyEphem library for amateur astronomers. He works at Dropbox, has taught Python programming courses for corporate clients, consulted on projects like the New England Wildflower Society's "Go Botany" Django site, and will be the chair of the PyCon conference in 2016 and 2017. Brandon believes that well-written code is a form of literature, that beautifully formatted code is a work of graphic design, and that correct code is one of the most transparent forms of thought.

A. Jesse Jiryu Davis (Crawler): Jesse is a staff engineer at MongoDB in New York. He wrote Motor, the async MongoDB Python driver, and he is the lead developer of the MongoDB C Driver and a member of the PyMongo team. He contributes to asyncio and Tornado. He writes at emptysqua.re.

Guido van Rossum (Crawler): Guido is the creator of Python, one of the major programming languages on and off the web. The Python community refers to him as the BDFL (Benevolent Dictator For Life), a title straight from a Monty Python skit.

Dann Toliver (Dagoba): Dann enjoys building things, like programming languages, databases, distributed systems, communities of smart friendly humans, and pony castles with his two-year-old.

Taavi Burns (DBDB): As the newest bass (and sometimes tenor) in Countermeasure, Taavi strives to break the mould... sometimes just by ignoring its existence. This is certainly true through the diversity of workplaces in his career: IBM (doing C and Perl), FreshBooks (all the things), Points.com (doing Python), and now at PagerDuty (doing Scala). Aside from that—when not gliding along on his Brompton folding bike—you might find him playing Minecraft with his son or engaging in parkour (or rock climbing, or other adventures) with his wife. He knits continental.

Leo Zovic: Leo (better known online as inaimathi) is a recovering graphic designer who has professionally written Scheme, Common Lisp, Erlang, Javascript, Haskell, Clojure, Go, Python, PHP and C. He currently blogs about programming, plays board games and works at a Ruby-based startup in Toronto, Ontario.

Dr. Christian Muise (Flow shop): Dr. Muise is a Research Fellow with the Model-based Embedded and Robotic Systems group at MIT's Computer Science and Artificial Intelligence Laboratory. He is interested in a variety of topics including AI, data-driven projects, mapping, graph theory, and data visualization, as well as Celtic music, carving, soccer, and coffee.

Yoav Rubin (CircleDB): Yoav is a Senior Software Engineer at Microsoft, and prior to that was a Research Staff Member and a Master Inventor at IBM Research. He works now in the domain of data security in the cloud, and in the past his work focused on developing cloud- or web-based development environments. Yoav holds an MSc in Medical Research in the field of Neuroscience and BSc in Information Systems Engineering.

Cate Huston (Image filters): Cate is a developer and entrepreneur focused on mobile. She’s lived and worked in the UK, Australia, Canada, China and the United States, as an engineer at Google, an Extreme Blue intern at IBM, and a ski instructor. Cate speaks internationally on mobile development, and her writing has been published on sites as varied as Lifehacker, The Daily Beast, The Eloquent Woman and Model View Culture. She co-curates Technically Speaking, blogs at Accidentally in Code and is @catehstn on Twitter.

Allison Kaptur (Interpreter): Allison is an engineer at Dropbox, where she helps maintain one of the largest networks of Python clients in the world. Before Dropbox, she was a facilitator at the Recurse Center, a writers' retreat for programmers in New York. She's spoken at PyCon North America about Python internals, and loves weird bugs.

Erick Dransch (Modeller): Erick is a software developer and 2D and 3D computer graphics enthusiast. He has worked on video games, 3D special effects software, and computer-aided design tools. If it involves simulating reality, chances are he'd like to learn more about it. You can find him online at erickdransch.com.

Carl Friedrich Bolz (Object model): Carl is a researcher at King's College London and is broadly interested in the implementation and optimization of all kinds of dynamic languages. He is one of the core authors of PyPy/RPython and has worked on implementations of Prolog, Racket, Smalltalk, PHP and Ruby.

Marina Samuel (OCR): Marina is an engineer at Mozilla and a current MSc student in Applied Computing (Artifical Intelligence) at the University of Toronto. She hopes to one day build robots that will take over the planet.

Dessy Daskalov (Pedometer): Dessy is an engineer by trade, an entrepreneur by passion, and a developer at heart. She's currently the CTO and co-founder of Nudge Rewards. When she’s not busy building product with her team, she can be found teaching others to code, attending or hosting a Toronto tech event, and online at dessydaskalov.com and @dess_e.

Eunsuk Kang (Same-origin policy): Eunsuk is a PhD candidate and a member of the Software Design Group at MIT. He received his SM (Master of Science) in Computer Science from MIT (2010), and a Bachelor of Software Engineering from the University of Waterloo (2007). His research projects have focused on developing tools and techniques for software modeling and verification, with applications to security and safety-critical systems.

Santiago Perez (Same-origin policy): Santiago is a PhD student in the Software Design Group at MIT. He received his SM in Computer Science from MIT (2015), and an undergraduate degree from ITBA (2011). He used to work at Google, developing frameworks and tools to make engineers more productive (2012). He currently spends most of his time thinking about design and version control.

Daniel Jackson (Same-origin policy): Daniel is a professor in the Department of Electrical Engineering and Computer Science at MIT, and leads the Software Design Group in the Computer Science and Artificial Intelligence Laboratory. He received an MA from Oxford University (1984) in Physics, and his SM (1988) and PhD (1992) in Computer Science from MIT. He was a software engineer for Logica UK Ltd. (1984-1986), Assistant Professor of Computer Science at Carnegie Mellon University (1992-1997), and has been at MIT since 1997. He has broad interests in software engineering, especially in development methods, design and specification, formal methods, and safety-critical systems.

Jessica B. Hamrick (Sampler): Jess is a PhD student at UC Berkeley where she studies human cognition by combining probabilistic models from machine learning with behavioral experiments from cognitive science. In her spare time, Jess is a core contributor to IPython and Jupyter. She also holds a BS and MEng in Computer Science from MIT.

Audrey Tang (Spreadsheet): A self-educated programmer and translator, Audrey works with Apple as an independent contractor on cloud service localization and natural language technologies. Audrey has previously designed and led the first working Perl 6 implementation, and served in computer language design committees for Haskell, Perl 5, and Perl 6. Currently Audrey is a full-time g0v contributor and leads Taiwan’s first e-Rulemaking project.

Leah Hanson (Static analysis): Leah Hanson is a proud alum of Hacker School and loves helping people learn about Julia. She blogs at blog.leahhanson.us and tweets at @astrieanna.

Ned Batchelder (Template engine): Ned is a software engineer with a long career, currently working at edX to build open source software to educate the world. He's the maintainer of coverage.py, an organizer of Boston Python, and has spoken at many PyCons. He blogs at nedbatchelder.com. He once had dinner at the White House.

Greg Wilson (Web server): Greg is the founder of Software Carpentry, a crash course in computing skills for scientists and engineers. He has worked for 30 years in both industry and academia, and is the author or editor of several books on computing, including the 2008 Jolt Award winner Beautiful Code and the first two volumes of The Architecture of Open Source Applications. Greg received a PhD in Computer Science from the University of Edinburgh in 1993.
Acknowledgments

The Architecture of Open Source Applications series would not exist without the hard work of Amy Brown and Greg Wilson. This particular book would not have been possible without the incredible efforts of our army of technical reviewers:

    Amber Yust
    Andrew Gwozdziewycz
    Andrew Kuchling
    Andrew Svetlov
    Andy Shen
    Anton Beloglazov
    Ben Trofatter
    Borys Pierov
    Carise Fernandez
    Charles Stanhope
    Chris AtLee
    Chris Seaton
    Cyryl Płotnicki-Chudyk
    Dan Langer
    Dan Shapiro
    David Pokorny
    Eric Bouwers
    Frederic De Groef
    Graham Lee
    Gregory Eric Sanderson
    James O'Beirne
    Jan de Baat
    Jana Beck
    Jessica McKellar
    Jo Van Eyck
    Joel Crocker
    Johan Thelin
    Johannes Fürmann
    John Morrissey
    Joseph Kaptur
    Josh Crompton
    Joshua T. Corbin
    Kevin Huang
    Maggie Zhou
    Marc Towler
    Marcin Milewski
    Marco Lancini
    Mark Reid
    Matthias Bussonnier
    Max Mautner
    Meggin Kearney
    Mike Aquino
    Natalie Black
    Nick Presta
    Nikhil Almeida
    Nolan Prescott
    Paul Martin
    Piotr Banaszkiewicz
    Preston Holmes
    Pulkit Sethi
    Rail Aliiev
    Ronen Narkis
    Rose Ames
    Sina Jahan
    Stefan Turalski
    William Lachance

Chris Seaton, John Morrissey, and Natalie Black deserve extended thanks for going above and beyond in their technical reviewing. The quantity and depth of their reviews was instrumental in moving the book forward at several sticking points.

We are very grateful to PagerDuty for their financial support.

Contributing

If you'd like to report errors or translate the content into other languages, please [open an issue](https://github.com/aosabook/500lines/).

Colophon

The cover font is Museo from the exljibris foundry, by Jos Buivenga.

The front cover photo is composed of twenty-three separate focus-stacked images of watch gear assemblies. The picture was taken by Kellar Wilson.

This book was built with open source software (with the exception of the cover). Programs like LaTeX, Pandoc, and Python were especially helpful.