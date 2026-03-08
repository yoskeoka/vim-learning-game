# Architectural Decision Records (ADR)

## [2026-03-08] Locale, Language, and Timezone Strategy

### Context
Vim Learning Gameはブラウザベースの教育ゲームとして、グローバルなVimユーザーを対象とする。UI言語、タイムゾーン、将来の多言語対応方針を決定する必要がある。

### Decision
- **対応言語（初期）**: 英語・日本語
- **デフォルト言語**: 英語（ユーザーのlocaleが未対応の場合のフォールバック）
- **言語選択**: ユーザーのブラウザ/OSロケールから自動検出。設定で手動切り替え可能
- **Timezone**: ユーザーのシステムlocaleに基づいて決定（サーバーサイドではUTC保存、表示時にクライアントサイドで変換）
- **追加言語**: ゲーム完成後に検討。コミュニティ（有志）による翻訳貢献に期待

### Consequences
- i18nの仕組みを初期から設計に組み込む必要がある（ハードコード文字列禁止）
- 2言語対応によりUI文言の作業量が倍になるが、教育コンテンツとしての到達範囲が大幅に広がる
- reversi-adventureと同一方針のため、i18n設計知見を共有できる
- 将来の言語追加は翻訳ファイルの追加のみで対応可能な設計とする

---

## [2026-03-08] Tech Stack Selection: Phaser v4 + TypeScript

### Context
Vim Learning Gameはキーボード操作で進行する2D教育ゲームであり、プラットフォーム・ゲームエンジン・実装言語を選定する必要がある。

ゲームの特性:
- テキスト表示（モノスペースフォント、カーソル点滅、シンタックスハイライト、行番号、ステータスバー）がビジュアルの中核
- キーボード入力が100%のインタラクション。Escape、Ctrl+キー、生文字入力等を確実に捕捉する必要がある
- 2Dのライトウェイトなゲーム。重厚なゲームエンジンは不要
- 複数のゲームスタイルをプロトタイピングしたい（高速な試行錯誤が重要）
- 将来的にNative（デスクトップ）/Steamリリースも視野に入れるが、まずはWebが最優先
- 課金（追加コンテンツ販売等）の導線を確保したい

### Decision Drivers
- テキスト描画品質（Vim画面の忠実な再現）
- キーボード入力の確実な捕捉
- Web対応（ブラウザでプレイ可能）
- Native/Steam対応の拡張性
- 課金統合の容易さ
- LLMコード生成精度（AI-Centered Development）
- プロトタイプの回転速度
- i18n対応のエコシステム

### Considered Options

#### A. JS/TS系 ゲームエンジン・フレームワーク

**Tier 1: メジャー・活発**

| エンジン | バージョン | Stars | npm/週 | 最終コミット | 種別 |
|---|---|---|---|---|---|
| **PixiJS** | v8.16.0 | 46,700 | 480,014 | 2日前 | レンダリングライブラリ（エンジンではない） |
| **Phaser** | v3.90.0 / v4.0.0-rc.6 | 39,100 | 107,409 | v3は10ヶ月前（凍結）、v4 RC6は2025年12月 | フルゲームエンジン |
| **p5.js** | v2.2.2 | 23,500 | 82,617 | 2週間前 | クリエイティブコーディング（LGPL-2.1） |

**Tier 2: 活発だが小規模**

| エンジン | バージョン | Stars | npm/週 | 最終コミット | 種別 |
|---|---|---|---|---|---|
| **KAPLAY** | v3001.0.12 / v4alpha | 1,500 | 11,874 | 昨日 | フルゲームエンジン（Kaboom.js後継） |
| **Excalibur.js** | v0.32.0 | 2,200 | 4,618 | 5日前 | TypeScript-firstゲームエンジン（pre-1.0） |
| **MelonJS** | v17.4.0 | 6,200 | 149 | 3週間前 | フルゲームエンジン（Tiled連携強い） |
| **LittleJS** | v1.18.0 | 4,000 | 362 | 3週間前 | 超軽量エンジン（JS13k向け） |

**Tier 3以下: ニッチ・非推奨**

