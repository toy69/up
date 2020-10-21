# RelaxNGと
# ParserCombinatorと
## @opaopa6969
---
# 自己紹介

* Z80おじさん
* 
---
# RelaxNGについて
* XMLのスキーマ言語
    * XML文書の構造を定義します
* DTDやXMLSchemaもその仲間ですが
* わかりやすくて表現力が高い
* モジューラビリティが高い
  * [XHTML](http://www.thaiopensource.com/relaxng/xhtml/)
---

# RelaxNG文法
* http://relaxng.org/tutorial-20011203.html （英語)
* http://www.kohsuke.org/relaxng/tutorial.ja.html （MS932)
* 1時間ほどで学べます！
---

|RelaxNG文法要素|説明|
|--------------|---|
|ZeroOrMore|*|
|OneOrMore|+|
|Optional|?|
|Choice|or|
|Interleave|順不同ですべてにマッチ|
|combine|文法の合成|
|define|文法の定義|
|ref|定義済み文法の呼び出し|

---
# Unlaxer
https://bitbucket.org/opaopa6969/unlaxer
* 安楽さ～（沖縄っぽいイントネーションで）
* java8で組まれたRelaxNGっぽいparser combinator
* Lexer(tokenizer)無し/parserのみ(PEGと一緒)
* 無限先読み
* バックトラックあり(begin/commit/rollback)
* 後方参照あり
* parse時にparserからアクセスできるcontextTreeを作る

---

# 日々のパーサ開発

* 小さなパーサ → stateマシンを作って
  * なんだかんだで面倒

* 大きなパーサ → javaccとかANTLRとか
  * 定義さえきちっとできればバグなく作れる
  * ANTLR worksとかXtextは素晴らしいよね
  * but,半年後のメンテナンス時に???ってなる
---
# データ操作におけるパーサの需要
* XMLやJSONがある
* パーサを1から書かなくてもライブラリがある
* schemaを定義すれば入力補完やvalidationもOK
  * 後でDEMO
---
# ApplicationにおけるRelaxNGの利用

* 最初にRNGを定義
* xjcやRelaxerを使用してクラス生成
* 生成されたクラスをトラバースしながら実装を行う
* DEMO
---

# Unlaxer開発の背景
* XML(RelaxNG)やJSONで済まない場面もたまにある
* つらい現実からの逃避
  * → パーサ作りはからくり時計みたいで楽しい！
* RelaxNGっぽいパーサコンビネータなら僕でも使えるかも？

---
# Unlaxer開発方針

* 汚くても実用的に
* 文法のデバッグがしやすく
* 正規表現くらいは置き換えられるように
  * 貪欲性の制御も
* Xtextみたいにエディタと連携できたらいいな
* 冗長上等…;

---
# Unlaxerの実装
---

## バックトラック実装
* begin/commit/rollback時ポジションをstackにpush/pop

* [sequence図](https://dl.dropboxusercontent.com/u/2069248/Unlaxer-presentation/parse-sequence.pdf) 

---
### バックトラック実装例 choice

[ChoiceInterface.java](https://bitbucket.org/opaopa6969/unlaxer/src/0dff9485690f41fa14cf4ce92b343d02cfe72641/unlaxer-common/src/main/java/org/unlaxer/parser/combinator/ChoiceInterface.java?at=master&fileviewer=file-view-default)
```java
public interface ChoiceInterface extends Parser{
 
 public default Parsed parse(ParseContext parseContext, TokenKind tokenKind, boolean invertMatch) {
  
  for (Parser parser : getChildren()) {
   parseContext.begin(this);
   Parsed parsed = parser.parse(parseContext, tokenKind, invertMatch);
   
   if (parsed.isSucceeded()) {
    parseContext.commit(this, tokenKind , new ChoiceCommitAction(parser));
    return parsed;
   }
   parseContext.rollback(this);
  }
  return Parsed.FAILED;
 }
}
```
---
## 無限先読みと後方参照


---
### 先読みと参照のサンプル

* [回文パーサ](https://bitbucket.org/opaopa6969/unlaxer/src/0dff9485690f41fa14cf4ce92b343d02cfe72641/unlaxer-common/src/test/java/sample/Usage003_01_Palidrome.java?at=master&fileviewer=file-view-default)
* 五種類のやり方↑
* matchOnly(parser)で先読み
* MatchedTokenParser(lookaheadParser)で先読みでマッチしたtokenにマッチするparserを生成

---
## Unlaxerでのdebug
---
* parser combinatorのデバッグは面倒
* parser自体にブレークポイントを置いたとしても繰り返しマッチする時とか…
* [リスナー](https://bitbucket.org/opaopa6969/unlaxer/src/0dff9485690f41fa14cf4ce92b343d02cfe72641/unlaxer-common/src/main/java/org/unlaxer/listener/CombinedDebugListener.java?at=master&fileviewer=file-view-default)を登録しブレークポイントを置く
* [CommentTest.java](https://bitbucket.org/opaopa6969/unlaxer/src/0dff9485690f41fa14cf4ce92b343d02cfe72641/unlaxer-vocabulary/ebnf/src/test/java/org/unlaxer/vocabulary/ebnf/informally/CommentTest.java?at=master&fileviewer=file-view-default)←ネストコメントのパーサのテスト
* パーサのログを利用して任意の階層でブレークを行うhttps://bitbucket.org/opaopa6969/unlaxer/src/0dff9485690f41fa14cf4ce92b343d02cfe72641/unlaxer-vocabulary/ebnf/src/test/resources/parserTest/org.unlaxer.vocabulary.ebnf.informally.CommentTest/test_testAllMatch_(7,L33).combined.log?at=master&fileviewer=file-view-default

---
## Unlaxer利用してsuggestなど

* [SuggestsCollectorParserTest.java](https://bitbucket.org/opaopa6969/unlaxer/src/0dff9485690f41fa14cf4ce92b343d02cfe72641/unlaxer-common/src/test/java/org/unlaxer/SuggestsCollectorParserTest.java?at=master&fileviewer=file-view-default)

* DEMO

---
# Unlaxer TODO

* notationを作ってparser combinator自動生成
  * とりあえずISO14977->parser combinator自動生成とか
* RelaxNGにおけるNVDLのようなものを作る
* schemaのセントラルリポジトリを
* editor組み込み
---
