# ProbPlan

ZENStudy Webアプリケーションコンテスト応募作品

不確実なタスクを **3点見積もり（楽観値・最頻値・悲観値）** で登録し、モンテカルロシミュレーション（三角分布）でプロジェクトの完了日や締切達成確率を予測するタスク管理ツールです。ビルド不要のバニラJS（ES Modules）+ Tailwind CDNのみで動作します。

## 主な機能

- プロジェクト / タスクのCRUD、優先度・締切・完了状態の管理
- タスクの3点見積もりから完了日数の確率分布をシミュレーションし、ヒストグラムで表示
- プロジェクト全体の完了予測（p50/p80/p90）と締切達成確率の算出
- `.pplp`（独自拡張子のJSON）形式でのエクスポート／インポート
- 全プロジェクト横断のタスク検索（⌘K / Ctrl+K）
- CLI画面（`terminal.html`）からプロジェクト・タスクをコマンド操作（日本語/英語出力切替対応）
- ライト / ダークテーマ、初回チュートリアル、`localStorage`破損時の復旧画面

## ディレクトリ構成

```
index.html      ホーム画面（プロジェクト一覧・最近のタスク）
detail.html     プロジェクト詳細画面（タスク一覧・シミュレーション）
terminal.html   CLI風の操作画面
css/style.css   Tailwindユーティリティで表現しきれないカスタムスタイル
image/          ファビコン・OGP画像
js/             アプリ本体（下記参照）
tests/          ブラウザ実行の自作テストスイート
```

### `js/` の役割分担

読みやすさを優先し、「モデル → コントローラー → サービス → シミュレーション → UI」の層でファイルを分けています。

| レイヤー | ファイル | 役割 |
|---|---|---|
| エントリポイント | `js/app.js` | ホーム画面。保存データの読込・各UIの初期化・画面間のイベント配線 |
| | `js/detail.js` | プロジェクト詳細画面。URLの`id`から対象プロジェクトを特定し、タスク操作を接続 |
| | `js/terminal.js` | CLI画面。入力の受付・履歴（↑↓）・コマンド結果の表示 |
| モデル | `js/models/task.js` | タスク本体（3点見積もり・優先度・完了状態） |
| | `js/models/project.js` | プロジェクト本体（タスクの集合、進捗率の計算） |
| | `js/models/projectManager.js` | 複数プロジェクトの横断検索・最近のタスク取得 |
| コントローラー | `js/controllers/projectController.js` | プロジェクトの作成・削除・アーカイブ等、モデル変更と保存を1操作にまとめる入口 |
| | `js/controllers/taskController.js` | タスクの作成・更新・完了切替・削除・並び替えの入口 |
| サービス | `js/services/storageService.js` | `localStorage`への保存・復元（破損データの検知を含む） |
| | `js/services/dataTransferService.js` | `.pplp`のエクスポート／インポートとフォーマット検証 |
| | `js/services/sampleDataService.js` | チュートリアル用サンプルプロジェクトの生成 |
| | `js/services/terminalCommandService.js` | CLIコマンドの構文解析・実行・日英メッセージの切替 |
| シミュレーション | `js/simulation/triangular.js` | 三角分布に従う乱数生成（逆変換法） |
| | `js/simulation/taskSimulation.js` | 1タスク分の完了日数シミュレーション（p50/p80/p90） |
| | `js/simulation/projectSimulation.js` | プロジェクト全体の完了予測・締切達成確率の算出 |
| | `js/simulation/histogram.js` | シミュレーション結果をCanvasへヒストグラム描画 |
| UI（画面部品） | `js/ui/projectView.js` / `projectDetailView.js` | ホーム／詳細画面のプロジェクト表示 |
| | `js/ui/taskView.js` / `recentTaskView.js` | 詳細画面のタスク一覧／ホームの直近タスク表示 |
| | `js/ui/projectModal.js` / `taskModal.js` | プロジェクト・タスクの作成／編集モーダル |
| | `js/ui/simulationModal.js` | シミュレーション結果・ヒストグラムの表示モーダル |
| | `js/ui/projectListModal.js` / `projectContextMenu.js` / `projectDeleteModal.js` | プロジェクト一覧モーダル・右クリックメニュー・削除確認 |
| | `js/ui/importPreviewModal.js` | インポートデータのプレビューと反映確認 |
| | `js/ui/taskSearch.js` | 全プロジェクト横断のタスク検索（⌘K / Ctrl+K） |
| | `js/ui/homePanels.js` | スマホ幅でのホーム画面スワイプ／タブ切替 |
| | `js/ui/tutorialModal.js` | 初回チュートリアル |
| | `js/ui/dataRecoveryView.js` | 保存データ破損時の復旧画面 |
| | `js/ui/undoToast.js` | 削除直後の取り消しトースト |
| | `js/ui/theme.js` | テーマ（ライト／ダーク）の切替・保存 |
| | `js/ui/customControls.js` | OS標準に依存しない共通select／date入力 |
| | `js/ui/modal.js` | 全モーダル共通のオーバーレイ管理 |
| | `js/ui/formValidation.js` / `taskDeadline.js` / `escapeHtml.js` | 入力検証、締切表示の整形、HTMLエスケープの共通ヘルパー |
| 共通検証 | `js/validation.js` | GUI・CLI・インポートが共有する名前／日付の検証ルール |

## 動かし方

ビルドや依存パッケージのインストールは不要です。ES Modules（`type="module"`）を使っているため、`file://`で直接開くのではなく簡易HTTPサーバー経由で開いてください。

```bash
python3 -m http.server 8000
# ブラウザで http://localhost:8000 を開く
```

- ホーム画面: `index.html`
- CLI画面: `terminal.html`（ホーム画面右上のアイコンからも遷移可能）

データはブラウザの`localStorage`に保存されます（キー: `probplan`）。ホーム画面・詳細画面・CLI画面は`storage`イベントで同期します。

## テストの実行

外部ライブラリを使わない自作のテストランナー（`tests/run-tests.js`）を用います。上記のローカルサーバーを起動した状態で、ブラウザの開発者コンソールから次を実行してください。

```js
const { runAllTests } = await import("./tests/run-tests.js");
await runAllTests();
```

結果は `console.table` で一覧表示され、失敗があれば例外が投げられます。対象領域は `tests/*.test.js`（モデル・コントローラー・サービス・シミュレーション・CLI・UI・検証ロジック・チュートリアル）に分かれています。

## 技術スタック

- Vanilla JavaScript（ES Modules、フレームワーク・ビルドツール不使用）
- Tailwind CSS（`@tailwindcss/browser` CDN版）+ `css/style.css`のカスタムスタイル
- `localStorage`によるクライアントサイド永続化
- Canvas APIによるヒストグラム描画

## ライセンス

MIT License（[LICENSE](./LICENSE)を参照）
