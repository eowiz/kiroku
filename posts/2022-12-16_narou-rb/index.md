---
title: Narou.rb on Apple M2
description: Narou.rb を Apple M2 で動かすための手順です。
tags:
  - Kindle
  - なろう
  - Narou.rb
---

Kindle Paperwhite を買ってしばらくはなろう小説を読んでいたが、最近すっかり読まなくなってしまった。
久しぶりに読みたいタイトルがあったので MacBook Air (Apple M2) にも Narou.rb を入れたのでその手順のメモを残しておく。

## 環境

###

MacBook Air (Apple M2)

## 手順

[Narou.rb 説明書](https://github.com/whiteleaf7/narou/wiki)に従って進めます。
macOS でインストールする場合の詳細が書かれていないため、その部分に関して補足します。

### rbenv

macOS にはデフォルトで Ruby がインストールされていますが、`gem install narou` を実行すると

```
$ gem install narou
Fetching mime-types-data-3.2022.0105.gem
Fetching rubyzip-2.3.2.gem
Fetching termcolorlight-1.1.1.gem
Fetching mime-types-3.4.1.gem
Fetching mail-2.6.6.gem
Fetching pony-1.13.1.gem
Fetching diff-lcs-1.5.0.gem
Fetching rack-2.2.4.gem
Fetching tilt-2.0.11.gem
Fetching rack-protection-2.2.3.gem
Fetching ruby2_keywords-0.0.5.gem
Fetching mustermann-2.0.2.gem
Fetching sinatra-2.2.3.gem
Fetching multi_json-1.15.0.gem
Fetching sinatra-contrib-2.2.3.gem
Fetching ffi-1.15.5.gem
Fetching sassc-2.4.0.gem
Fetching temple-0.9.1.gem
Fetching haml-5.2.2.gem
Fetching memoist-0.11.0.gem
Fetching systemu-2.6.5.gem
Fetching erubis-2.7.0.gem
Fetching open_uri_redirections-0.2.1.gem
Fetching concurrent-ruby-1.1.10.gem
Fetching narou-3.8.2.gem
Fetching i18n-1.12.0.gem
Fetching tzinfo-2.0.5.gem
Fetching activesupport-7.0.4.gem
Fetching unicode-display_width-1.8.0.gem
Fetching webrick-1.7.0.gem
Fetching psych-4.0.6.gem
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.
```

権限が原因で失敗します。管理者権限で実行すると Ruby のバージョンが原因で失敗します。

```
$ sudo gem install narou
Password:
Fetching rack-2.2.4.gem
Fetching diff-lcs-1.5.0.gem
Fetching mime-types-3.4.1.gem
Fetching pony-1.13.1.gem
Fetching mime-types-data-3.2022.0105.gem
Fetching mail-2.6.6.gem
Fetching termcolorlight-1.1.1.gem
Fetching rubyzip-2.3.2.gem
Fetching tilt-2.0.11.gem
Fetching rack-protection-2.2.3.gem
Fetching ruby2_keywords-0.0.5.gem
Fetching mustermann-2.0.2.gem
Fetching sinatra-2.2.3.gem
Fetching multi_json-1.15.0.gem
Fetching sinatra-contrib-2.2.3.gem
Fetching ffi-1.15.5.gem
Fetching sassc-2.4.0.gem
Fetching temple-0.9.1.gem
Fetching haml-5.2.2.gem
Fetching memoist-0.11.0.gem
Fetching systemu-2.6.5.gem
Fetching erubis-2.7.0.gem
Fetching open_uri_redirections-0.2.1.gem
Fetching concurrent-ruby-1.1.10.gem
Fetching i18n-1.12.0.gem
Fetching tzinfo-2.0.5.gem
Fetching activesupport-7.0.4.gem
Fetching unicode-display_width-1.8.0.gem
Fetching webrick-1.7.0.gem
Fetching psych-4.0.6.gem
Fetching narou-3.8.2.gem
Successfully installed termcolorlight-1.1.1
RubyZip 3.0 is coming!
**********************

The public API of some Rubyzip classes has been modernized to use named
parameters for optional arguments. Please check your usage of the
following classes:
  * `Zip::File`
  * `Zip::Entry`
  * `Zip::InputStream`
  * `Zip::OutputStream`

Please ensure that your Gemfiles and .gemspecs are suitably restrictive
to avoid an unexpected breakage when 3.0 is released (e.g. ~> 2.3.0).
See https://github.com/rubyzip/rubyzip for details. The Changelog also
lists other enhancements and bugfixes that have been implemented since
version 2.3.0.
Successfully installed rubyzip-2.3.2
Successfully installed mime-types-data-3.2022.0105
Successfully installed mime-types-3.4.1
Successfully installed mail-2.6.6
Successfully installed pony-1.13.1
Successfully installed diff-lcs-1.5.0
Successfully installed rack-2.2.4
Successfully installed tilt-2.0.11
Successfully installed rack-protection-2.2.3
Successfully installed ruby2_keywords-0.0.5
Successfully installed mustermann-2.0.2
Successfully installed sinatra-2.2.3
Successfully installed multi_json-1.15.0
Successfully installed sinatra-contrib-2.2.3
Building native extensions. This could take a while...
Successfully installed ffi-1.15.5
Building native extensions. This could take a while...
Successfully installed sassc-2.4.0
Successfully installed temple-0.9.1
Successfully installed haml-5.2.2
Successfully installed memoist-0.11.0
Successfully installed systemu-2.6.5
Successfully installed erubis-2.7.0
Successfully installed open_uri_redirections-0.2.1
Successfully installed concurrent-ruby-1.1.10
Successfully installed i18n-1.12.0
Successfully installed tzinfo-2.0.5
ERROR:  Error installing narou:
        The last version of activesupport (>= 6.1, < 8.0) to support your Ruby & RubyGems was 6.1.7. Try installing it with `gem install activesupport -v 6.1.7` and then running the current command again
        activesupport requires Ruby version >= 2.7.0. The current ruby version is 2.6.10.210.
```

Ruby のバージョンを 2.7.0 以上にアップデートすることで解決しそうですが、大人しく推奨されている [rbenv](https://github.com/rbenv/rbenv) で Ruby をインストールします。
Homebrew を導入している場合は、以下のコマンドでインストールできます。

```
$ brew install rbenv ruby-build
```

Homebrew を使っていない場合は、他の方法でインストールしてください。
rbenv がインストールできたらインストール可能な Ruby のバージョンを確認します。

```
$ rbenv install --list
2.7.7
3.0.5
3.1.3
jruby-9.4.0.0
mruby-3.1.0
picoruby-3.0.0
rbx-5.0
truffleruby-22.3.0
truffleruby+graalvm-22.3.0

Only latest stable releases for each Ruby implementation are shown.
Use 'rbenv install --list-all / -L' to show all local versions.
```

最新の `3.1.3` をインストールします。

```
rbenv install 3.1.3

To follow progress, use 'tail -f /var/folders/23/j_t4c7z10r95n4xtd9690jsh0000gn/T/ruby-build.20221216230514.73120.log' or pass --verbose
Downloading openssl-3.0.7.tar.gz...
-> https://dqw8nmjcqpjn7.cloudfront.net/83049d042a260e696f62406ac5c08bf706fd84383f945cf21bd61e9ed95c396e
Installing openssl-3.0.7...
Installed openssl-3.0.7 to /Users/<username>/.rbenv/versions/3.1.3

Downloading ruby-3.1.3.tar.gz...
-> https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.3.tar.gz
Installing ruby-3.1.3...
ruby-build: using readline from homebrew
ruby-build: using gmp from homebrew
Installed ruby-3.1.3 to /Users/<username>/.rbenv/versions/3.1.3


NOTE: to activate this Ruby version as the new default, run: rbenv global 3.1.3
```

次に rbenv でインストールした Ruby にパスを通すために、

```
$ rbenv init
```

を実行し、表示された指示に従ってシェルの設定を追加します。
設定後はシェルに応じて設定ファイルを再読み込みします。

```
$ source .zshrc
```

ここまでの設定が正しく行えているならば、インストールしたバージョンに応じて `rbenv global` で最新の Ruby が使えるようになります。

```
$ rbenv global 3.1.3

$ ruby --version
ruby --version
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [arm64-darwin22]
```

Ruby のバージョンが変わらない場合はシェルを再起動してください。

### narou

Ruby がインストールできたので Narou.rb 本体のインストールをします。

```
$ gem install narou

gem install narou
Fetching systemu-2.6.5.gem
Fetching rack-protection-2.2.3.gem
Fetching tilt-2.0.11.gem
Fetching webrick-1.7.0.gem
Fetching unicode-display_width-1.8.0.gem
Fetching termcolorlight-1.1.1.gem
Fetching rack-2.2.4.gem
Fetching mustermann-2.0.2.gem
Fetching sinatra-2.2.3.gem
Fetching multi_json-1.15.0.gem
Fetching sinatra-contrib-2.2.3.gem
Fetching ffi-1.15.5.gem
Fetching sassc-2.4.0.gem
Fetching rubyzip-2.3.2.gem
Fetching mime-types-data-3.2022.0105.gem
Fetching mime-types-3.4.1.gem
Fetching mail-2.6.6.gem
Fetching pony-1.13.1.gem
Fetching open_uri_redirections-0.2.1.gem
Fetching memoist-0.11.0.gem
Fetching temple-0.9.1.gem
Fetching haml-5.2.2.gem
Fetching erubis-2.7.0.gem
Fetching diff-lcs-1.5.0.gem
Fetching concurrent-ruby-1.1.10.gem
Fetching tzinfo-2.0.5.gem
Fetching i18n-1.12.0.gem
Fetching narou-3.8.2.gem
Fetching activesupport-7.0.4.gem
Successfully installed webrick-1.7.0
Successfully installed unicode-display_width-1.8.0
Successfully installed tilt-2.0.11
Successfully installed termcolorlight-1.1.1
Successfully installed systemu-2.6.5
Successfully installed rack-2.2.4
Successfully installed rack-protection-2.2.3
Successfully installed mustermann-2.0.2
Successfully installed sinatra-2.2.3
Successfully installed multi_json-1.15.0
Successfully installed sinatra-contrib-2.2.3
Building native extensions. This could take a while...
Successfully installed ffi-1.15.5
Building native extensions. This could take a while...
Successfully installed sassc-2.4.0
RubyZip 3.0 is coming!
**********************

The public API of some Rubyzip classes has been modernized to use named
parameters for optional arguments. Please check your usage of the
following classes:
  * `Zip::File`
  * `Zip::Entry`
  * `Zip::InputStream`
  * `Zip::OutputStream`

Please ensure that your Gemfiles and .gemspecs are suitably restrictive
to avoid an unexpected breakage when 3.0 is released (e.g. ~> 2.3.0).
See https://github.com/rubyzip/rubyzip for details. The Changelog also
lists other enhancements and bugfixes that have been implemented since
version 2.3.0.
Successfully installed rubyzip-2.3.2
Successfully installed mime-types-data-3.2022.0105
Successfully installed mime-types-3.4.1
Successfully installed mail-2.6.6
Successfully installed pony-1.13.1
Successfully installed open_uri_redirections-0.2.1
Successfully installed memoist-0.11.0
Successfully installed temple-0.9.1
Successfully installed haml-5.2.2
Successfully installed erubis-2.7.0
Successfully installed diff-lcs-1.5.0
Successfully installed concurrent-ruby-1.1.10
Successfully installed tzinfo-2.0.5
Successfully installed i18n-1.12.0
Successfully installed activesupport-7.0.4
************************************************************

3.8.2: 2022/09/10
-----------------
#### 修正内容
- フォルダが存在しない場合に自動で作成する様に修正

************************************************************
Successfully installed narou-3.8.2
Parsing documentation for webrick-1.7.0
Installing ri documentation for webrick-1.7.0
Parsing documentation for unicode-display_width-1.8.0
Installing ri documentation for unicode-display_width-1.8.0
Parsing documentation for tilt-2.0.11
Installing ri documentation for tilt-2.0.11
Parsing documentation for termcolorlight-1.1.1
Installing ri documentation for termcolorlight-1.1.1
Parsing documentation for systemu-2.6.5
Installing ri documentation for systemu-2.6.5
Parsing documentation for rack-2.2.4
Installing ri documentation for rack-2.2.4
Parsing documentation for rack-protection-2.2.3
Installing ri documentation for rack-protection-2.2.3
Parsing documentation for mustermann-2.0.2
Installing ri documentation for mustermann-2.0.2
Parsing documentation for sinatra-2.2.3
Installing ri documentation for sinatra-2.2.3
Parsing documentation for multi_json-1.15.0
Installing ri documentation for multi_json-1.15.0
Parsing documentation for sinatra-contrib-2.2.3
Installing ri documentation for sinatra-contrib-2.2.3
Parsing documentation for ffi-1.15.5
Installing ri documentation for ffi-1.15.5
Parsing documentation for sassc-2.4.0
Installing ri documentation for sassc-2.4.0
Parsing documentation for rubyzip-2.3.2
Installing ri documentation for rubyzip-2.3.2
Parsing documentation for mime-types-data-3.2022.0105
Installing ri documentation for mime-types-data-3.2022.0105
Parsing documentation for mime-types-3.4.1
Installing ri documentation for mime-types-3.4.1
Parsing documentation for mail-2.6.6
Installing ri documentation for mail-2.6.6
Parsing documentation for pony-1.13.1
Installing ri documentation for pony-1.13.1
Parsing documentation for open_uri_redirections-0.2.1
Installing ri documentation for open_uri_redirections-0.2.1
Parsing documentation for memoist-0.11.0
Installing ri documentation for memoist-0.11.0
Parsing documentation for temple-0.9.1
Installing ri documentation for temple-0.9.1
Parsing documentation for haml-5.2.2
Installing ri documentation for haml-5.2.2
Parsing documentation for erubis-2.7.0
Installing ri documentation for erubis-2.7.0
Parsing documentation for diff-lcs-1.5.0
Installing ri documentation for diff-lcs-1.5.0
Parsing documentation for concurrent-ruby-1.1.10
Installing ri documentation for concurrent-ruby-1.1.10
Parsing documentation for tzinfo-2.0.5
Installing ri documentation for tzinfo-2.0.5
Parsing documentation for i18n-1.12.0
Installing ri documentation for i18n-1.12.0
Parsing documentation for activesupport-7.0.4
Installing ri documentation for activesupport-7.0.4
Parsing documentation for narou-3.8.2
Installing ri documentation for narou-3.8.2
Done installing documentation for webrick, unicode-display_width, tilt, termcolorlight, systemu, rack, rack-protection, mustermann, sinatra, multi_json, sinatra-contrib, ffi, sassc, rubyzip, mime-types-data, mime-types, mail, pony, open_uri_redirections, memoist, temple, haml, erubis, diff-lcs, concurrent-ruby, tzinfo, i18n, activesupport, narou after 22 seconds
29 gems installed
```

:tada:

### AozoraEpub3

[AozoraEpub3](https://w.atwiki.jp/hmdev/) をダウンロードして解凍します。

### Kindle Previewer 3

[Kindle Previewer 3](https://kdp.amazon.co.jp/ja_JP/help/topic/G202131170) をインストールする。
インストール後、kindlegen は[ここ](https://app.k-server.info/game/mac_kindlegen/)を参考に Kindle Previewer 3 から抜き出し、AozoraEpub3 と同じディレクトリに置く。

### その後…

あとは公式の手順書に従って使えるようになります。

それでは、よいなろうライフを :wave:
