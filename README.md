# IC経由ナビ

福岡都市高速を使わずに一般道でNEXCOのICまで行き、そこから高速に乗るルートを比べて、Googleマップで案内を始めるWebアプリ（PWA）。

## しくみ
1. 出発地から各IC（古賀・若宮・福岡・太宰府・筑紫野・鳥栖・今宿・前原）までを「有料道路なし」で計算
2. 各ICから目的地までを「高速あり」で計算し、合計が速い順に並べる
3. 都市高速も使うおまかせルートと比べて「何分の差か」を表示
4. Googleマップのリンクを作る
   - 共通: ICまでの一般道ルート上の点を経由地（最大2か所）にし、最後にICの入口を経由させる
   - Android: `google.navigation:...&avoid=t` で「有料道路なし」にしてICまで案内（2段階）

| 用途 | 無料モード | Google APIキーあり |
|---|---|---|
| 経路計算 | Valhalla（FOSSGISの公開サーバー、渋滞なし） | Routes API（渋滞反映、料金表示あり） |
| 場所検索 | Nominatim (OpenStreetMap) | Places API (New) Text Search |

ICの選び方は常にValhallaで計算し、APIキーがあるときは上位2つのICだけGoogleで計算し直す（APIの呼び出し回数を抑えるため）。

## スマホで使う（GitHub Pagesで公開）
現在地を使うにはHTTPSが必要なので、GitHub Pagesで公開するのがいちばん簡単です。
1. GitHubで新しいリポジトリ（例: `ic-navi`）を作り、このフォルダの中身をpushする
2. リポジトリの Settings → Pages → Branch を `main` / `/ (root)` にして保存する
3. `https://<ユーザー名>.github.io/ic-navi/` をスマホで開く
4. ホーム画面に追加する（Android: メニュー →「ホーム画面に追加」、iPhone: 共有 →「ホーム画面に追加」）

## Google APIキー（任意）
1. Google Cloudでプロジェクトを作り、課金を有効にする
2. 「Routes API」と「Places API (New)」を有効にする
3. APIキーを作り、次の制限をかける
   - アプリケーションの制限: ウェブサイト → `https://<ユーザー名>.github.io/*`
   - APIの制限: Routes API と Places API (New) のみ
4. アプリの⚙設定にキーを貼る（キーはその端末のlocalStorageにだけ保存される）

## 手元で動かす
```
python -m http.server 8765
```
http://localhost:8765 を開く。

## ICを追加・修正する
`index.html` の `ICS` 配列を編集します。座標は「ICのランプが一般道に接続する地点」にしてください。料金所の座標だとOSM上で有料区間の中と判定され、「有料道路なし」の計算が遠回りになります。

## 既知の制限
- 無料モードの時間は渋滞を考慮しないので、市街地では実際より短めに出る
- Googleマップのリンクでは、経由地ごとに「経由地に到着」と表示される
- iPhoneのGoogleマップはリンクで「有料道路を使わない」を指定できないため、経由地で誘導している。経由地の間で都市高速を使われた場合は、Googleマップのルートオプションで手動で切り替える
- Androidの2段階案内（intentリンク）は実機で未検証
- 公開サーバー（Valhalla・Nominatim）は個人で使う程度を想定している。連打しないこと