| エンジン | Stars | 状態 |
|---|---|---|
| **Two.js** | 8,600 | 描画ライブラリ（ゲーム用ではない） |
| **Kontra.js** | 1,000 | JS13kジャム特化 |
| **CreateJS/EaselJS** | 8,200 | 8年以上放置。非推奨 |
| **Impact.js** | 2,100 | 2年放置。非推奨 |

**Vanilla TS（フレームワークなし）:**
- Canvas/DOM直接操作。最大の自由度。バンドルサイズゼロ
- ただしシーン管理・オーディオ・アセット読込等をすべて自前実装

#### B. Rust系

| エンジン | バージョン | Stars | Web対応 | Native |
|---|---|---|---|---|
| **Bevy** | v0.18.1 | 44,900 | ⚠️ WASM 15-30MB。WebGPUブラウザ対応限定 | ✅ ネイティブ |
| **Macroquad** | v0.4.x | 4,300 | ✅ WASM 2-5MB（WebGL1） | ✅ ネイティブ |

#### C. Go系

| エンジン | バージョン | Stars | Web対応 | Native |
|---|---|---|---|---|
| **Ebitengine** | v2.9.9 | 13,000 | ✅ WASM 5-10MB（WebGL2） | ✅ ネイティブ |

### 比較結果

#### Web対応

| エンジン | Web | テクノロジー | バンドルサイズ |
|---|---|---|---|
| **Phaser** | ✅ 最適（ブラウザがプライマリターゲット） | Canvas + WebGL | ~1MB（Compressor使用で~300KB） |
| **PixiJS** | ✅ 最適 | WebGL + WebGPU + Canvas | ~300-500KB（tree-shakeable） |
| **KAPLAY** | ✅ 最適 | WebGL | ~200-300KB |
| **Vanilla TS** | ✅ 最適 | Canvas/DOM | ~0KB（自分のコードのみ） |
| **Bevy** | ⚠️ 実験的 | WASM + WebGPU | 15-30MB（カジュアルWebゲームに重すぎ） |
| **Macroquad** | ✅ 良好 | WASM + WebGL1 | 2-5MB |
| **Ebitengine** | ✅ 良好 | WASM + WebGL2 | 5-10MB |

#### Native / Steam対応

| エンジン | Native | Steam実績 |
|---|---|---|
| **Phaser** | Tauri/Electron経由 | Vampire Survivors（元Phaser製）。Tauri経由で小バイナリ（~5MB） |
| **PixiJS** | Tauri/Electron経由 | Webコンテンツ中心。Steam実績少ない |
| **Bevy** | ✅ ネイティブバイナリ | `bevy_steamworks`あり。Tiny Glade等 |
| **Macroquad** | ✅ ネイティブバイナリ | `steamworks-rs`。商用タイトル少ない |
| **Ebitengine** | ✅ ネイティブバイナリ | `go-steamworks`。Bear's Restaurant実績 |

#### 課金対応

| エンジン | Web課金 | Steam課金 |
|---|---|---|
| **JS/TS系全般** | ✅ JS生態系（Stripe等）直結。最も簡単 | Steamworksノードバインディング |
| **Rust/Go系** | ⚠️ WASM境界で困難。別途Webフロントエンド要 | steamworks-rs / go-steamworks |

#### テキスト描画（Vimゲームで最重要）

| エンジン | テキスト描画 | モノスペース / カーソル | リッチテキスト |
|---|---|---|---|
| **Phaser** | ✅ `Text`, `BitmapText`, `DOMElement`（HTMLオーバーレイ） | 良好。Webフォント使用可。カーソルはスプライトまたはテキスト操作 | `DOMElement`でHTML/CSSオーバーレイ使用可 |
| **PixiJS** | ✅ `Text`, `BitmapText`, `HTMLText`（DOM経由） | 優秀。`HTMLText`でCSS付きモノスペース。ピクセルパーフェクト | `HTMLText`でHTML/CSSサポート |
| **Vanilla TS** | ✅ Canvas `fillText()` または DOM要素 | **最良** — DOM `<pre>`, `<span>`, CSSモノスペース、CSS点滅カーソル | **最良** — フルHTML/CSS |
| **Bevy** | ⚠️ `Text2d`コンポーネント。基本的 | 中程度。レイアウト制御限定的 | 限定的 |
| **Macroquad** | ⚠️ `draw_text()`。基本的 | 基本的。レイアウトエンジンなし | 非常に限定的 |
| **Ebitengine** | ✅ `text/v2`パッケージ | 良好。TTF/OTFモノスペース対応 | 中程度。手動スタイリング |

