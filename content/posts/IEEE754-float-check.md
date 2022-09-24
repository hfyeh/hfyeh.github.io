---
layout: post
title: "C 語言中的浮點數與 IEEE 754"
date: 2019-07-02
tags: [c]
---

今天與同事討論 C 語言中，對一個浮點數取 floor 的問題。剛好最近有看到 IEEE 754 標準如何規範浮點數格式[^1]，就趁此機會仔細驗證一下。仍建議看第一手資料，以下純為記錄和分享用途而已。

先從一個問題開始，我用 GCC 編譯以下的程式碼並執行：

<script src="https://gist.github.com/hfyeh/3ab390a5dfbca920a675b75134900e93.js"></script>

``` shell
floor(f1), 20 digital points: 2.00000000000000000000
floor(f2), 20 digital points: 3.00000000000000000000
```

為什麼 2.999999 取 floor 得到 2.0，而 2.9999999 取 floor 會得到 3.0 呢？我們的都知道，是因為浮點數有精度問題，單精度浮點數的精準度在絕大多數的機器上有效位數只到小數後第六位。這篇的目標就是從標準的角度去親自確認這件事情。

<!--more-->

首先，單精度浮點數是 32 bits，任何的小數都會轉換成二進制的科學符號後被存在這 32 個位元裡。

借用 Wikipedia 的圖
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Float_example.svg/1180px-Float_example.svg.png" style="background-color:#fff" />

這 32 個 bit，被分成三大部分，Sign、Exponent、Fraction（或稱 Significand/Mantissa）。

- Sign：0 表示為正數，1 表示為負數。
- Exponent：共 8 bits，值為 0~255，代表 2 的指數次方。由於指數也需要有正負號，如採用 Two's Complement 來代表正負，則 0 到 127 代表 0 到 127，128 到 255 代表 -128 到 -1。
- Significand：共 23 bits，但由於規定第一有效位一定要是 1，故有 24 有效位數[^2]。

使用[這個轉換器](https://www.h-schmidt.net/FloatConverter/IEEE754.html)可以知道，當想要對 f1 這個長度 8 bits 的記憶體位置存入 2.999999 時，其實際存起來的的值是 2.99999904632568359375。有效位為小數後六位（最低位 2^-23 = 0.000000119209... 為可存的離散數間的最小間隔）。對這個值取 float，自然就是得到 2.0。而對 f2 這個長度 8 bits 的記憶體位置存入 2.9999999 時，實際存起來的值是 3.0，對 3.0 取 floor 會得到 3.0。

可以實際執行下列程式確認這點。

<script src="https://gist.github.com/hfyeh/9120ca3a6189e1fdd8f169186fa27690.js"></script>

``` shell
f1, 6 digital points: 2.999999
f2, 6 digital points: 3.000000
f1, 20 digital points: 2.99999904632568359375
f2, 20 digital points: 3.00000000000000000000
floor(f1), 20 digital points: 2.00000000000000000000
floor(f2), 20 digital points: 3.00000000000000000000
```

同時也可以發現，`printf` 是會對其參數做過一些處理才顯示出來的（15:16）
，這是為什麼有時候我們不能盡信 `printf` 給出的資訊的原因。用 GDB 或 Compiler 提供的 Debugger 直接對記憶體求值，才會得到未經處理的真實值。

參考資料：https://www.bottomupcs.com/types.xhtml

[^1]: 舉例來說，可以參考 IEEE 754 對[單精度浮點數](https://zh.wikipedia.org/wiki/%E5%96%AE%E7%B2%BE%E5%BA%A6%E6%B5%AE%E9%BB%9E%E6%95%B8)的定義。
[^2]: 先假設我們只有 4 bits Significand 可用，Exponent 為 0 的情況下，0.5 會是這樣存 `0100`，其十進位換算方式是 `(0*2^0 + 1*2^-1 + 0*2^-2 + 0*2^-3)*2^0 = 0.5`，但這樣存的缺點是能存的離散數的差只有到 2^-3 = 0.125。假如我們規定首位一定要是 1（就像十進位的科學記號，個位數一定會大於零），以此例來說，我們需要向左推兩位，Siginicant bits 就會變成 `(1)0000`，同時因為對 Significand 左推，Exponent 必須減二變為 -2，計算結果就變成 `(1*2^0 + 0*2^-1 + 0*2^-2 + 0*2^-3 + 0*2^-4)*2^-1 = 0.5`，會得到一樣的結果，但是能存的離散數的差變小為 2^-4 = 0.0625。
