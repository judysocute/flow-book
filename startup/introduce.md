---
description: 強大、可選的型別檢查工具
---

# Flow 簡介

Flow 是一套 JavaScript 的型別檢查工具，透過型別標註來檢查程式是否依照使用者預期的型別呈現。

```javascript
// @flow
function square(n: number): number {
    return n * n;
}

square('2'); // 錯誤!
```

Flow 本身是看得懂我們的 JavaScript 程式的，所以我們也不需要上面那樣辛苦的定義型別，Flow 其實是可以自己「猜出來」的。

可以的話我們應該儘可能的在保持程式碼簡潔、易讀的原則下進行開發。

```javascript
// @flow
function square(n) {
  return n * n; // 錯誤!
}

square("2");
```

Flow 是可以漸進式的引入專案的，只要在想要引入的檔案內加入 `@flow` 註解就好了，其他還沒有要使用的維持原樣就好。在使用上是非常無痛的！



