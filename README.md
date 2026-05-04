# Party Game Web

Firebase Realtime Databaseで同期して遊べる、GitHub Pages向けの静的Webアプリです。ビルドツールやnpmは使わず、`index.html` をそのまま配信して動かします。

## 遊べるゲーム

- ワードウルフ: 少数派のお題を持つ人を会話で探すゲーム
- スパイ探し: 1人または複数人だけ場所を知らない。質問と会話でスパイを見つけるゲーム
- NGワードゲーム: 自分だけ知らないNGワードを言わずに会話するゲーム

## その他ゲーム

- トップ画面の「その他ゲーム」ボタンから、その他ゲーム画面に切り替えられます。
- ガーティックフォンへの外部リンクを追加しています。
- ガーティックフォンのカード画像は `assets/gartic-phone.png` を使います。
- skribbl.io への外部リンクを追加し、カード画像は `assets/skribbl-io.png` を使います。
- 一致するまで終われまテンへの外部リンクを追加し、カード画像は `assets/itchi-made-owarema-ten.png` を使います。

## NGワードゲームの遊び方

1. トップ画面で「NGワードゲーム」を選びます。
2. ホストが部屋を作り、共有URLまたは部屋コードで参加者を集めます。
3. ホストが制限時間とNGワードジャンルを設定して開始します。
4. 各プレイヤーは他の人のNGワードを見ながら、自分のNGワードを言わないように会話します。
5. 誰かが自分のNGワードを言ったら、ホストがその人を「脱落にする」で同期します。
6. 時間終了後、または任意のタイミングでホストが結果発表します。

## NGワードゲームの勝利条件

- 勝者: 結果発表時に脱落していないプレイヤー全員
- 脱落者: ホストにより脱落済みにされたプレイヤー
- 結果画面では勝者、脱落者、全員のNGワードを表示します。

## スパイ探しの遊び方

1. トップ画面で「スパイ探し」を選びます。
2. ホストが部屋を作り、共有URLまたは部屋コードで参加者を集めます。
3. ホストが話し合い時間、場所ジャンル、スパイ人数を設定します。
4. ゲーム開始後、一般人には「場所」と「役職」が表示されます。
5. スパイには場所は表示されず、候補場所リストだけが表示されます。
6. 質問例を参考にしながら話し合い、ホストが投票へ進めます。
7. スパイ人数と同じ人数まで、スパイだと思う人に投票できます。
8. 投票後、スパイだけが候補場所から正解だと思う場所を選びます。
9. ホストが結果発表します。

## スパイ探しの勝利条件

- スパイ側の勝利:
  - 市民側の投票でスパイ全員を見つけられなかった
  - または、スパイの誰かが正解の場所を当てた
- 市民側の勝利:
  - 市民側の投票でスパイ全員を見つけた
  - かつ、スパイが誰も正解の場所を当てられなかった

## ホスト設定

- 話し合い時間: 1分、3分、5分、10分
- 場所ジャンル: 日常、学校、お店、旅行、ファンタジー、すべて
- NGワードジャンル: 日常、学校、食べ物、ゲーム、アニメ・漫画、なんでも
- スパイ人数:
  - 初期値は1人
  - 3〜5人は1人
  - 6〜8人は1人または2人
  - 9人以上は最大3人

## Firebaseの主なデータ構造

```txt
rooms/{roomCode}
  gameType: "wordwolf" | "spyhunt" | "ng-word"
  status: "waiting" | "playing" | "voting" | "spyGuess" | "results"
  hostId
  hostChangedAt
  roundNumber
  settings
    timerSeconds
    topicGenre
    spyCount
  players/{playerId}
    name
    joinedAt
  game
    topicSet
    minorityPlayerId
    ngWords/{playerId}
    eliminated/{playerId}
    genre
    startedAt
    timerSeconds
    roundNumber
  spyHunt
    genre
    location
    locationId
    roles/{playerId}
    spyIds/{playerId}
    candidateLocations
    questionExamples
  votes/{voterPlayerId}
    targetPlayerId
    targetPlayerIds/{targetPlayerId}
    votedAt
  spyGuesses/{spyPlayerId}
    location
    guessedAt
  result
    suspectedPlayerIds
    voteCounts
    allSpiesFoundByVote
    anySpyGuessedLocation
    winner
    revealedAt
```

`game` はワードウルフとNGワードゲーム用、`spyHunt` と `spyGuesses` はスパイ探し用です。既存のワードウルフ部屋に `gameType` が無い場合は、ワードウルフとして扱います。

## 途中退出・ホスト変更

- 「部屋を退出」でプレイヤーは部屋から削除されます。
- ブラウザを閉じた場合も、可能な範囲でFirebaseの `onDisconnect` により退出扱いになります。
- ホストが退出した場合、残っている参加者のうち `joinedAt` が最も古い人が新しいホストになります。
- ゲーム中に退出者が出ても、投票・結果集計で画面が止まらないように、現在残っているプレイヤーを基準に表示します。

## 動作確認手順

1. `index.html` をブラウザで開きます。
2. トップ画面に「ワードウルフ」「スパイ探し」「NGワードゲーム」のカードが表示されることを確認します。
3. ワードウルフを選び、部屋作成、参加、開始、投票、結果発表、次のラウンドが動くことを確認します。
4. スパイ探しを選び、部屋作成、複数プレイヤー参加、ホスト設定、開始ができることを確認します。
5. 一般人には場所と役職、スパイには候補場所リストが表示されることを確認します。
6. 投票、スパイの場所当て、結果発表、次のラウンドが動くことを確認します。
7. NGワードゲームを選び、他人のNGワード表示、自分のNGワード非表示、ホストだけの脱落操作、結果発表、次のラウンドが動くことを確認します。
8. 退出、ホスト退出後のホスト変更、スマホ幅表示を確認します。

## 現時点の制限事項

- Firebase Realtime Databaseへの接続が必要です。
- `onDisconnect` は通信状態やブラウザ環境によって即時反映されない場合があります。
- 結果発表はホストが強制実行できるため、未投票者や未回答スパイがいても進行できます。
- スパイ探しのお題データは `index.html` 内の静的データです。
- NGワードゲームのNGワードデータは `index.html` 内の静的データです。

## 制作

- 制作: LegendManbou
- https://x.com/LegendManbou
