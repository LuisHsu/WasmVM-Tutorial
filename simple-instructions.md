# 算術、參數和控制指令

討論完堆疊和數值形別之後，接下來就能透過指令做簡單的運算

回到 [第一個 WebAssembly 程式](/getting-start.md)，你會看到下面這段程式碼

```
(module
    (func $main
        i32.const 3
        unreachable
    )
    (start $main)
)
```

你可以像下面一樣，用 `;;`** **在程式裡做註解，在`;;`之後，一直到換行為止的文字都會被省略

```
(module
    (func $main
        ;; lalala
        i32.const 3 ;; lalala
        unreachable
    )
    (start $main)
)
```

如果想要一次省略多行，可以用 `(;`** **和`;)`夾住，中間的文字都會被省略

```
(module
    (func $main
        (; woolala
         ;)
        i32.const (; lalala ;) 3
        unreachable
    )
    (start $main)
)
```

---

## 算術指令

### 常數宣告

* `i32.const 整數`
  這個指令會把一個 32 位元的整數放進堆疊
* `i64.const 整數`
  這個指令會把一個 64 位元的整數放進堆疊
* `f32.const 小數` 
  這個指令會把一個單精度浮點數放進堆疊
* `f64.const 小數`
  這個指令會把一個雙精度浮點數放進堆疊

在先前的範例中我們是用 10 進位的數字輸入整數，你也可以在整數或小數的數字前面加上 `0x`，表示輸入的是 16 進位的數字

或是在浮點數的指令裡用 `inf` 輸入無限，`nan` 輸入 NaN

```
(module
    (func $main
        i32.const 3
        i64.const 0x14
        f32.const -0.25
        f64.const -0x2.1
        f32.const -inf
        f32.const nan
        unreachable
    )
    (start $main)
)
```

執行之後得到以下的結果

```
Values in the stack:
Type: f32, Value: nan
Type: f32, Value: -inf
Type: f64, Value: -32.0625
Type: f32, Value: -0.25
Type: i64, Value: 20
Type: i32, Value: 3
```

你可以發現比較晚輸入的數會比較早被列出來，符合堆疊 **後進先出** 的特性

浮點數除了小數之外，也可以用科學記號的方式表示

* 10 進位：$$1.08\times10^{-2} \Rightarrow$$ 1.08e-2 或 1.08E-2
* 16 進位：$$1.08\times16^{-2} \Rightarrow$$ 0x1.08p-2 或 0x1.08P-2

```
(module
    (func $main
        f32.const 1.08e-2
        f32.const 0x1.08p-2
        unreachable
    )
    (start $main)
)
```

```
Values in the stack:
Type: f32, Value: 0.257812
Type: f32, Value: 0.0108
```

### 整數一元運算

一元運算只接受一個數值，因此在堆疊中就是單純的把一個數拿出來，運算完再把結果放回去。

* i32.clz
  * 計算這個整數的位元表示中，最左邊的 1 的左邊有幾個 0
* i32.ctz
  * 計算這個整數的位元表示中，最右邊的 1 的右邊有幾個 0
* i32.popcnt
  * 計算這個整數的位元表示中，總共有幾個 1
* i32.eqz
  * 檢查這個整數是否為 0，是的話放入 1，不是的話放入 0

以下是 i64 的版本，運算方式和 i32 一樣

* i64.clz
* i64.ctz
* i64.eqz
* i64.popcnt

```
(module
    (func $main
        i32.const 2248752  ;; 00000000 00100010 01010000 00110000
        i32.clz 
        unreachable
        i32.const 2248752
        i32.ctz 
        unreachable
        i32.const 2248752
        i32.popcnt 
        unreachable
        i32.const 2248752
        i32.eqz
        unreachable
        i32.const 0
        i32.eqz
        unreachable
    )
    (start $main)
)
```

```
Values in the stack:
Type: i32, Value: 10
Values in the stack:
Type: i32, Value: 4
Values in the stack:
Type: i32, Value: 6
Values in the stack:
Type: i32, Value: 0
Values in the stack:
Type: i32, Value: 1
```

### 整數二元運算

二元運算接受兩個數值，因此在堆疊中把兩個數拿出來，運算完再放回去。

雖然堆疊有後進先出的性質，不過 WebAssembly 為了讓程式比較直覺，在二元指令運算的時候會把順序調換，所以先進堆疊的會放在運算的左邊，後進的會放在右邊，和從堆疊出來的順序相反

下面如果有提到 a, b 兩數，a 表示兩個數值中比較早進入堆疊的數，b 表示比較晚進入堆疊的數

#### 四則運算

* i32.add
  * 兩數相加
* i32.sub
  * 兩數相減
* i32.mul
  * 兩數相乘
* i32.div\_s
  * 兩數當作**有號整數**相除，取商數，捨去小數部份