#### キーボード入力

| エンジン | キーボード処理 | Vimゲーム適性 |
|---|---|---|
| **Phaser** | ✅ `KeyboardManager`、キーキャプチャ（ブラウザデフォルト抑止） | 良好。Escape、修飾キー等をキャプチャ可能 |
| **Vanilla TS** | ✅ DOM `KeyboardEvent`直接。完全制御 | 最良。`event.key`, `event.code`, 修飾キー, `preventDefault`のフルコントロール |
| **Bevy** | ✅ `ButtonInput<KeyCode>` | ネイティブ:優秀。WASM:一部ブラウザキーが横取りされる |
| **Ebitengine** | ✅ `inpututil.IsKeyJustPressed()` | 良好。WASM対応テスト済み |

**補足: 全画面問題**: ブラウザでは`Ctrl+W`（タブ閉じ）等の一部OSショートカットが横取りされる。対策として:
- `preventDefault()`でほとんどのキーは非全画面でも捕捉可能
- 非全画面時はフッターに警告バー表示（赤帯「Full screen recommended」）
- ブラウザが横取りするキー（Ctrl+W, Ctrl+N等）はVimカリキュラムでは非優先のため実用上問題なし
- 全画面推奨だが必須ではない設計にする

#### LLMコード生成精度

| エンジン | LLM精度 | 理由 |
|---|---|---|
| **Phaser** | ⭐⭐⭐⭐⭐ | 10年以上のチュートリアル・StackOverflow蓄積。安定v3 API。2000+サンプル |
| **PixiJS** | ⭐⭐⭐⭐ | 大きなコーパス。ただしv8でAPI変更あり、LLMがv5/v6/v7コードを生成することも |
| **Vanilla TS** | ⭐⭐⭐⭐⭐ | Canvas/DOM APIは最も豊富な訓練データ |
| **KAPLAY** | ⭐⭐ | Kaboom.jsからのリブランド + API乖離。v4 alpha。LLMが旧API生成 |
| **Excalibur.js** | ⭐⭐ | 小規模コミュニティ。pre-1.0でAPI変更あり |
| **Bevy** | ⭐⭐⭐ | コーパス成長中だが3ヶ月ごとのAPI破壊。LLMが旧バージョンコード生成 |
| **Ebitengine** | ⭐⭐⭐ | 中規模コーパス。API比較的安定。Go生成は一般に良好 |

#### 開発速度

| エンジン | ホットリロード | ビルド時間 |
|---|---|---|
| **JS/TS系全般** | ✅ Vite HMR（即座に反映） | 即時 |
| **Bevy** | ❌ HMRなし | 30秒以上（インクリメンタル） |
| **Macroquad** | ❌ HMRなし | ~16秒（クリーン） |
| **Ebitengine** | ❌ HMRなし | 高速（Go特性） |

### Decision Outcome

**Phaser v4 (RC) + TypeScript を採用。**

Vim画面部分は Phaser の `DOMElement` を活用し、DOM/CSSでモノスペースフォント・カーソル・ハイライトを描画する。ゲーム演出部分（シーン管理、アニメーション、音声、トゥイーン等）は Phaser のビルトイン機能を使用する。

#### Phaser v4を選ぶ理由（v3ではなく）
- 本プロジェクトは積極的に完成まで推し進めるフェーズではなく、本格開発時にはv4安定版が出ている想定
- v3→v4マイグレーションの手間を最初から回避
- 2Dライトウェイトな使い方のため、RC段階のエッジケースバグを踏むリスクが低い
- Phaser v4はWebGLレンダラー全面刷新・パフォーマンス向上等、v3の上位互換

