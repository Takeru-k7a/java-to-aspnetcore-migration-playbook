# 全体解析結果：架空の備品台帳

> 全て架空の記入例。ソース・人物・コミット・実行結果は実在しない。実際に解析やテストを実行した記録ではない。

## 1. 基本情報・対象

| 項目 | 内容 |
| --- | --- |
| 文書ID／版／状態 | ANALYSIS-DEMO / v1 / 確定（例） |
| 作成者／日時 | 作業者A / 2026-11-10 10:00 JST（架空） |
| 共通ルールの固定版 | demo-docs@4444444444444444444444444444444444444444:docs/rules.md |
| Javaの基準版 | J1 = demo-java@1111111111111111111111111111111111111111 |
| 既存C#の基準版 | C0 = demo-csharp@2222222222222222222222222222222222222222 |
| 対象／除外 | 備品検索・詳細の2画面とログイン状態の参照。サインイン基盤そのものは基盤担当が別途提供 |
| 環境 | 架空のJava MVC、JDK 17、移行先.NET 10／C# 14、SQLiteの架空データ。ビルドはdotnet build Demo.sln |
| 委任範囲・判断者・確認者 | 架空依頼REQ-DEMO-001：同等移行とView基盤変更は作業者が確定可能。仕様判断は依頼者B、確認は確認者C |
| 関連資料 | 本例の基準版・根拠は説明用。業務資料や証跡ファイルは添付していない |

以下のソースパスはJ1またはC0を接頭辞として固定参照する。略称は文書間で同じ意味とする。

## 2. 解析台帳・全体カバレッジ

| 系統／パス・モジュール | 対象数・一覧 | 役割 | 確認方法・根拠 | 状態 | 未確認／除外理由と影響 |
| --- | --- | --- | --- | --- | --- |
| Java設定 | 2：pom.xml、config/routes.xml | 依存とルート | 依存宣言・全ルートを照合 | 解析済 | なし |
| Java画面 | 4：src/AssetSearchAction.java、src/AssetDetailAction.java、web/assets/search.jsp、web/assets/detail.jsp | 2画面 | 全入口とテンプレートの参照を追跡 | 解析済 | なし |
| Java業務／共通 | 3：src/AssetDao.java、src/LoginGuard.java、db/schema.sql | 検索・認証参照・DB | 呼び出し、SQL、テーブル定義を照合 | 解析済 | なし |
| 既存C# | 7：Demo.sln、src/Demo/Demo.csproj、src/Demo/Program.cs、src/Demo/Data/AssetRepository.cs、samples/Search/SampleSearchController.cs、samples/Search/SampleSearchViewModel.cs、samples/Search/Index.cshtml | MVC登録・DB検索基盤・画面sample | 起動設定、公開メソッド、sample一式と共通部品参照を調査 | 解析済 | 認証接続はまだ存在しない |
| Java生成物 | 1：target/demo.war | ビルド生成物 | 元ソースの確認で代替 | 除外 | ソース対応の対象外 |

棚卸し17ファイル、解析済16、除外1、一部0、未確認0。架空の全ファイル一覧は上表に列挙。画面ごとの詳細な条件分岐と実行結果はPhase 2で確認する。

## 3. システム構成

- Java：routes.xml → Action → AssetDao → SQLite。JSPで表示。LoginGuardがログイン状態を確認。
- 既存C#：MVC登録、AssetRepositoryと画面sampleのみ。業務画面Controller、View、認証の接続は未実装。
- 画面sample：C0:samples/Search/のController・ViewModel・Razor Viewの3ファイル。プロジェクト外のひな形であり、実行画面には未接続。SCR-001では一式を新規配置して先にコミットし、次に画面固有の変更を行う。JS・CSS・共通部品の追加コピーは不要という設定。SCR-002での適用可否は詳細調査で決める。
- 移行先：Controller → 既存AssetRepository → 同じ論理スキーマ。ViewModelとRazor Viewを追加する。
- バッチ・外部API：本架空システムにはなし。DB更新機能もない。

## 4. 概略ソース対応表