* i32.div\_u
  * 兩數當作**無號整數**相除，取商數，捨去小數部份
* i32.rem\_s
  * 兩數當作**有號整數**相除，取餘數
* i32.rem\_u
  * 兩數當作**無號整數**相除，取餘數
* 以上指令都有 i64 版本

```
(module
    (func $main
        i32.const 16
        i32.const 11
        i32.add
        unreachable
        i32.const 16
        i32.const 11
        i32.sub
        unreachable
    )
    (start $main)
)
```

```
Values in the stack:
Type: i32, Value: 27
Values in the stack:
Type: i32, Value: 5
```

#### 位元運算

為了方便講解，以下是以 8 位元的整數來舉例

* **i32.and**
  * AND \(或\) 運算：兩個位元都是 1 才會輸出 1，否則輸出 0
  * 把兩個數的位元對齊之後，每個位數分別做 AND 運算

![](/images/and.png)

* **i32.or**
  * OR \(且\) 運算：兩個位元都是 0 才會輸出 0，否則輸出 1
  * 把兩個數的位元對齊之後，每個位數分別做 OR 運算

![](/images/or.png)

* **i32.xor**
  * XOR 運算：兩個位元相同的話輸出 0，否則輸出 1
  * 把兩個數的位元對齊之後，每個位數分別做 XOR 運算

![](/images/xor.png)

* **i32.shl**
  * 把 a 數的位元向左移 b 位，並在右邊補 0

![](/images/shl.png)

* **i32.shr\_u**
  * 把 a 數的位元向右移 b 位，並在右邊補 0

![](/images/rsh_u.png)

* **i32.shr\_s**
  * 把 a 數的位元向右移 b 位，如果 a 數的 sign 是 0 的話補 0，1 的話補 1

![](/images/rsh_s.png)

* **i32.rotl**
  * 把 a 數的位元向左移 b 位，然後把超出去的位元補到右邊

![](/images/lrot.png)

* **i32.rotr**
  * 把 a 數的位元向右移 b 位，然後把超出去的位元補到左邊

![](/images/rrot.png)

* 以上指令都有 i64 版本

#### 比較運算

* **i32.eq**
  * 如果兩數相等輸出 1，否則輸出 0
* **i32.ne**
  * 如果兩數相等輸出 0，否則輸出 1
* **i32.lt\_s**
  * 把兩數當作**有號整數**比較，如果 $$a < b$$輸出 1，否則輸出 0
* **i32.le\_s**
  * 把兩數當作**有號整數**比較，如果 $$a \le b$$ 輸出 1，否則輸出 0
* **i32.lt\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \lt b$$ 輸出 1，否則輸出 0
* **i32.le\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \le b$$ 輸出 1，否則輸出 0
* **i32.gt\_s**
  * 把兩數當作**有號整數**比較，如果 $$a \gt b$$ 輸出 1，否則輸出 0
* **i32.ge\_s**
  * 把兩數當作**有號整數**比較，如果 $$a \ge b$$ 輸出 1，否則輸出 0
* **i32.gt\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \gt b$$ 輸出 1，否則輸出 0
* **i32.ge\_u**
  * 把兩數當作**無號整數**比較，如果 $$a \ge b$$ 輸出 1，否則輸出 0
* 以上指令都有 i64 版本

### 浮點數一元運算

這些操作不會轉換型別，所以就算是得到整數，那個整數的型別一樣是 f32 或 f64

* f32.abs
  * 取絕對值
* f32.neg
  * 正數變負數，負數變正數
* f32.ceil
  * 取大於等於那個數字的最小整數
* f32.floor
  * 取小於等於那個數字的最大整數
* f32.trunc
  * 捨去小數部份，留下整數
* f32.nearest

  * 如果小數部份$$ \lt 0.5$$，捨去小數；如果小數部份$$ \gt 0.5$$  則進位；如果小數部份$$ = 0.5$$，取相鄰整數中是**偶數**的數

  * 乍看之下和四捨五入很像，不過在 "五" 的時候是取偶數，所以 22.5 會得到 22，23.5 會得到 24

* f32.sqrt

  * 開平方根

* 以上指令都有 f64 版本

### 浮點數二元運算

下面如果有提到 a, b 兩數，a 表示兩個數值中比較早進入堆疊的數，b 表示比較晚進入堆疊的數

這些操作不會轉換型別，所以就算是得到整數，那個整數的型別一樣是 f32 或 f64

* f32.add
  * 兩數相加
* f32.sub
  * 兩數相減
* f32.mul
  * 兩數相乘
* f32.div
  * 兩數相除
* f32.eq
  * 如果兩數相等輸出 1，否則輸出 0
* f32.ne
  * 如果兩數相等輸出 0，否則輸出 1
