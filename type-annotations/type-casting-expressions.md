---
description: 中文名稱我自己寫寫的~ 可能不是正規的表達，但是我個人認為是有助於理解的
---

# 型別扮演表達式 \( Type Casting Expressions \)

常常我們在做型別確認的時候，會要寫一個 `function` 或是宣告變數來確認

```javascript
// @flow
function fn(num: number) {
    // ...
}

fn(123);
fn('123'); // 錯誤!

// 或是
const num: number = 123;
const num2: string = 123; // 錯誤!
```

用久了確實是有點麻煩。這還只是基礎型別呢！如果哪天要使用自定義的型別那真是惡夢阿~

我們的痛苦 Flow 是知道的， Flow 支援 **型別扮演表達式 \( Type Casting Expressions \)** 來解決這個問題。

## 語法 <a id="grammar"></a>

`Type Casting Expression` 的語法是在小括號內 `( )`  使用 `value`加上冒號 `:` 在接上 `Type` 完成一個表達式。

```javascript
(value: Type)
```

> 小括號是必要的哦！避免跟其他表達式混淆。

它可以應用在各種場景：

```javascript
// 這裡只是表達式的範例
// 實際寫上去是會報錯的哦~ 因為我們並沒有 Type, value, prop 等變數的定義
let val = (value: Type);
let obj = { prop: (value: Type) };
let arr = ([(value: Type), (value: Type)]: Array<Type>);
```

在表達式中，還可以放入其他邏輯表達式。

```javascript
(2 + 2: number)
```

當我們用 `babel` 編譯過後，它只會留下 `value` 的部分而已。

```javascript
// @flow
(value: Type)
```

```javascript
value
```

\( 這段雖然 IDE 會報錯，但是 `babel` 還是可以轉的哦，自己試試看吧！  \)

## 型別扮演 & 型別轉換 <a id="type-casting"></a>

其實這個 `Type Casting Expressions` 的中文名我也想了很久，到底是要用型別扮演，還是轉換會比較好 \( 開始後悔年輕沒有好好上體育老師的國文課了... \)

因為以下的語法是通的哦

```javascript
// @flow
let value = 42;

(value: 42); // ok!
(value: number); // ok!
(value: 43); // 錯誤!
```

第 4 行我們問他說，`value` 可不可以是 `42` 他說可以。  
第 5 行我們問他說，`value` 可不可以是 `number` 他也說可以。  
第 6 行我們問他說，`value` 可不可以是 `43` 他說不行。

所以我的解讀是，它作為扮演來解釋會比較精準一點。

## 回傳值 <a id="return"></a>

當我們使用 `Type Casting Expression` 回傳時，在 Flow 中它會回傳它的型態，什麼意思呢？

看下面的範例吧

```javascript
let value = 42;

(value: 42); // ok
(value: number); // ok

let newValue = (value: number);

(newValue: 42); // 錯誤!
(newValue: number); // ok
```

以我們直覺上來說，`newValue` 等於 `value`，所以當我們問它 `(newValue: 42)` 的時候應該要說"是"才對 \( 實際上他也是 `42` 沒錯 \)，但是 Flow 硬生生的就給它報錯了。  
為什麼呢？因為在 Flow 中 `Type Casting Expression` 的回傳的型別資訊是冒號後面的型別，而不是值得型別 \( 儘管實際上在 JavaScript 當中是值 \)，所以在範例中，他只知道 `newValue` 是 `number` 而不知道它是 `42`。

## 透過 any 轉換型別 <a id="through-any"></a>

在剛才的演示中，我們有發現 `Type Casting` 這個動作只能在扮演相同的型別，是不可以扮演成不同的型別的。

但是有一個型別它可以扮演所有型別，那就是 any，在很多情況下我們會避免使用 any，畢竟使用了 any，就等於失去了型別判斷的意義，容易造成無法預期的錯誤。

### 不好的使用場景 <a id="bad"></a>

一樣拿官方的範例來做為演示

```javascript
let value = 42;

(value: number);// ok
(value: string); // 錯誤

let newValue = ((value: any) string);

(newValue: number); // 錯誤
(newValue: string); // ok
```

在一開始，我們問說 `(value: string)` 的時候 Flow 跟我們說不可以，他不是 `string`  
後來我們先問說 `(value: any)`，Flow 說可以，他是 `any`，現在 Flow 已經把 `(value: any)` 當成 any 來看待了，  
我們後面又接著問 `((value: any): string)` ，Flow 說可以，`any` 可以是 `string`。

因此我們在最後兩行 `(newValue: number)` 的時候錯了，反倒是 `(newValue: string)` 的時候正確。可是實際上的 `newValue` 值卻是數字 `42`，而非字串。

當然，這樣的轉換型別是不建議的，用得不好就像上例這樣，Flow 都被自己搞到給出錯誤的提示，但是如果 `any` 型別使用得當，在某些真的需要使用型別轉換的場景則又顯得格外的實用。

### 好的使用場景 <a id="better"></a>

官方給了我們一個非常實用的用例。  
用例場景是當我們要 clone 物件的時候，我們可能會先寫出像這樣的一段

```javascript
// @flow
function cloneObject(obj) {
    const clone = {};
    
    Object.keys(obj).forEach((key) => {
        clone[key] = obj[key]
    })
    
    return clone;
}

// --- 測試 ---

type Person = {
    name: string,
    age: number,
};

const person001: Person = {
    name: 'Wayne',
    age: 26,
};

(cloneObject(person001): Object); // ok
(cloneObject(person001): Person); // 錯誤
```

這是很簡單的 clone 物件 function，他可以回傳一個 `Object` 型態，但是卻沒有辦法做到回傳收到的 `Person` 型態。  
\( `Object` 型態源自於 `const clone = {};` 的預設型別 \)

如果我們想要讓它能如預期回傳收到的 `Person` 型態就可以用到剛才的 `any` 型態了。

```javascript
// @flow
function cloneObject(obj) {
    const clone = {};
    
    Object.keys(obj).forEach((key) => {
        clone[key] = obj[key]
    })
    
    return ((clone: any): typeof obj); // <<
}

// --- 測試 ---

type Person = {
    name: string,
    age: number,
};

const person001: Person = {
    name: 'Wayne',
    age: 26,
};

(cloneObject(person001): Object); // ok
(cloneObject(person001): Person); // ok
```

我們只有動到 `return` 那行，先將 `clone` 物件轉換為 `any` 型態，現在它可以扮演任何型別了，再用 `typeof` 取出 `obj` 的型別，將 `clone` 轉過去，就大功告成了！還不錯吧？

## TODO: 剩下的晚點寫~ <a id="todo"></a>

