---
title: Java + GraalVM Native Image
description: Java で書いたプログラムで GraalVM Native Image を作成する方法の紹介です。
tags:
  - Java
  - GraalVM
updated_at: 2023-01-07
---

## TL;DL

https://github.com/eowiz/java-graalvm-sample

## はじめに

[GraalVM](https://www.graalvm.org/) をご存知だろうか。
もう何年も前に Java や JavaScript、Ruby を始めとする複数言語のランタイムとして機能し、ネイティブイメージの作成もするという随分と野心的なプロジェクトが始まったなぁ…という感想と共に頭の片隅にその名があるかもしれません。
少なくとも私は当時そのような感想を抱きました。

それから月日が経ち、GraalVM のことは忘れてしまっていたわけだが、Spring Boot 3 がリリースされ、試しに使ってみるかと [Spring Boot Initializer](https://start.spring.io/) の Dependencies を眺めていたところ GraalVM Native Support が追加されていたのを目にしました。

いつ頃から追加されていたのかは不明だが、Spring Boot で使えるのであれば実用に耐えうるところまで開発が進んだという目安になるかと思い、GraalVM について調べて Java + Gradle で GraalVM Native Image の最低限の設定がわかったのでそのメモを残します。

ただの Hello World だとリフレクションの扱いが含まれないため、Lombok の `@Slf4j` とロガー (≠ 標準) を使って Hello World を出力する GraalVM Native Image の作成をサンプルとして扱います。

## 環境

Mac Book Air M2

## 準備

### SDKMAN!

複数の JDK を管理する方法はいくつか存在するが、ここでは [SDKMAN!](https://sdkman.io/) を使った手順を紹介します。
SDKMAN! のインストールは公式サイトの [Installation](https://sdkman.io/install) に従い、

```
$ curl -s "https://get.sdkman.io" | bash
```

を実行した後に、

```
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

で SDKMAN! へのパスが通るようになります。

```
$ sdk versoin
```

コマンドでバージョンが表示されるようになっていれば、正常にインストールが行えてます。
使っているシェルが fish などの特殊なシェルの場合は `source` コマンドではなくプラグインを入れる必要があるかもしれません。

- [【環境構築】fish shellを用いたDebian環境にSDKMANおよびJava(JDK)をインストールする方法 - Debimate](https://debimate.jp/2020/08/30/%E3%80%90%E7%92%B0%E5%A2%83%E6%A7%8B%E7%AF%89%E3%80%91fish-shell%E3%82%92%E7%94%A8%E3%81%84%E3%81%9Fdebian%E7%92%B0%E5%A2%83%E3%81%ABsdkman%E3%81%8A%E3%82%88%E3%81%B3javajdk%E3%82%92%E3%82%A4/)

### GraalVM (JDK)

SDKMAN! のインストールが正常に行えたら、GraalVM をインストールします。

インストール可能はバージョンは、

```
$ sdk list java
```

で確認することができます。実行結果は以下のようになります。

```
$ sdk list java
================================================================================
Available Java Versions for macOS ARM 64bit
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 Corretto      |     | 19.0.1       | amzn    |            | 19.0.1-amzn         
               |     | 17.0.5       | amzn    |            | 17.0.5-amzn         
               |     | 11.0.17      | amzn    |            | 11.0.17-amzn        
               |     | 8.0.352      | amzn    |            | 8.0.352-amzn        
 Gluon         |     | 22.1.0.1.r17 | gln     |            | 22.1.0.1.r17-gln    
               |     | 22.1.0.1.r11 | gln     |            | 22.1.0.1.r11-gln    
 GraalVM       |     | 22.3.r19     | grl     | installed  | 22.3.r19-grl        
               |     | 22.3.r17     | grl     |            | 22.3.r17-grl        
               |     | 22.3.r11     | grl     |            | 22.3.r11-grl        
               |     | 22.2.r17     | grl     |            | 22.2.r17-grl        
               |     | 22.2.r11     | grl     |            | 22.2.r11-grl        
               |     | 22.1.0.r17   | grl     |            | 22.1.0.r17-grl      
               |     | 22.1.0.r11   | grl     |            | 22.1.0.r11-grl      
 Java.net      |     | 21.ea.4      | open    |            | 21.ea.4-open        
               |     | 21.ea.3      | open    |            | 21.ea.3-open        
               |     | 21.ea.2      | open    |            | 21.ea.2-open        
               |     | 21.ea.1      | open    |            | 21.ea.1-open        
               |     | 20.ea.30     | open    |            | 20.ea.30-open       
               |     | 20.ea.29     | open    |            | 20.ea.29-open       
               |     | 20.ea.28     | open    |            | 20.ea.28-open       
               |     | 20.ea.27     | open    |            | 20.ea.27-open       
               |     | 20.ea.26     | open    |            | 20.ea.26-open       
               |     | 19.0.1       | open    |            | 19.0.1-open         
 Liberica      |     | 19.0.1.fx    | librca  |            | 19.0.1.fx-librca    
               |     | 19.0.1       | librca  |            | 19.0.1-librca       
               |     | 17.0.5.fx    | librca  |            | 17.0.5.fx-librca    
               |     | 17.0.5       | librca  |            | 17.0.5-librca       
               |     | 11.0.17.fx   | librca  |            | 11.0.17.fx-librca   
               |     | 11.0.17      | librca  |            | 11.0.17-librca      
               |     | 8.0.352.fx   | librca  |            | 8.0.352.fx-librca   
               |     | 8.0.352      | librca  |            | 8.0.352-librca      
 Liberica NIK  |     | 22.3.r17     | nik     |            | 22.3.r17-nik        
               |     | 22.3.r11     | nik     |            | 22.3.r11-nik        
               |     | 22.2.r17     | nik     |            | 22.2.r17-nik        
               |     | 22.2.r11     | nik     |            | 22.2.r11-nik        
 Microsoft     |     | 17.0.5       | ms      |            | 17.0.5-ms           
               |     | 11.0.17      | ms      |            | 11.0.17-ms          
 Oracle        |     | 19.0.1       | oracle  |            | 19.0.1-oracle       
               |     | 17.0.5       | oracle  |            | 17.0.5-oracle       
 SapMachine    |     | 19.0.1       | sapmchn |            | 19.0.1-sapmchn      
               |     | 17.0.5       | sapmchn |            | 17.0.5-sapmchn      
               |     | 11.0.17      | sapmchn |            | 11.0.17-sapmchn     
 Semeru        |     | 18.0.2       | sem     |            | 18.0.2-sem          
               |     | 17.0.4.1     | sem     |            | 17.0.4.1-sem        
               |     | 11.0.16.1    | sem     |            | 11.0.16.1-sem       
 Temurin       |     | 19.0.1       | tem     |            | 19.0.1-tem          
               |     | 17.0.5       | tem     |            | 17.0.5-tem          
               |     | 11.0.17      | tem     |            | 11.0.17-tem         
 Zulu          |     | 19.0.1       | zulu    |            | 19.0.1-zulu         
               |     | 19.0.1.fx    | zulu    |            | 19.0.1.fx-zulu      
               | >>> | 17.0.5       | zulu    | installed  | 17.0.5-zulu         
               |     | 17.0.5.fx    | zulu    |            | 17.0.5.fx-zulu      
               |     | 11.0.17      | zulu    |            | 11.0.17-zulu        
               |     | 11.0.17.fx   | zulu    |            | 11.0.17.fx-zulu     
               |     | 11.0.16.1    | zulu    | local only | 11.0.16.1-zulu      
               |     | 8.0.352      | zulu    |            | 8.0.352-zulu        
               |     | 8.0.352.fx   | zulu    |            | 8.0.352.fx-zulu     
================================================================================
Omit Identifier to install default version 17.0.5-tem:
    $ sdk install java
Use TAB completion to discover available versions
    $ sdk install java [TAB]
Or install a specific version by Identifier:
    $ sdk install java 17.0.5-tem
Hit Q to exit this list view
================================================================================
```

今回インストールするのは Vendor が GraalVM で Identifier にある 22.3.r19-grl になります。
ここは新しいバージョンがリリースされれば異なったバージョンが表示されていると思うので、実際に実行して表示されたバージョンをメモしてください。

インストール方法は、コマンドの実行結果の末尾に表示されていますが、

```
$ sdk install java 22.3.r19-grl
```

で行えます。インストールしただけではパスが通らないため、

```
$ sdk use java 22.3.r19-grl
```

もしくは

```
$ sdk default java 22.3.r19-grl
```

を実行してください。`use` ではコマンドを実行したシェルでだけ設定が有効になり、`default` ではシェルを起動し直しても指定したバージョンの JDK がパスに設定されたままになります。
ここまでの設定が問題なく行えていれば、`java --version` の結果が意図したバージョンになっていると思います。

```
$ java --version
openjdk 19.0.1 2022-10-18
OpenJDK Runtime Environment GraalVM CE 22.3.0 (build 19.0.1+10-jvmci-22.3-b08)
OpenJDK 64-Bit Server VM GraalVM CE 22.3.0 (build 19.0.1+10-jvmci-22.3-b08, mixed mode, sharing)
```

GraalVM CE にパスが通っています。

### GraalVM Updater

次に [GraalVM Updater](https://www.graalvm.org/latest/reference-manual/graalvm-updater/) を使って `native-image` をインストールします。
GraalVM のインストールに成功していれば、`gu` コマンドが使えるようになっているので、それを使って

```
$ gu install native-image
```

で `native-image` がインストールできます。
これを忘れると Gradle Plugin による GraalVM Native Image の生成には成功しますが、`native-image-agent` を使った設定ファイルの生成に失敗するため気をつけてください。

## Hello, World!

まずは、普通に Logger を使って Hello, World! を出力するプログラムを書きます。
[プロジェクトの雛形](https://github.com/eowiz/java-graalvm-sample/commit/cbd7fb72529d12d7b3066559da0f3e5889c09f6c)は IntelliJ を使って生成した前提で説明を進めます。

### 依存ライブラリ

今回は以下のライブラリを使用します。

- [lombok](https://projectlombok.org/)
- [slf4j](https://www.slf4j.org/)
- [logback](https://logback.qos.ch/)

本音を言えば、logback ではなく [log4j2](https://logging.apache.org/log4j/2.x/) を使いたかったのですが、GraalVM Native Image の生成が上手くいかなかったため大人しく logback を使うことにしました。

### build.gradle

[Lombok Gradle Plugin](https://plugins.gradle.org/plugin/io.freefair.lombok) を `plugins` に追加して、`dependencies` に slf4j、logback への依存を設定します。

```groovy
plugins {
  // ...
  
  id "io.freefair.lombok" version "6.6.1"
}

dependencies {
  // ...

  implementation 'org.slf4j:slf4j-api:2.0.6'
  implementation 'ch.qos.logback:logback-core:1.4.5'
  implementation 'ch.qos.logback:logback-classic:1.4.5'
}
```

### Main.java

`Main.java` で `Hello, world!` を標準出力 (`System.out.println`) している箇所を `log.info` で出力するようにします。

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Main {

  public static void main(String[] args) {
    log.info("Hello world!");
  }
}
```

### 実行可能な jar ファイル

GraalVM Native Image で `main` メソッドを実行できるようにするためには、実行可能な jar ファイルを生成しなければなりません。
そのため `build.gradle` に以下の設定を追加します。

```groovy
jar {
	manifest {
		attributes 'Main-Class': 'com.github.eowiz.Main'
	}
}
```

`mainClass` に設定する値は `main` メソッドを含むクラス名と、そのクラスが置かれたパッケージに合わせて修正してください。

これだけで上手くいくかと思いきや slf4j がエラーを吐いてしまった。

```
$ ./gradlew clean build

$ java -jar build/libs/java-graalvm-sample-1.0-SNAPSHOT.jar
Exception in thread "main" java.lang.NoClassDefFoundError: org/slf4j/LoggerFactory
        at com.github.eowiz.Main.<clinit>(Main.java:5)
Caused by: java.lang.ClassNotFoundException: org.slf4j.LoggerFactory
        at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641)
        at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521)
        ... 1 more
```

上手く動いたGradle のバージョンを上げて `jar` の設定を弄ったら解決した。

```
$ ./gradlew wrapper --gradle-version 7.6 && ./gradlew wrapper
```

`build.gradle` で `jar.from` を設定する。
`duplicatesStrategy` はエラーが出たため追加しました (何故この設定が必要なのかまで調べてません)。

```groovy
jar {
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE

    manifest {
        attributes 'Main-Class': 'com.github.eowiz.Main'
    }

    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
```

これで無事、実行可能な jar ファイルが作れました。

```
$ ./gradlew clean build

$ java -jar build/libs/java-graalvm-sample-1.0-SNAPSHOT.jar
10:51:32.215 [main] INFO com.github.eowiz.Main - Hello world!
```

後で気がつきましたが実行可能な jar が作れるような設定がなくても GraalVM Native Image は作れました。

### Gradle plugin for GraalVM Gradle PluginNative Image building

ここからが本題。
Gradle で GraalVM Native Image を作成するためのプラグインはいくつかあるが、公式が [Gradle Plugin](https://graalvm.github.io/native-build-tools/latest/gradle-plugin.html) を配布しているのでそれを使う。

```groovy
plugins {
    id 'org.graalvm.buildtools.native' version '0.9.19'
}
```

これで `nativeRun`、`nativeCompile` がタスクに追加される。
あとは `nativeRun` を実行するだけと思いきやタスクが失敗する。

```
$ ./gradlew nativeRun

> Task :nativeCompile FAILED
[native-image-plugin] GraalVM Toolchain detection is enabled
[native-image-plugin] GraalVM uses toolchain detection. Selected:
[native-image-plugin]    - language version: 19
[native-image-plugin]    - vendor: GraalVM Community
[native-image-plugin]    - runtime version: 19.0.1+10-jvmci-22.3-b08
[native-image-plugin] Native Image executable path: /Users/eowiz/.sdkman/candidates/java/22.3.r19-grl/lib/svm/bin/native-image
Error: Please specify class (or <module>/<mainclass>) containing the main entry point method. (see --help)

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':nativeCompile'.
> Process 'command '/Users/eowiz/.sdkman/candidates/java/22.3.r19-grl/bin/native-image'' finished with non-zero exit value 1

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 313ms
6 actionable tasks: 1 executed, 5 up-to-date
```

これを解決するためには plugin に `application` を追加して `mainClass` を設定します。

```groovy
plugins {
  // ...
  id 'application'
}

application {
    mainClass = 'com.github.eowiz.Main'
}
```

これでめでたく GraalVM Native Image が作れるようになります。

```
./gradlew nativeRun

> Task :nativeCompile
[native-image-plugin] GraalVM Toolchain detection is enabled
[native-image-plugin] GraalVM uses toolchain detection. Selected:
[native-image-plugin]    - language version: 19
[native-image-plugin]    - vendor: GraalVM Community
[native-image-plugin]    - runtime version: 19.0.1+10-jvmci-22.3-b08
[native-image-plugin] Native Image executable path: /Users/eowiz/.sdkman/candidates/java/22.3.r19-grl/lib/svm/bin/native-image
========================================================================================================================
GraalVM Native Image: Generating 'java-graalvm-sample' (executable)...
========================================================================================================================
[1/7] Initializing...                                                                                    (3.5s @ 0.18GB) :nativeCompile
 Version info: 'GraalVM 22.3.0 Java 19 CE'
 Java version info: '19.0.1+10-jvmci-22.3-b08'
 C compiler: cc (apple, arm64, 14.0.0)
 Garbage collector: Serial GC
[2/7] Performing analysis...  [************]                                                            (10.3s @ 2.03GB) :nativeCompile
   5,378 (82.81%) of  6,494 classes reachable
   8,003 (59.32%) of 13,492 fields reachable
  25,355 (51.14%) of 49,575 methods reachable
     162 classes,    87 fields, and   731 methods registered for reflection
      61 classes,    61 fields, and    55 methods registered for JNI access
       5 native libraries: -framework CoreServices, -framework Foundation, dl, pthread, z
[3/7] Building universe...                                                                               (1.4s @ 0.67GB) :nativeCompile
[4/7] Parsing methods...      [*]                                                                        (1.1s @ 2.10GB) :nativeCompile
[5/7] Inlining methods...     [***]                                                                      (0.7s @ 2.91GB) :nativeCompile
[6/7] Compiling methods...    [***]                                                                      (8.2s @ 4.33GB) :nativeCompile
[7/7] Creating image...                                                                                  (2.3s @ 0.88GB) :nativeCompile
  10.49MB (46.36%) for code area:    15,359 compilation units
  11.66MB (51.53%) for image heap:  155,267 objects and 8 resources
 488.63KB ( 2.11%) for other data
  22.62MB in total
------------------------------------------------------------------------------------------------------------------------
Top 10 packages in code area:                               Top 10 object types in image heap:
 790.41KB java.util                                            2.21MB byte[] for code metadata
 495.60KB java.lang.invoke                                     1.49MB java.lang.String
 471.84KB c.s.org.apache.xerces.internal.impl.xs.traversers    1.34MB byte[] for general heap data
 415.56KB com.sun.org.apache.xerces.internal.impl              1.19MB java.lang.Class
 413.91KB com.sun.crypto.provider                            978.70KB byte[] for java.lang.String
 412.45KB java.lang                                          513.00KB int[][]
 288.50KB java.util.concurrent                               420.16KB com.oracle.svm.core.hub.DynamicHubCompanion
 282.70KB com.sun.org.apache.xerces.internal.impl.xs         289.15KB java.lang.String[]
 270.74KB java.text                                          278.58KB java.util.concurrent.ConcurrentHashMap$Node
 257.52KB java.util.regex                                    259.11KB java.lang.Object[]
   6.39MB for 234 more packages                                2.68MB for 1232 more object types
------------------------------------------------------------------------------------------------------------------------
                        0.7s (2.5% of total time) in 21 GCs | Peak RSS: 5.00GB | CPU load: 4.40
------------------------------------------------------------------------------------------------------------------------
Produced artifacts:
 /Users/eowiz/ghq/github.com/eowiz/java-graalvm-sample/build/native/nativeCompile/java-graalvm-sample (executable)
 /Users/eowiz/ghq/github.com/eowiz/java-graalvm-sample/build/native/nativeCompile/java-graalvm-sample.build_artifacts.txt (txt)
========================================================================================================================
Finished generating 'java-graalvm-sample' in 29.3s.
    [native-image-plugin] Native Image written to: /Users/eowiz/ghq/github.com/eowiz/java-graalvm-sample/build/native/nativeCompile

> Task :nativeRun
11:15:46.577 [main] INFO com.github.eowiz.Main - Hello world!

BUILD SUCCESSFUL in 31s
7 actionable tasks: 3 executed, 4 up-to-date
```

`nativeRun` ではなくコンパイルして実行可能ファイルを実行する場合は、

```
$ ./gradlew nativeCompile
```

を実行すると `build/native/nativeCompile` にファイルが作られるので、

```
$ build/native/nativeCompile/java-graalvm-sample
11:20:26.079 [main] INFO com.github.eowiz.Main - Hello world!
```

で動作を確認できます。

ここまで設定を行うとリポジトリは[このように](https://github.com/eowiz/java-graalvm-sample/tree/a60d85a2dd8296c45c263af4d775136a718086bf)なっていると思います。

これで SLF4J + Logback で上手く動いているように感じてしまいますが、ここでは話が終わらないわけです。
logback 使ってるのに `logback.xml` 書いてないじゃん m9(^Д^) と指を指されてしまうので `src/main/resources` に `logback.xml` を置いて実行するとログに関する設定が反映されないことが判明します。
`logback.xml` に以下の設定を記述して、`./gradlew run` と `./gradlew nativeRun` の結果を比べてみましょう。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] %-5p [%t] %c: %m%n</pattern>
    </encoder>
  </appender>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

まずは `./gradlew run`。

```
[2023-01-07 11:39:34.817] INFO  [main] com.github.eowiz.Main: Hello world!
```

`logback.xml` の内容が反映されてます。
次に `./gradlew nativeRun`。

```
11:40:33.771 [main] INFO com.github.eowiz.Main - Hello world!
```

`logback.xml` 設定する前と変わっていません。
こういうことです…。

さて、`logback.xml` の設定を `nativeRun` で実行したときにも反映させるためにはどうすればよいでしょうか。
答えは `resource-config.json` と `reflect-config.json` を `src/main/resources/META-INF/native-image/com.github.eowiz` に置きます。
`com.github.eowiz` の部分はパッケージ構成によって変更してください。
`resource-config.json` の中身は、

```json
{
  "resources": {
    "includes": [
      {
        "pattern": "\\Qlogback.xml\\E"
      }
    ]
  },
  "bundles": []
}
```

とし、`reflect-config.json` の中身は、

```json
[
  {
    "condition": {
      "typeReachable": "ch.qos.logback.classic.LoggerContext"
    },
    "name": "ch.qos.logback.core.ConsoleAppender",
    "allPublicConstructors": true
  },
  {
    "name": "ch.qos.logback.core.ConsoleAppender",
    "queryAllPublicMethods": true,
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.encoder.PatternLayoutEncoder",
    "queryAllPublicMethods": true,
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.core.pattern.PatternLayoutEncoderBase",
    "methods": [
      {
        "name": "setPattern",
        "parameterTypes": [
          "java.lang.String"
        ]
      }
    ]
  },
  {
    "name": "ch.qos.logback.core.encoder.LayoutWrappingEncoder",
    "methods": [
      {
        "name": "setParent",
        "parameterTypes": [
          "ch.qos.logback.core.spi.ContextAware"
        ]
      }
    ]
  },
  {
    "name": "ch.qos.logback.core.OutputStreamAppender",
    "methods": [
      {
        "name": "setEncoder",
        "parameterTypes": [
          "ch.qos.logback.core.encoder.Encoder"
        ]
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.pattern.DateConverter",
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LevelConverter",
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LineSeparatorConverter",
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.pattern.LoggerConverter",
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.pattern.MessageConverter",
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  },
  {
    "name": "ch.qos.logback.classic.pattern.ThreadConverter",
    "methods": [
      {
        "name": "<init>",
        "parameterTypes": []
      }
    ]
  }
]
```

とします。
GraalVM では Native Image を作成するために AOT (Ahead Of Time) コンパイルが行われるため、リフレクションなどで実行時に動的に読み込まれるクラスについては事前に設定ファイルに記述しておく必要があり、その設定ファイルが `reflect-config.json` となってます。
また、リフレクションで読み込まれるクラスだけでなく、リソースについても設定ファイルに記述しておく必要があり、こちらが `resource-config.json` に対応するわけです。
これら2つ以外にも設定ファイルがありますが、今回必要になるのはこの2つだけです。

- [Reflection Use in Native Images](https://www.graalvm.org/22.0/reference-manual/native-image/Reflection/)
- [Accessing Resources in Native Images](https://www.graalvm.org/22.1/reference-manual/native-image/Resources/)

ここで疑問に思うのが Native Image を正常に動作させるための設定はどのようにすればわかるか、ということです。
設定方法は2つあって、

- 実行してみて実行時エラーが発生したらリフレクションで読み込みに失敗したクラスを設定に追加する
  - 基本的には `java.lang.ClassNotFoundException` が発生
  - 読み込みに失敗したクラスを設定ファイルに追加
  - 再実行して失敗したら設定ファイルに追加を動くようになるまで繰り返す
- `native-image-agent` を使って設定ファイルを自動生成する

のいずれか、もしくは両方で常に正常に動作する設定を探していくことになります。
この後、紹介するが `native-image-agent` で設定ファイルを生成する方法は限界があるため、多くの実用的なアプリケーションでは両方の手段を取ることになります。
先ほど載せた設定ファイルは手動でエラーが無くなるまでトライ&エラーして作りました。

`resource-config.json` と `reflect-config.json` は `src/main/resources/META-INF/native-image/com.github.eowiz` ではなく `src/main/resources/META-INF` 直下に置いても動作します。
しかし、設定が上書きされる可能性があるため、特に理由がなければ `groupID`、`artifactID` まで含めた方が安全だと思われます。

- [Native Image Build Configuration](https://www.graalvm.org/22.0/reference-manual/native-image/BuildConfiguration/)

### Agent

設定ファイルの作成は以下のタスクで行えます。

```
$ ./gradlew -Pagent run
$ ./gradlew metadataCopy --task run --dir src/main/resources/META-INF/native-image/com.github.eowiz
```

- [Assisted Configuration with Tracing Agent](https://www.graalvm.org/22.0/reference-manual/native-image/Agent/)
- [Reflection support and running with the native agent](https://graalvm.github.io/native-build-tools/latest/gradle-plugin.html#agent-support)

上記のコマンドを実行すると `src/main/resources/META-INF/native-image/com.github.eowiz` 配下に設定ファイルが生成されます。
中身を見ると先ほど作成した設定ファイルと同じような内容のファイルが生成されます。
これにて一件落着。次は Spring Boot で GraalVM Native Image の作成を試してみます。

```
$ ./gradlew nativeRun

> Task :nativeRun
[2023-01-07 15:52:13.817] INFO  [main] com.github.eowiz.Main: Hello world!
```