| 対応ID | Java根拠（J1） | C#根拠／予定（C0起点） | 関係 | 確度 | 関連ID・注意点 |
| --- | --- | --- | --- | --- | --- |
| MAP-001 | src/AssetSearchAction.java#execute、web/assets/search.jsp | src/Demo/Controllers/AssetsController.cs#Index、src/Demo/Models/AssetSearchViewModel.cs、src/Demo/Views/Assets/Index.cshtml | 分割・新規 | 予定 | SCR-001 |
| MAP-002 | src/AssetDao.java#search | src/Demo/Data/AssetRepository.cs#Search | 再利用 | 確認済 | SCR-001。検索条件と順序の詳細比較はPhase 2 |
| MAP-003 | src/AssetDetailAction.java#execute、web/assets/detail.jsp | src/Demo/Controllers/AssetsController.cs#Details、src/Demo/Views/Assets/Details.cshtml | 新規 | 予定 | SCR-002 |
| MAP-004 | src/LoginGuard.java#check | 認証接続先は未定 | 基盤への対応付け | 未確認 | COM-001。認証必須の接続試験を阻害 |

## 5. 画面一覧

| 画面ID | 名称・機能 | 入口／遷移元・先 | Java根拠／対応ID | 既存C# | 共通依存 | 順序 |
| --- | --- | --- | --- | --- | --- | --- |
| SCR-001 | 備品検索 | GET /assets、一覧→詳細 | AssetSearchAction、MAP-001・002 | DB検索のみ | COM-001 | 1：詳細画面の入口 |
| SCR-002 | 備品詳細 | GET /assets/{id}、検索へ戻る | AssetDetailAction、MAP-003 | 未実装 | COM-001 | 2：検索画面から遷移 |

## 6. 画面別概要タスク

| 概要タスクID | 画面 | 作業概要 | 根拠 | 依存・先行条件 | Phase 2の調査点 |
| --- | --- | --- | --- | --- | --- |
| OV-SCR-001-004 | SCR-001 | sample一式を配置して先にコミット | C0:samples/Search/、MAP-001 | sampleの固定版 | 一式のコピー対応、内容一致、実装とのコミット分離 |
| OV-SCR-001-001 | SCR-001 | 検索処理を移す | MAP-001・002 | 既存Repository | 空文字、大小文字、並び順 |
| OV-SCR-001-002 | SCR-001 | 検索画面をRazor Viewで再現 | MAP-001 | 検索結果の契約 | 0件表示、入力保持、詳細リンク |
| OV-SCR-001-003 | SCR-001 | 認証と画面遷移を接続検証 | MAP-003・004 | COM-001、SCR-002 | 未ログイン時・戻る操作 |
| OV-SCR-002-001 | SCR-002 | 詳細表示と戻る操作を移す | MAP-003 | COM-001 | 不明ID、権限、戻り先 |

## 7. 共通処理・画面間依存

| 共通ID | 処理・根拠 | 利用画面 | 既存C# | 担当・指示書 | 方針・影響・確認 |
| --- | --- | --- | --- | --- | --- |
| COM-001 | ログイン状態参照、J1:src/LoginGuard.java#check、MAP-004 | SCR-001・002 | 未実装 | 基盤担当D。指示書未作成 | 担当Dが共通指示書を作成し提供。両画面からの接続検証が必要。画面担当が認証を独自実装しない |

## 8. 不明点・発見事項・判定

| 管理ID | 内容・根拠 | 影響 | 阻害 | 判断先・状態 |
| --- | --- | --- | --- | --- |
| FND-SCR-001-001 | C0に認証接続がなく接続先未定 | SCR-001・002、COM-001 | 個別の検索実装は可能。接続検証と画面完了を阻害 | 基盤担当D・未解決 |

- Phase 1判定：完了（記入例）。作業者A、2026-11-10。全対象の棚卸しと概略解析、対応、画面、概要タスク、依存を記録。
- 詳細調査：両画面とも着手可能。認証の未確定をもって移行可能と断定しない。
- 保存版：demo-docs@4444444444444444444444444444444444444444:examples/analysis.md（架空）。
- 版履歴：v1初版。
