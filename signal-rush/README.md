# SIGNAL RUSH の構造とメンテナンス

## 読み始める場所

実装は [index.html](index.html) の1ファイル。ゲームのルールや分岐の詳細はコードを正本とし、この文書は修正箇所を探すために使う。

| 対象 | 探す場所 |
|---|---|
| 配色・余白・モバイル表示 | head内のstyle。状態別の配色はdata-stateのセレクター |
| 表示領域・操作部品 | main。arenaが入力領域、trialsが計測結果 |
| ゲーム進行 | activate()、beginRound()、finishRound() |
| タップ・キー・中断 | script末尾のイベント登録 |
| 記録保存 | storageKey、起動時の読み込み、finishRound()の保存 |

## 状態とデータ

stateが現在の進行、scoresが今回の成功した計測値（ミリ秒）の配列。DOMは表示先として使い、進行の判定にはstateを使う。display()が状態と表示を同期する。

| 状態 | 意味 | 入力したとき |
|---|---|---|
| idle | 初期画面 | 待機を開始 |
| waiting | ランダムな待ち時間 | フライングとしてearlyへ |
| go | 合図表示中・計測中 | 記録を確定 |
| result | 1回分の結果 | 次の回の待機を開始 |
| early | フライング | 同じ回の待機を開始 |
| paused | 計測中のタブ非表示・ウィンドウのフォーカス喪失 | 同じ回の待機を開始 |
| complete | 5回完了 | 今回の記録を初期化して待機を開始 |

scoresへの追加はfinishRound()だけが行う。フライング・中断では追加しない。reset()はscoresを消すがbestは維持する。

## 1回の計測

1. activate()からbeginRound()へ入り、waitingを表示する。
2. 1,600〜4,600ms未満のランダムなタイマーで待機する。
3. requestAnimationFrame内でgoを表示し、performance.now()をstartedAtに保存する。
4. 次の有効な入力でfinishRound()を呼び、経過時間を整数ミリ秒に丸めてscoresへ追加する。
5. 5回未満ならresult、5回なら平均値を計算してcompleteを表示する。

clearPending()はタイマーと描画フレーム予約を両方キャンセルする。待機中のリセット・フライング・中断から古い合図が発火しないようにするための処理。

計測開始は合図のDOM更新直後。画面に実際に表示された瞬間との一致は保証しない。表示・入力機器・ブラウザの遅延を含む遊び用の数値。

## 入力経路

| 入力 | 経路・意図 |
|---|---|
| タップ・左クリック | pointerdownでactivate()。押した時点で反応させる |
| 通常のclick | pointerdownとの二重処理を避けるため無視 |
| detailが0のclick | 支援技術などからの操作としてactivate() |
| Space・Enter | documentのkeydownで既定動作を抑止してactivate()。長押しのrepeatは無視 |
| リセット上のSpace・Enter | リセットボタンの標準動作を使う |
| タブ非表示・ウィンドウblur | waiting/goのときだけ計測を中断 |

キーとポインターの処理を変更したときは、1入力で状態が二度進まないことを確認する。

## ベスト記録

localStorageのsignal-rush.best-average.v1に、5回平均の整数ミリ秒を文字列で保存する。数値が小さいほど良い。起動時は正の有限数だけを読み込む。保存が禁止されている場合も、そのページ内ではbestを更新してプレイを続行する。

保存はブラウザ・サイトのオリジン単位。端末間の同期は行わない。ゲームを別のURLやローカルファイルから開いた場合は、同じ保存領域になるとは限らない。

## よくある変更

| 変更したいもの | 関連箇所・確認点 |
|---|---|
| 計測回数 | 5の固定値がrenderScores()とfinishRound()、表示文、HTMLにある。まとめて変更する |
| 待機時間 | beginRound()の1600と3000 |
| 合図の色 | CSSのdata-state="go"。文字の合図も表示する |
| ベストの基準 | finishRound()の平均計算・比較と保存形式 |
| リトライのルール | activate()とreset()。scoresをどこで保持・初期化するか |
| 公開先 | 下記のGitHubとSitesの同期関係 |

## 確認項目と現状

自動検証ではフライング時の記録維持、5回計測、平均値、ベスト更新、再プレイ、リセットをDOMの代替環境で確認した。ブラウザでの実操作確認は未実施。

修正後は、実端末でタップ・キーの二重入力、長押し、待機中のリセット、合図後のタブ切り替え、完了後の再プレイ、再読み込み後のベスト保持を確認する。localStorageを利用できない環境でもプレイできることを確認する。

## ソースと公開先

- GitHubの管理先はisikoro1/gamesのsignal-rush/。
- デプロイ先は https://signal-rush-mwtczb2n.oliveechidna1.chatgpt.site で、所有者のChatGPTログインが必要。
- 現在のGitHubとSitesは別のソース管理先。GitHubへのコミットだけではSitesに自動反映されない。ゲームを変更したときは同じ内容をSitesにも反映して再デプロイする。
- 元のSites用チェックアウトではdist/index.htmlに同じHTMLを配置している。
