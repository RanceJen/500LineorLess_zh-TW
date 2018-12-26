# 分散式系統(Clustering by Consensus    by:Dustin J. Mitchell)

[原文出處](http://aosabook.org/en/500L/clustering-by-consensus.html)

translated by<`RanceJen`>

> Dustin is an open source software developer and release engineer at Mozilla. He has worked on projects as varied as a host configuration system in Puppet, a Flask-based web framework, unit tests for firewall configurations, and a continuous integration framework in Twisted Python. Find him as [@djmitche](http://github.com/djmitche) on GitHub or at dustin[@mozilla.com](dustin@mozilla.com).

Dustin 是一個開源軟體開發者和 Mozilla 的佈署工程師，他曾在開發過各式各樣在  Puppet 中的端口配置系統、以 Flask 為基礎的網頁框架，防火牆設置的單元測試以及一個用 Twisted Python 開發的持續集合框架

> 譯註：上面沒翻出來的都是框架名，至於 Mozilla 我相信大家都認識就沒什麼好翻的了。

### Introduction 引言
>
> In this chapter, we'll explore implementation of a network protocol designed to support reliable distributed computation. Network protocols can be difficult to implement correctly, so we'll look at some techniques for minimizing bugs and for catching and fixing the remaining few. Building reliable software, too, requires some special development and debugging techniques.

在這個章節中，我們會探索如何實現一個網路協議，並使其支援有可靠性的分散式運算。要正確的實作網路協議是有其難度的，所以我們會使用一些技巧來最小化、排查並修復 bug，以建構一個可靠的軟體，這同時也會需要一些特定的開發和除錯技巧。