# ページロード過程
## 1. HTMLをパースしている最中にscriptエレメントが出現したときどうなる？
- [デモページ](https://www.ktsuchiy.work/index.html)
- scriptエレメント出現時の動作をわかりやすくするため、HTMLを少し細工
- 細工したHTMLは以下の通り。
  ```
  <html>
    <head>
      <script>...（ポイント3）
        window.addEventListener("DOMContentLoaded", function() {
        alert("DOMContentLoaded!");
        });
        window.addEventListener("load", function() {
        alert("load!");
        });
      </script>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>テストページ</title>
        <link rel="stylesheet" href="main.css">
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" ></script>
        <script type="text/javascript" src="script.js?delay=5" ></script> ...（ポイント1）
        <script>alert("HTMLパース再開!");</script>...（ポイント2）
    </head>
    <body>
    〜省略〜
    ```
- （ポイント1）
    - script.jsのレスポンスを悪く（5秒）にしておく -> わかりやすくするため...
    - script.jsの中で実行時にアラートを発生させる -> 実行タイミングをわかりやすくする。
- （ポイント2）
    - scriptエレメントの下行実行時にアラートを発生させる -> HTMLパースの再開タイミングをわかりやすくする。
- （ポイント3）
    - DOMContentLoaded / loadイベントのタイイングを把握するため、このタイミングでもアラートを発生させる。
- [デモ実行！！！](https://www.ktsuchiy.work/index_js_delay.html)
- デモの結果より、、、scriptエレメントが出現すると以下の動作となる！！
    - script.jsを取得する。（このデモでは5秒要する）
    - script.jsを実行する。（実行した旨のアラート表示）
    - script.jsの取得、実行までHTMLパースが止まる
    - script.jsの取得、実行が終わると、HTMLパースが再開する
    - なお、scriptエレメント以降にあるリソース（例えば、画像ファイル）の取得はscript.js取得や実行と並行して実施される
- このデモのように、script.jsの取得や実行に時間を要すと、HTMLパースが遅れ、DOM構築が遅れ、レンダリングが遅れ、描画が遅れる。
- LCPの悪化などにつながってしまう。

## 2. このscript.jsは初期描画に関係しない場合、ページの初期描画を速くする方法は？
- 細工したHTMLは以下の通り。
  ```
  <html>
    <head>
      <script>
        window.addEventListener("DOMContentLoaded", function() {
        alert("DOMContentLoaded!");
        });
        window.addEventListener("load", function() {
        alert("load!");
        });
      </script>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>テストページ</title>
        <link rel="stylesheet" href="main.css">
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" ></script>
    </head>
    <body>
    〜省略〜
        <script type="text/javascript" src="script.js?delay=5" ></script> ...（ポイント1）
    </body>
    ```
- （ポイント1）
    - script.jsの記述場所をbody最下部とする。
- [デモ実行！](https://www.ktsuchiy.work/index_js_delay_last.html)
- デモの結果より、、、
    - 初期描画は速くなった！
    - script.jsの実行、DOMContentLoaded、Loadは遅い
    - 仮にDOMContentLoaded後に実施したいイベントハンドラがある場合、、
    - 表示されるけど、反応しないとか？どんな問題が出るの？？

## 3. DOMContentLoadedを速くする方法はある？
- 細工したHTMLは以下の通り。
  ```
  <html>
    <head>
      <script>
        window.addEventListener("DOMContentLoaded", function() {
        alert("DOMContentLoaded!");
        });
        window.addEventListener("load", function() {
        alert("load!");
        });
      </script>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>テストページ</title>
        <link rel="stylesheet" href="main.css">
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" ></script>
        <script type="text/javascript" src="script.js?delay=5" async></script>...（ポイント1）
    </head>
    <body>
    ```
- （ポイント1）
    - async！！
- [デモ実行！](https://www.ktsuchiy.work/index_js_delay_async.html)
- デモの結果より、、、
    - 初期描画は変わらず速い
    - DOMContentLoadedも速くなった
- asyncにより、このscriptの処理と実行がHTMLパース処理やDOM構築処理から切り離される
- 初期描画に影響なく、DOM/レンダリング/描画に関係しないscriptはasyncで処理を分離/独立させることができる
- 注意点は...
    - DOM/レンダリング/描画に関係するscriptには使わない
    - scriptをダウンロード次第実行する
    - scriptのタイミングや実行順序（他scriptとの）が読めなくなり、最終的なページ描画の一貫性が保てなくなるため
    - では、DOM/レンダリング/描画に関係するscript（初期描画には関係しないが）について、body最下部に記述する以外に良い方法はない？

## 4. 初期描画に関係しないけど、実行タイミング、順序をコントロールする方法（body最下部にscriptタグを記述するより良い方法）？
- 細工したHTMLは以下の通り。
  ```
  <html>
    <head>
      <script>
        window.addEventListener("DOMContentLoaded", function() {
        alert("DOMContentLoaded!");
        });
        window.addEventListener("load", function() {
        alert("load!");
        });
      </script>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>テストページ</title>
        <link rel="stylesheet" href="main.css">
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" ></script>
        <script type="text/javascript" src="script.js?delay=5" defer></script>...（ポイント1）
    </head>
    <body>
    ```
- （ポイント1）
    - defer！！
- [デモ実行！](https://www.ktsuchiy.work/index_js_delay_defer.html)
- デモの結果より、、、
    - script.js取得し始めるタイミングが速い（body最下部記述の場合に比べ）
    - 違いはそれだけ..

## 5. 疑問
- DOMContentLoadedが終わってから、レンダリング、描画じゃないの？？