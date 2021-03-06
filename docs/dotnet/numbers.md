# 数値型

Fable では、F# の数値型を使いますが、`int64`、`uint64`、`bigint`、`decimal`を除いて、すべて JS Number (64-bit floating type)に変換されます。

Fable の数値は.NET のセマンティックとほぼ互換性がありますが、Javascript の型に変換すると、次のような結果になります。

- (非標準) 浮動小数点数はすべて 64 ビット(`double`)で実装されています。このため、`float32`の数値は予想以上に正確です。
- (非標準) 32 ビット以下の算術整数は、 `double` に埋め込まれた整数として、期待とは異なる切り捨てで実装されます。
- (OK)型間の変換は正しく切り捨てられます。
- (OK) 64 ビットと 32 ビットの整数のビット演算は正しく，適切なビット数に切り詰められます．
- (非標準) 16 ビットと 8 ビット整数のビット演算は、JavaScript の 32 ビットビット演算のセマンティクスを使用しています。結果は期待通りに切り捨てられず、シフトオペランドはデータ型に合うようにマスクされません。
- (OK) Longs はカスタム実装で、.NET と同じセマンティクスを持ち、64 ビットで切り捨てられますが、処理速度は遅くなります。

32 ビット整数は、.NET とは 2 つの点で異なります。

- オーバーフロー時に 32 ビットに切り捨てられることはなく、52 ビットの精度が基本となっています。必要であれば，`>> 0`で強制的に切り捨てることができます．
- 52 ビットの絶対値を超えると浮動小数点は精度を失います。そのため、オーバーフローすると、予期しない下位の 0 ビットになります。

精度の損失は、単一の乗算で確認できます。

```fsharp
((1 <<< 28) + 1) * ((1 <<< 28) + 1) >>> 0
```

乗算の積は内部的には `0x0100_0000_2000_0000` に丸められた 2 重表現になっています。これが `>> 0` によって 32 ビットに切り捨てられると、結果は `0x2000_0000` となり、.NET の正確な低次ビットの値 `0x2000_0001` ではありません。

同じ問題は、繰り返される算術演算が内部（切り捨てられない）値を大きくする場合にも見られます。たとえば、線形合同乱数生成器です。

```fsharp
let rng (s:int32) = 10001*s + 12345
```

その結果にこれを繰り返し適用して生成される数字は、`s`の値が 2^53 を超えたとき、4 回目の疑似乱数以降はすべて偶数にります。

```fsharp
let rec randLst n s =
    match n with
    | 0 -> [s]
    | n -> s :: randLst (n-1) (rng s)

List.iter (printfn "%x") (randLst 7 1)
```

その結果、出力された擬似乱数のリストは Fable では動作しません。

|    Fable |     .NET |
| -------: | -------: |
|        1 |        1 |
|     574a |     574a |
|  d524223 |  d524223 |
| 6a89e98c | 6a89e98c |
| 15bd0684 | 15bd0685 |
| 3d8b8000 | 3d8be20e |
| 50000000 | 65ba5527 |
|        0 | 2458c8d0 |

## ワークアラウンド

- 低次ビットの正確な演算が必要で、オーバーフローにより 2^53 より大きな数値が発生する場合は、 `int32`, `uint32` の代わりに、正確な 64 ビットを使用する `int64`, `uint64` を使用します。
- また、2^53 より大きな数値が発生する前に、 `>> 0` や `>> 0u` を使ってすべての演算を切り捨てることもできます。

## 印刷

.NET の `printf`, `sprintf`, `ToString` からは、ちょっとした変更があります。負の符号付き整数は、16 進数では符号＋大きさで表示されますが、.NET では 2 の補数ビットパターンで表示されます。
