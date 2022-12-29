---
title: 2022-12-29
tags:
  - Java
  - TIL
---

## Java

### ロガーインスタンスの生成方法

どの OSS か忘れてしまったが普段見る Logger の宣言方法と別の方法が使われていたのでメモ。

依存関係に [Lombok](https://projectlombok.org/) が含まれることを気にしないプログラムでは `@Slf4j` を使っているため、Logger のインスタンスの生成方法を意識することはない。
しかし、世知辛い事情から Lombok が使えなかったり、依存関係を最小限にしておきたい場合は、Logger のインスタンスを生成するコードを書く必要が出てくる。
そういった時、以下のように書かれているのをよく見る。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyClass {
  private static final Logger log = LoggerFactory.getLogger(MyClass.class);
}
```

インターネットにある記事を見てもこのように書かれていることが多いと思うが、この書き方には少しだけ問題がある。
誰かが楽をしようとして他のクラスからコピペした場合に誤ったクラス名が `Logger` に渡されてしまう。
そうなった場合、ロガーは誤ったログを出力してしまい、問題が発生してログを漁っているとき原因の特定が難しくなる。

そこで、以下のように常に正しいクラスを `Logger` に渡すことができる。

```java
import java.lang.invoke.MethodHandles;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyClass {

  private static final Logger log = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
}
```

`MyClass.class` を直接渡すのではなく、`MethodHandles` を使ってクラスを取ってくるようにする。

#### 参考URL

- [コピペによるLogger名の間違いを防止する - Qiita](https://qiita.com/pale2f/items/11ddc190e8231cae11e1)
- [MethodHandleを使ったら、リフレクションより早くメソッドを実行できる？ - tyablog.net](https://tyablog.net/2020/02/24/methodhandle-vs-reflection/)