* f32.lt
  * 如果 $$a < b$$ 輸出 1，否則輸出 0
* f32.le
  * 如果 $$a \le b$$ 輸出 1，否則輸出 0
* f32.gt
  * 如果 $$a > b$$ 輸出 1，否則輸出 0
* f32.ge
  * 如果 $$a \ge b$$ 輸出 1，否則輸出 0
* f32.copysign
  * 讓 a 的正負號等於 b 的正負號
* f32.min
  * 取兩數之中比較小的數
* f32.max
  * 取兩數之中比較大的數

### 型別轉換運算

這一系列的操作都是一元運算 \(只接受一個數值\)。和前面的指令不同，這些指令在運算後會把數值的型別轉換成另一種特定的型別

#### Extend

* **i64.extend\_s/i32**
  * 把 i32 轉換成 i64，多出來的部份填上原本**有號整數**的 sign 位元

![](/images/extend.png)

* **i64.extend\_u/i32**
  * 把 i32 轉換成 i64，多出來的部份填上 0

#### Wrap

* **i32.wrap/i64**
  * 把 i64 轉換成 i32，多出來的部份直接捨去

#### Truncate

Truncate 運算是把浮點數根據實際數值轉成整數，如果浮點數的數值是 NaN，$$\pm \infty$$，或是超出該整數型別能表示的數，會得到未定義的結果 \(依據不同的機器、作業系統或執行時期的狀況，可能得到不一樣的結果\)

* **i32.trunc\_s/f32**
  * 將 f32 轉換成 i32 **有**號整數
* **i32.trunc\_s/f64**
  * 將 f64 轉換成 i32 **有**號整數
* **i32.trunc\_u/f32**
  * 將 f32 轉換成 i32 **無**號整數
* **i32.trunc\_u/f64**
  * 將 f64 轉換成 i32 **無**號整數
* **i64.trunc\_s/f32**
  * 將 f32 轉換成 i64 **有**號整數
* **i64.trunc\_s/f64**
  * 將 f64 轉換成 i64 **有**號整數
* **i64.trunc\_u/f32**
  * 將 f32 轉換成 i64 **無**號整數
* **i64.trunc\_u/f64**
  * 將 f64 轉換成 i64 **無**號整數

#### Promote / Demote

* f64.promote/f32
  * 將 f32 依據實際數值轉換成 f64
* f32.demote/f64
  * 將 f64 依據實際數值轉換成 f32

#### Convert

* f32.convert\_s/i32
  * 將 i32 **有**號整數轉換成 f32 
* f32.convert\_s/i64
  * 將 i64 **有**號整數轉換成 f32
* f32.convert\_u/i32
  * 將 i32 **無**號整數轉換成 f32
* f32.convert\_u/i64
  * 將 i64 **無**號整數轉換成 f32
* f64.convert\_s/i32
  * 將 i32 **有**號整數轉換成 f64 
* f64.convert\_s/i64
  * 將 i64 **有**號整數轉換成 f64
* f64.convert\_u/i32
  * 將 i64 **無**號整數轉換成 f64
* f64.convert\_u/i64
  * 將 i64 **無**號整數轉換成 f46

#### Reinterpret

Reinterpret 比較特別，是針對相同位元長度的整數或浮點數，在位元不變的情況下，重新把整數用浮點數的位元格式詮釋成浮點數，或把浮點數用整數的位元格式詮釋成整數

* i32.reinterpret/f32
  * 將 f32 重新詮釋成 i32
* i64.reinterpret/f64
  * 將 f64 重新詮釋成 i64
* f32.reinterpret/i32
  * 將 i32 重新詮釋成 f32
* f64.reinterpret/i64
  * 將 i64 重新詮釋成 f64

---

## 參數指令

* drop
  * 從堆疊中拿出一個數值，然後捨棄不用
* select
  * 從堆疊中拿出 3 個數值，這邊假設為 a, b, c
    * 如果 $$c \ne 0$$ ，把 a 放回堆疊
    * 如果 $$c = 0$$ ，把 b 放回堆疊

---

## 控制指令

* nop
  * 不做任何事
* unreachable
  * 這個指令的本意是製造一個例外狀況，不過在 WasmVM 裡利用製造的中斷來實作系統呼叫
  * 在有開啟系統呼叫時，unreachable 會執行系統呼叫
  * 沒開啟系統呼叫時
    * 以 Debug 模式編譯，會輸出堆疊裡的數值，不放回堆疊，方便除錯
    * 以 Release 模式編譯，會中止程式，並得到錯誤訊息

### 區塊指令

接下來的 block、loop、if 指令會開啟新的程式區塊 \(block\)。其實就是在堆疊裡放入一個標籤 \(Label\)，這個 Label 會記錄目前所處的函式、進入 block 時程式執行的位置，以及離開 block 的時候要接著執行的位置

