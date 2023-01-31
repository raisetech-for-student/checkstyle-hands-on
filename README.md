# 概要

CheckstyleをCIに導入する流れを学ぶためのハンズオンです。

# 前提
- Java 17

# そもそもCheckstyleとは

Checkstyleはインデントが揃っていない、変数名なのにキャメルケースじゃないといったソースコード内の問題を検出してくれます。  

こういったツールのことをlint、linter、静的解析ツールといわれます。  
つまり、CheckstyleはJavaのソースコードのための静的解析ツールと言えます。  

ちなみにCheckstyle以外にもSpotbugsやPMDというツールもあります。  

lintについてはこちらの資料もぜひ参考にしてください。  

https://kiwi-wasp-3c9.notion.site/lint-ffd53c185fe14622ac312ce5e16ad90e  

静的解析ツールを活用することでソースコードのバグの検出漏れをふせぐことができます。  
また、静的解析ツールは初学者がミスをしがちな点を指摘してくれるので新人エンジニアにとってはよい先生になってくれます。

# ハンズオン

## このハンズオンで学ぶこと

このハンズオンでは以下のことを学べます。
- ローカルでCheckstyleを実行する方法
- Checkstyleのレポートの読み方
- CIにCheckstyleを導入し、Pull Requestでチェックをする方法

## 準備

まずはこのハンズオンのリポジトリをforkしてください。  
forkできましたら、ご自身のローカルPCにcloneしてエディタでプロジェクトを開いてください。  

参考: https://docs.github.com/ja/get-started/quickstart/fork-a-repo

## ローカルでCheckstyleを実行

ルートディレクトリ上で下記コマンドを実行してみてください。  

```shell
% ./gradlew checkstyleMain
```

BUILD SUCCESSFULとコンソールに表示されるまで待ちましょう。

## レポートの確認

`./build/reports/checkstyle/main.html`を開きましょう。  
お好きなブラウザでOKです。  

レポートを見るとエラーが無いことがわかります。  

<img width="500" alt="checkstylereport" src="https://user-images.githubusercontent.com/62045457/215107282-8b7334c7-74c3-4d10-8658-ac671fe6c921.png">

## あえてエラーになるようなJavaファイルを作成する

以下のようなパッケージとJavaファイルを作成してください。

src/main/java/com/raisetech/checkstylesample下にSAMPLEというパッケージ名を作ってください。    

SAMPLE配下にクラス名をsampleClassとしたJavaファイルを作成しましょう。

```java
package com.raisetech.checkstylesample.sample;

public class sampleClass {
    private String FirstName;

    private String family_name;

    private int age;

    public String getFirstName() {
        return FirstName;
    }

    public String getFamily_name() {
        return family_name;
    }

    public int getAge() {
        return age;
    }
}
```

ルートディレクトリ上で下記コマンドを実行してみてください。

```shell
% ./gradlew checkstyleMain
```

BUILD SUCCESSFULとコンソールに表示されるまで待ちましょう。  

レポートを確認するとエラー内容が表示されているはずです。  

<img width="500" alt="checkstylereport-with-error" src="https://user-images.githubusercontent.com/62045457/215107342-b7bfffdc-0349-4eed-ae4e-114c3170df76.png">

## エラー内容を読み解く

```shell
Package name 'com.raisetech.checkstylesample.sample' must match pattern '^[a-z]+(\.[a-z][a-z0-9]*)*$'.
```

問題のある箇所はソースコードで言うところのこちらになります。

```java
package com.raisetech.checkstylesample.sample;
```

`'^[a-z]+(\.[a-z][a-z0-9]*)*$'`についてですが、これは文字化けではなく正規表現と呼ばれるものです。  

正規表現が初耳の人はぜひ調べてみてください。  

`'^[a-z]+(\.[a-z][a-z0-9]*)*$'`は小文字の英字から始まり、末尾に「.」と小文字の英字、数字の組み合わせが0回以上続く文字列を表しています。  

Checkstyleのメッセージ内容としては、

「パッケージ名はsampleのような小文字のアルファベットで始まり、その後に小文字のアルファベットや数字、またはドットで続く文字列パターンにマッチするようにしてください」  

という意味合いになります。  

他のエラーメッセージを訳してみると、次のようになります。  
正規表現は意訳しています。  

- 型名のsampleClassは大文字アルファベット始まりになるようにしましょう
- メンバーのFirstNameは小文字始まりのアルファベット始まりになるようにしましょう
- メンバーのfamily_nameはアンダースコアが含まれないようにしましょう
- 8行目のメンバーのageはインデントはスペース4つでなく2つにしましょう