#### Phaser v4の現状（2026年3月時点）
- v4.0.0-rc.6（2025年12月24日リリース）。安定版リリース日は未発表
- Phaser v3（v3.90.0）は最終リリースとしてmasterブランチ凍結済み
- npm `@latest` はv3。v4は `npm install phaser@beta` で取得
- Phaser Studio Inc.（Richard Davey運営）は商業的に活発: Editor v5ベータ、MCP Server、週複数回ブログ更新
- OpenAI、Claude Codeとの連携事例あり。AI-Centered Developmentとの相性良好

### Pros and Cons of Top Options

#### Phaser v4 + TypeScript（採用）
- Good, because フル機能ゲームエンジン（シーン管理、入力、音声、トゥイーン、物理）がバッテリー込み
- Good, because `DOMElement`でHTMLオーバーレイ可能 — Vim画面をDOM/CSSで忠実に描画
- Good, because `KeyboardManager`にキーキャプチャ機能あり（ブラウザデフォルト抑止）
- Good, because 39k Stars、12年の歴史、2000+サンプル、チュートリアル・書籍豊富
- Good, because LLMコード生成精度が最高クラス（AI-Centered Development最適）
- Good, because Vite HMRで高速プロトタイピング
- Good, because Webが第一級。Native/Steamは Tauri 経由で可能（Vampire Survivors実績）
- Good, because JS生態系でStripe等の課金統合が容易
- Good, because MIT License。Phaser Editorは有料だが、フレームワーク自体は完全無料
- Bad, because v4はまだRC段階。一部APIが安定版までに変わる可能性
- Bad, because NativeはTauri/Electron経由が必要（真のネイティブバイナリではない）
- Bad, because v4のLLM訓練データはv3に比べてまだ少ない

#### PixiJS + TypeScript（次善）
- Good, because 最高性能の2Dレンダラー。WebGPU対応。npm週48万DL
- Good, because `HTMLText`でCSS付きリッチテキスト。テキスト描画はPixiJSが最良
- Good, because tree-shakeableで軽量（300-500KB）
- Bad, because ゲームエンジンではなくレンダリングライブラリ。シーン管理・入力・音声は別途必要
- Bad, because キーボード処理はDOM直接（PixiJS自体に入力管理なし）
- Rejected because ゲームエンジン機能を自前構築するオーバーヘッドがPhaserのバッテリー込みに対して不利

#### Vanilla TS + DOM/Canvas（代替）
- Good, because テキスト描画・キーボード入力で最大の自由度。フレームワーク依存ゼロ
- Good, because バンドルサイズゼロ。LLM生成精度最高
- Bad, because シーン管理、オーディオ、アセット読込、アニメーションをすべて自前実装
- Rejected because ゲーム演出を入れる段階で結局フレームワークが必要になり、二度手間

#### Ebitengine + Go（次善のNative候補）
- Good, because Web + Native両対応。GoクロスコンパイルでSteamネイティブバイナリ
- Good, because API安定（v2、8年の歴史）。Bear's Restaurant等の実績
- Good, because `go-steamworks`（作者自身がメンテ）
- Bad, because テキスト描画がDOM/CSSベースに劣る（TTFフォント描画は可能だがリッチテキスト限定的）
- Bad, because WASM 5-10MBのバンドルサイズ
- Bad, because HMRなし。プロトタイプ速度がJS/TS系に劣る
- Rejected because Vim画面の忠実な再現にはDOM/CSSが圧倒的に有利

#### Bevy + Rust
- Good, because 最高パフォーマンス。44.9k Stars。ネイティブバイナリ
- Bad, because WASM 15-30MBはカジュアルWebゲームに重すぎ
- Bad, because テキスト描画が基本的。エディタUIの再現に不向き
- Bad, because ECSパラダイムは2Dライトウェイトゲームにオーバースペック
- Bad, because 3-5ヶ月ごとのAPI破壊。LLMが旧バージョンコード生成
- Rejected because Web体験・テキスト描画・API安定性のすべてで劣る

#### Macroquad + Rust
- Good, because Raylib風の簡潔API。WASM 2-5MBで軽い
- Bad, because テキスト描画が基本的（`draw_text()`のみ、レイアウトエンジンなし）
- Bad, because 小規模コミュニティ（4.3k Stars）。LLM訓練データ薄い
- Rejected because テキスト描画とLLM精度で劣る

### Consequences

