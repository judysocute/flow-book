# 實際使用

[安裝](installation.md)完 Flow 之後，我們先從最基本的使用來更進一步認識 Flow 吧！大多時候我們要新建立一個 Flow 專案時，[官方](https://flow.org/en/docs/usage/)有給我們一個基本可以遵循的 SOP：

* 使用 `flow init` 指令[初始化專案](usage.md#initialize-project)
* 啟用[背景檢查程序](usage.md#bei-jing-jian-cha-cheng-xu)
* 使用 `// @flow` 註解指定要使用 Flow 的檔案
* 開始[撰寫 Flow 程式](usage.md#ding-yi-flow-dang-an)
* 享受 Flow 的[錯誤檢查](usage.md#cuo-wu-jian-cha)

## 初始化專案 <a id="initialize-project"></a>

初始化 Flow 專案很簡單，只要在任何一個想要初始化的專案資料夾內執行

```bash
flow init
```

執行後，會在專案資料夾內產生一個 `.flowconfig` 檔案。這個專案資料夾下的 `.flowconfig` 檔案會告訴 Flow 背景程序要怎麼檢查 Flow 的檔案，哪些要包括、哪些要忽略等等。

這樣簡單的初始化過後，我們就可以跟別人說我們的專案是一個 Flow 專案了~~ \( 只是沒有寫 Flow 程式而已^^ \)

> 其實在很多 Flow 專案內，一個初始化、沒有多餘設定的 .flowconfig 檔案是很正常的！  
> 但是如果真有寫客製化的需求，可以參考 [.flowconfig 設定](../setting/.flowconfig.md) 章節來進行設定。

## 背景檢查程序 <a id="backgroup-process"></a>

Flow 可以快速的找出我們程式的錯誤，當我們初始化專案之後，我們可以啟動 Flow 檢查程序讓 Flow 來檢查我們的 Flow 檔案。

```bash
flow status
```

這指令不僅會檢查我們 Flow 檔案，還會在背景啟動程序檢查的程序，會持續檢查檔案的改變並且馬上檢查是否有錯誤。

> 也可以只輸入 `flow` 就達到跟 `flow status` 一樣的效果。  
> 其實就把他們兩個當成一樣指令就好了~~

> 一次只會有一個檢查程序在背景執行，所以當你執行了多次 `flow status` 指令，他們會共用行程。

> 如果想停掉背景檢查程序，可以執行 `flow stop` 指令。

## 定義 Flow 檔案 <a id="define-flow-file"></a>

剛才說到 Flow 背景檢查程序會檢查全部的 Flow 檔案。  
那他是怎麽知道「這是個 Flow 檔案」、「需不需要檢查」的呢？

只要在 JavaScript 檔案的最前頭使用註解。

```javascript
// @flow
```

這就是單純的 JavScript 註解加上 `@flow` 的關鍵字而已。Flow 背景程序會取得所有有標示這個註解的檔案並且進行程式的檢查。

> 當然也可以使用 `/* @flow */` 這樣的註解型式囉！

> 如果不想要只檢查有標註的檔案，想要全部檔案都檢查的話可以使用 `flow check --all` 指令，試一下吧！！

## 撰寫 Flow 程式 <a id="write-flow-code"></a>

到這階段我們的環境設置、初始化都完成了，已經可以開始寫 Flow 程式囉！只要是有 `// @flow` 標註的檔案都已經啟用了型別檢查功能，像是這樣的檔案：

```javascript
// @flow
function foo(x: ?number): string {
  if (x) {
    return x;
  }
  return "default string";
}

```

我們可以在函式的參數後加入接收的型別指定與在函式的最後面指明回覆的型別。

眼尖的你可能已經看出這段函式是有問題的，它同時有可能回覆 `int` 或是 `string`，並不符合我們指定的 `string` 型別。但有了 Flow 我們不需要自己看出來，Flow 背景檢查程序會幫我們找出這段錯誤的！

## 錯誤檢查 <a id="check-your-code"></a>

Flow 美好的地方在於它幾乎是「秒回」錯誤檢查的結果，任何時候想要查看錯誤檢查結果只要簡單地敲入：

```bash
# 或是輸入同意義的 `flow status`
flow
```

若是還沒有啟動 [背景檢查程序](usage.md#backgroup-process) ，將會在執行的時候先啟動程序，並且回覆檢查結果。\( 可能會稍微多花個 1 ~ 2 秒啟動程序 \)  
啟動程序之後當我們在寫專案的同時，它就會在背景不斷的檢查程式碼了，當我們再執行 Flow 的時候，回覆速度就真的「秒回」的等級了！！！

以[上面](usage.md#write-flow-code)的程式來說，會回傳這樣的結果：

```bash
$ flow
Error ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ src/index.js:4:12

Cannot return x because number [1] is incompatible with string [2]. [incompatible-return]

        1│ // @flow
 [1][2] 2│ function foo(x: ?number): string {
        3│   if (x) {
        4│     return x;
        5│   }
        6│   return "default string";
        7│ }



Found 1 error
```