英語&正規表現なので最初はハードルが高いですが、このメッセージを読み解くのはとても大事です。  
繰り返し間違うことで、どの行に問題があるかみるだけでも何が問題かわかるようになってきます。  

## Pull Request上でCheckstyleを実行する

次に、Pull Request上で同様のチェックを行う方法について解説します。  

下記ファイルにPull Requestの作成時、変更をpush時に自動でCheckstyleを実行するワークフローを作成しています。  
https://github.com/raisetech-for-student/checkstyle-hands-on/blob/8506e8aba5331f0c214bf812af76afedd87dc41a/.github/workflows/checkstyle.yml  

ワークフローはGitHub Actionsにより作成しています。

みなさんの手元のソースコードにも同じものがあります。  

ワークフロー内でCheckstyleを実行しているのはこちらです。  

https://github.com/raisetech-for-student/checkstyle-hands-on/blob/8506e8aba5331f0c214bf812af76afedd87dc41a/.github/workflows/checkstyle.yml#L21-L30

では、さきほどのsampleClass.javaをpushしてみましょう。  

```shell
% git checkout -b test/checkstyle
% git add .
% git commit -m "CIでのCheckstyleの動作確認のための変更"
% git push origin head -u
```

test/checkstyleブランチからご自身のmainブランチに対してPull Requestを作成しましょう。  

作成したら、下記キャプチャのようにCIが自動で起動してCheckstyleが実行されます。  

<img width="500" alt="ci-inprogress" src="https://user-images.githubusercontent.com/62045457/215106874-ebc5702f-5dd6-4404-a150-0f8f833139fc.png">

しばらく待つとAll checks have failedとなります。  

<img width="500" alt="ci-failed" src="https://user-images.githubusercontent.com/62045457/215106905-59f9e833-bf08-4fdb-a181-ef1a2f4f1b9c.png">

Detailsをクリックすると遷移先でCheckstyleのエラー内容を見ることができます。  

<img width="500" alt="ci-details" src="https://user-images.githubusercontent.com/62045457/215106954-4500837e-7a3b-48c7-9c42-d8f2fa0b6257.png">

<img width="500" alt="ci-logs" src="https://user-images.githubusercontent.com/62045457/215107028-dbe6c2e8-b635-45e6-b309-c0767802b4bd.png">

また、Files changedを押すと、レポートでみたようなファイル内のどこに問題があるのか、何が問題なのかがひと目で分かります。  

<img width="500" alt="ci-fileschanged" src="https://user-images.githubusercontent.com/62045457/215107105-59b6fa15-6690-4d4a-bf5e-84fef2ad2e91.png">

<img width="500" alt="ci-fileschanged-detail" src="https://user-images.githubusercontent.com/62045457/215107163-051805c0-fdb9-48cf-a0d8-a10b3a54812a.png">

便利ですね。

## 自分のリポジトリにCheckstyleを導入する方法

最後に自身のリポジトリにCheckstyleを導入する方法を2つ紹介します。

### ローカルで実行できるようにする方法

公式ドキュメントにあるように、build.gradleのpluginsにcheckstyleを追加します。  
https://docs.gradle.org/current/userguide/checkstyle_plugin.html  
```groovy
plugins {
    id 'org.springframework.boot' version '2.6.7'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'checkstyle'
}
```
バージョンは資料作成時点のものです。  
それから、build.gradle末尾に下記の記載も追加しておきましょう。

```groovy
checkstyle {
    toolVersion = '10.2'
}
```

また、Checkstyleの設定ファイルも必要です。  

`./config/checkstyle`ディレクトリを作成してその下にcheckstyle.xmlを配置しましょう。  

ファイルの中身についてはハンズオンではgoogleのものを採用しました。  
https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml  

この中身をかえることで独自の設定を適用できます。  

この状態で

```shell
% ./gradlew checkstyleMain
```

を実行するとCheckstyleが実行できます。


## CIで実行する方法

シンプルです。  

このファイルをコピーしていただければいいです。  
https://github.com/raisetech-for-student/checkstyle-hands-on/blob/8506e8aba5331f0c214bf812af76afedd87dc41a/.github/workflows/checkstyle.yml

配置する際には`./github/workflows`を作成して、その配下に置くようにしましょう。  

ワークフローファイルの中身は1行ずつ読んで理解するようにしましょう。  