**ポジティブ:**
- Phaser v4のバッテリー込み機能で、ゲーム演出の実装コストが低い
- DOMElementでVim画面をHTML/CSSで描画でき、テキスト表現の自由度が最大
- Webファースト配信で敷居の低いプレイ体験。URLだけでプレイ可能
- JS生態系のi18n（i18next等）、決済（Stripe等）をスムーズに統合可能
- LLM精度の高さがAI-Centered Developmentの速度に直結

**ネガティブ:**
- v4 RC段階のため、安定版リリースまでの間にAPIが一部変わる可能性がある
- Tauri経由のNativeビルドは真のネイティブに比べてバンドルサイズが大きい（ただしElectronの~150MBに比べTauriは~5MBで実用的）

**リスク緩和策:**
- Phaser v4のバージョンをpinし、安定版リリース時に明示的にアップグレードする
- ゲームの核心（Vimエンジン）はPhaser非依存のTypeScriptモジュールとして実装し、エンジン差し替え時のリスクを最小化
- テキスト描画をDOMElementに集約し、Phaser側のテキスト描画APIへの依存を最小化

---

## [2026-03-08] Vim Engine: Custom Build

### Context
ゲームの核となるVim操作エミュレーション部分の実装方針を決定する必要がある。既存のVimエミュレーションライブラリを採用するか、自作するかの判断。

ゲームとしての要件:
- Vim機能の段階的アンロック（ステージ進行に応じて使えるコマンドが増える）
- 入力途中のコマンド解析の可視化（今何を入力しようとしているか）
- チャレンジ検証（「指定された操作だけでゴールに到達できたか？」）
- ゲームの学習カリキュラムに合わせたVim機能のサブセットコントロール

### 既存ライブラリ調査

#### JavaScript/TypeScript

| ライブラリ | Stars | npm/週 | 単独利用可能？ | 段階的アンロック |
|---|---|---|---|---|
| **@replit/codemirror-vim** | ~431 | 212,000 | ❌ CodeMirror 6に密結合 | ❌ `Vim.map()`/`unmap()`は使えるが機能ゲートAPIなし |
| **monaco-vim** | ~400 | 112,000 | ❌ Monaco Editorに依存 | ❌ CM5 vim.jsラッパー。同上 |
| **VSCodeVim/Vim** | 15,100 | - | ❌ VS Code拡張APIに密結合 | ❌ `vim.handleKeys`で一部キー委譲可能だが設計外 |
| **vim.wasm** | 5,600 | - | △ 自己完結だがフルVimバイナリ | ❌ 実際のVim。vimrcで制御は脆弱 |
| **Ace Editor vim mode** | 27,000+ | - | ❌ Ace Editorに統合 | ❌ CM5 vim.jsベース |

**共通の祖先: CodeMirror 5 `vim.js`**（~6000行）
- @replit/codemirror-vim、monaco-vim、Ace vim modeの3つすべてがこのファイルに由来
- 準独立のVimコマンドパーサー・エグゼキューターを含むが、実際にはCodeMirrorのカーソル/選択/編集APIに密結合
- 概念的には分離可能だが、実用的には書き直しと同等の工数

#### Rust

| ライブラリ | Stars | 単独利用？ | 備考 |
|---|---|---|---|
| **Zed Editor `vim` crate** | 55,000+ (本体) | ❌ ZedのGPUIフレームワークに密結合 | 構造は優秀だがZedのEditor, Workspace, Buffer型を参照 |
| **Helix Editor `helix-core`** | 35,000+ | △ コア編集ロジックは分離crate | Kakoune系でありVimエミュレーションではない |
| **xi-editor** | 20,000+ | △ xi-ropeが再利用可能 | **アーカイブ済み**。Vimエミュレーションなし |

#### Go
- **スタンドアロンのVimエンジンライブラリは存在しない**
- Neovim RPCクライアント（go-client）はあるが、Neovimプロセス起動が必要でヘビーウェイト

### Decision
**自作Vimエンジンを採用。既存ライブラリは不採用。**

### 理由

