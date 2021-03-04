---
description: 必要環境安裝
---

# 環境安裝

## 編譯器設定 <a id="compiler"></a>

說到底我們的程式還是要給編譯器看的，Flow 程式編譯器是看不懂的，這時候就要把我們的程式預先編譯成編譯器看得懂的，我們可以選用 [Babel](https://babeljs.io/) 或 [flow-remove-types](https://github.com/facebook/flow/tree/master/packages/flow-remove-types)。本書選用的是 Babel。

Babel 呢，是專門用來處理 JavaScript 的轉換器，它提供了 ES6 向下兼容的功能、語法轉換等等功能。更甚至可以支援 Flow 語法。  
Babel 會解析 Flow 語法，將型別註解移除轉換為一般的 JavaScript。

首先，我們需要安裝 `@babel/core`, `@babel/cli`, `@babel/preset-flow`

```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-flow
```

安裝完之後建立 `.babelrc` 檔案，在 preset 區塊內加入 `"@babel/preset-flow"` 告訴 babel 如何處理 Flow 的內容。

```javascript
{
  "presets": ["@babel/preset-flow"]
}
```

再來將[前篇](introduce.md)範例的檔案加到 `src` 資料夾內後執行

```bash
./node_modules/.bin/babel src/ -d lib/                                                               00:36:41 
```

也可以加到 `package.json` 的 scripts 內

```javascript
{
  "name": "test-prj",
  "main": "index.js",
  "scripts": {
    "build": "babel src/ -d lib/",
    "prepublish": "npm run build"
  }
}
```

以下給個編譯前後範例。

{% tabs %}
{% tab title="編譯前" %}
{% code title="src/index.js" %}
```javascript
// @flow
function square(n: number): number {
  return n * n;
}

square(2);
```
{% endcode %}
{% endtab %}

{% tab title="編譯後" %}
{% code title="lib/index.js" %}
```javascript
function square(n) {
  return n * n;
}

square(2);
```
{% endcode %}
{% endtab %}
{% endtabs %}

可以看到 babel 編譯過後把最上面的 `// @flow` 跟 Flow 的型別註解都拿掉了，變成了一個再單純不過的 JavaScript 程式碼。

{% hint style="info" %}
為什麼 package.json 內要特別加個 prepublish 呢？  
因為 prepublish 會在 npm publish 之前執行，這樣可以保證在發佈到 npm 的時候一定是經過編譯的程式。算是一個好習慣吧！！  
\( 也會在 npm install 的時候執行哦，見[官方文件](https://docs.npmjs.com/cli/v7/using-npm/scripts#prepare-and-prepublish) \)
{% endhint %}

## 設定 Flow 環境 <a id="flow-environment"></a>

用於檢查初始化 Flow 專案、檢查檔案內 Flow 規則。

如果使用過 `@vue/cli` 或 `create-react-app` 等工具，其實使用上也差不多！

首先安裝 `flow-bin` 到 `devDependency` 

```bash
npm install --save-dev flow-bin
```

再加入 `"flow"` 到 `package.json` 中的 scripts 內

```javascript
{
  "name": "test-prj",
  "main": "index.js",
  "scripts": {
    // ... 略過 ...
    "flow": "flow"
  },
  "devDependencies": {
    "flow-bin": "^0.145.0"
  }
}
```

**執行 Flow:**

第一步先初始化專案:

```bash
npm run flow init

> flow
> flow "init"
```

執行成功的話會產生一個 `.flowconfig` 檔案

完成後再來檢查專案:

```bash
npm run flow

> flow
No errors!
```

詳細使用下個章節再來細細解說！

