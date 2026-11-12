# タスク指示書：SCR-001 備品検索

> 全て架空の記入例。参照するコードやコミットは実在しない。[全体解析](analysis.md)のJ1・C0を用いる。

## 1. 指示の識別・確定記録

| 項目 | 内容 |
| --- | --- |
| 指示書ID／版／状態 | INS-SCR-001 / v1 / 確定（例） |
| 対象 | SCR-001 備品検索 |
| 作成者／日時 | 作業者A / 2026-11-10 14:00 JST |
| 確定者／日時／委任根拠 | 作業者A / 同日15:00 JST / REQ-DEMO-001による同等移行・View基盤変更の委任 |
| 確定版の保存方法 | demo-docsの固定コミット。識別値は報告書に記載 |
| 共通ルール／全体解析 | demo-docs@4444444444444444444444444444444444444444 の docs/rules.md／examples/analysis.md |
| Java／既存C# | J1／C0（全体解析で定義） |
| 旧指示書・CR | 初版につきなし |
| 作業者／判断者／確認者 | 作業者A／依頼者B／確認者C。全て架空 |

## 2. 範囲と詳細調査

- 目的：OV-SCR-001-001～003に対応し、検索条件、表示、認証後の操作を再現する。
- 変更対象：AssetsController.Index、AssetSearchViewModel、Views/Assets/Index.cshtmlおよび対応テスト。詳細画面と認証基盤の実装は対象外。
- 既存動作：GET /assets?name=検索語。名前の部分一致、大文字小文字を区別、空文字は全件、ID昇順。未指定も全件。0件は「該当する備品はありません」。入力値は保持する。
- 根拠：J1:src/AssetSearchAction.java#execute、src/AssetDao.java#search、web/assets/search.jsp。DBを変更しない。C0:src/Demo/Data/AssetRepository.cs#Searchは同じ検索契約を持つ。
- 既存の空白の扱い：前後空白を除去せず検索する。改善案が出ても本移行でTrimを追加しない。
- 再利用：AssetRepository.Searchを変更せず使用。認証はCOM-001、詳細遷移先はSCR-002に依存。
- 技術的調整：JSPをRazor ViewとViewModelへ移す。HTTP入力はControllerで受ける。業務条件や出力エンコードの意味を変更しない。
- 保留：FND-SCR-001-001の認証接続先、およびSCR-002の完成待ち。TASK-SCR-001-003は両方の提供版が固定されるまで着手しない。

## 3. 詳細タスク一覧

| タスクID | 親概要ID | 区分 | 作業内容・出力先 | 根拠 | 依存・着手条件 | 受入条件・検証 |
| --- | --- | --- | --- | --- | --- | --- |
| TASK-SCR-001-001 | OV-SCR-001-001 | 同等移行 | AssetsController.IndexとAssetSearchViewModel、検索テストを作成 | J1:AssetSearchAction.execute・AssetDao.search、C0:AssetRepository.Search、MAP-001・002 | なし | 部分一致、空文字、大小文字、空白、順序が一致。CHK-SCR-001-001・003 |
| TASK-SCR-001-002 | OV-SCR-001-002 | 技術的調整 | Views/Assets/Index.cshtmlと表示テストを作成 | J1:web/assets/search.jsp、MAP-001 | TASK-SCR-001-001のモデル契約 | 入力保持・0件表示・安全な文字表示・詳細URLが一致。CHK-SCR-001-002・003 |
| TASK-SCR-001-003 | OV-SCR-001-003 | 同等移行 | 認証と詳細画面を接続検証し証跡を保存 | J1:LoginGuard.check・AssetDetailAction.execute、MAP-003・004 | COM-001とSCR-002の提供版。現時点は保留 | 未ログイン時とログイン後の詳細・戻り操作が一致。CHK-SCR-001-004 |

取消済みタスク：なし。

## 4. 検証計画

合成データ：ID 1「Pen」、ID 2「pen」、ID 3「Box」、ID 4「<箱>」。既存Javaの期待値は上記固定ソースの条件から得る。実案件ではJavaでの確認証跡も保存する。

| 検証ID | 対象タスク | 観点 | 操作・入力 | 期待値 | 環境・前提 |
| --- | --- | --- | --- | --- | --- |
| CHK-SCR-001-001 | TASK-SCR-001-001 | 必須・正常／境界 | dotnet test tests/Demo.Tests/Demo.Tests.csproj --filter FullyQualifiedName~SearchTests。未指定、空文字、Pen、pen、半角空白を前後に付けたPen、zzz、DB例外を試す | 未指定・空文字は1,2,3,4、Penは1、penは2、前後空白付きとzzzは0件。DB例外は既存の共通エラー処理へ伝播 | SQLite合成DB、J1:AssetDao.search・AssetSearchAction.execute |
| CHK-SCR-001-002 | TASK-SCR-001-002 | 必須・表示／特殊文字 | dotnet test tests/Demo.Tests/Demo.Tests.csproj --filter FullyQualifiedName~SearchViewTests。検索画面を描画して入力値、0件、ID 4、詳細リンクを確認 | 入力保持、規定の0件文言、<箱>を文字として表示、/assets/4のリンク | ローカル表示テスト。認証は代替し、認証検証としては扱わない。J1:search.jsp |
| CHK-SCR-001-003 | TASK-SCR-001-001・002 | 必須・ビルド／回帰 | dotnet build Demo.sln、dotnet test Demo.sln | ビルドと全テスト成功 | 指定SDK、合成DB |
| CHK-SCR-001-004 | TASK-SCR-001-003 | 必須・権限／遷移／共通影響 | 未ログインで/assetsにアクセス。次に試験用利用者でログイン→Pen検索→ID 1の詳細→検索へ戻る | 未ログインは/loginへ遷移し一覧非表示。ログイン後はID 1の詳細、戻るとPen条件を保持 | COM-001・SCR-002の提供版。J1:LoginGuard.check・AssetDetailAction.execute |

- 入力不正：自由入力のため禁止文字は設けない。特殊文字はCHK-002、空・未指定・0件はCHK-001で確認。
- DB更新・競合：参照専用のため対象外。DB失敗はCHK-001、共通認証はCHK-004で確認。
- 証跡保存先：非公開の `evidence/SCR-001/`（架空）。

## 5. 終了・引き継ぎ条件

- 3タスク・4検証が全て必須。共通ルールに沿って人間が判定する。
- 報告先：[report.md](report.md)。接続検証の保留を残したまま画面完了にしない。
- 不明点は依頼者Bへ。接続検証を待つ間も検索・表示の独立タスクは進める。
- 確定前点検：作業者Aが根拠・範囲・依存・検証の対応を点検済みという設定。