1. **段階的アンロックAPIが皆無**: 既存実装はすべてVimをall-or-nothingシステムとして扱う。「利用可能なコマンドセットを動的に制限する」概念がない
2. **UI密結合**: すべてのライブラリ（Neovim RPC以外）が特定エディタのAPIに深く統合されている。Vimコマンドパーサー＋テキスト操作ロジックだけを抽出するには実質書き直しが必要
3. **ゲーム固有の要件**: 入力途中コマンドの可視化、チャレンジ検証、学習進捗トラッキングは既存ライブラリの設計範囲外
4. **過剰か不足**: vim.wasmはフルVim（過剰、制御不能）、TUIライブラリは最低限の入力のみ（不足）

### 設計方針

TypeScriptでPhaser非依存のモジュールとして実装。以下のレイヤー構成:

1. **キーパーサー**: キーシーケンス → 構造化コマンド `{count, operator, motion, register, textObject}` に変換。参考: `@replit/codemirror-vim`のパーサー構造（~500行相当）
2. **コマンドレジストリ**: 各コマンドにenable/disableフラグ。ゲーム進行によるアンロック管理
3. **テキストバッファ**: カーソル・選択範囲付きのバッファモデル。undo/redo対応
4. **モーションエグゼキューター**: 純粋関数 `(buffer, cursor, motion) → newCursor`
5. **オペレーターエグゼキューター**: 純粋関数 `(buffer, cursor, range) → newBuffer`
6. **モード状態機械**: Normal → Insert → Visual → Command-line の遷移管理

初期サポート範囲（推定2000-3000行）:
- モーション: `h j k l w b e W B E 0 $ ^ f F t T ; , / ? n N gg G`
- オペレーター: `d c y p`
- テキストオブジェクト: `iw aw i" a" i( a( ib ab`
- カリキュラム進行に応じて段階的に追加

### Consequences
- Vimの全機能再現ではなく、学習カリキュラムに必要なサブセットを段階的に実装
- Vimマニュアルが実装の正式リファレンス。@replit/codemirror-vimのパーサー構造は設計参考
- Phaser非依存モジュールのため、将来のエンジン変更時もVimエンジンは再利用可能
- テストが書きやすい（純粋関数ベースの設計）
- 実装工数は既存ライブラリ採用に比べて大きいが、ゲーム要件を満たすための必須投資

---

## [2026-03-09] セーブデータ・課金・アンロック方針

### Context
Web/Nativeで動作するゲームのセーブデータ管理、将来の課金導入、コンテンツアンロックの設計方針を決定する必要がある。

### Decision

#### セーブデータ
- **初期**: LocalStorage（Web）/ ファイル（Native）にJSON形式で保存
- **Export/Importボタン**をバックアップ手段として常に提供
- **将来（課金導入後）**: 課金ユーザー向けにCloud Save（クロスデバイス同期）を追加
- **保存アダプターを抽象化**し、LocalStorage / Cloud Save / Steam Cloud / Tauriファイルを差し替え可能にする
- **ローカルセーブの暗号化・署名は不要**。クライアント側データはDevTools等で改ざん可能であり、保護コストに見合わない。改ざんされて困るデータ（購入状態、ランキング）はサーバー側で管理する

#### 課金モデル
- **Freemium + コンテンツ課金**を採用。基本操作は無料、中級～上級パックを有料化
- **課金の導入タイミング**: プロトタイプ完成後、本番プロダクトのコンテンツ・ステージ構成を練る段階で設計する。初期は全コンテンツ無料
- Web: Stripe Checkout → Webhook → 自前API → エンタイトルメントDB
- Steam: Steam DLC API で所有確認
- **信頼の境界はサーバー側**。アンロック判定はサーバーで行い、クライアントは参照のみ

#### アンロックシステム
- EntitlementProvider（権限プロバイダー）を抽象化し、Web/Steam/ゲスト等のプラットフォーム差を吸収
- 各ステージ/パックに必要な権限を宣言的に定義
- 無料→有料の境界はデータ変更のみで調整可能な設計

### Consequences
- 初期はLocalStorage + Export/Importのみで、サーバーサイド不要。即プレイ可能
- 課金導入時にサーバーサイド（認証、エンタイトルメント管理、Stripe Webhook、Cloud Save API）が必要になる
- アダプター抽象化により、課金導入時の既存コード変更を最小化できる

---
