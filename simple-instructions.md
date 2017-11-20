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

* 10 進位：$$$$$$1.08\times10^{-2} \Rightarrow$$ 1.08e-2 或 1.08E-2
* 16 進位：$$$$$$1.08\times16^{-2} \Rightarrow$$ 0x1.08p-2 或 0x1.08P-2

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

### 整數二元運算

### 浮點數一元運算

### 浮點數二元運算

### 型別轉換

---

## 參數指令

---

## 控制指令



