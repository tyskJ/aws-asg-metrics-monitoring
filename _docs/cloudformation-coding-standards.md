# AWS CloudFormation（YAML）コーディング規約

> 適用対象：AWS CloudFormationのYAMLテンプレート、関連するパラメータファイル・テストコード  
> 位置づけ：人間と生成AIが共通して参照するコーディング基準  
> 前提：AWS CloudFormationのYAMLテンプレートを対象とし、標準Transformおよびマクロの利用に関する共通ルールを含む。AWS SAM、AWS CDK、独自マクロの実装詳細はプロジェクト固有規約で補足する。  
> 文書バージョン：1.0

## 改訂履歴

| 版数 | 改訂日     | 改訂内容 |
| ---- | ---------- | -------- |
| 1.0  | 2026-11-11 | 初版作成 |

## 目次

- [AWS CloudFormation（YAML）コーディング規約](#aws-cloudformationyamlコーディング規約)
  - [改訂履歴](#改訂履歴)
  - [目次](#目次)
  - [1. 目的と適用方針](#1-目的と適用方針)
  - [2. スタックとテンプレートの構成](#2-スタックとテンプレートの構成)
    - [2.1 用語](#21-用語)
    - [2.2 許容する配置例](#22-許容する配置例)
    - [2.3 構成ルール](#23-構成ルール)
  - [3. テンプレートファイルの命名と配置](#3-テンプレートファイルの命名と配置)
  - [4. 識別子・命名規則](#4-識別子命名規則)
  - [5. テンプレートのセクション](#5-テンプレートのセクション)
  - [6. 入力パラメータと環境別設定](#6-入力パラメータと環境別設定)
  - [7. 値の参照・条件・出力](#7-値の参照条件出力)
    - [7.1 Outputsの記述ルール](#71-outputsの記述ルール)
  - [8. リソースの定義とプロパティの網羅性](#8-リソースの定義とプロパティの網羅性)
    - [8.1 定義方法](#81-定義方法)
    - [8.2 プロパティの網羅性](#82-プロパティの網羅性)
  - [9. ネストしたスタック・再利用](#9-ネストしたスタック再利用)
  - [10. Transform・条件分岐・繰り返し](#10-transform条件分岐繰り返し)
    - [10.1 基本ルール](#101-基本ルール)
    - [10.2 `Fn::ForEach`の記述例](#102-fnforeachの記述例)
    - [10.3 独自マクロ・複数Transform](#103-独自マクロ複数transform)
  - [11. 実行環境・権限・リージョン](#11-実行環境権限リージョン)
  - [12. 変更セットとスタックのライフサイクル](#12-変更セットとスタックのライフサイクル)
  - [13. YAMLの記述スタイルとコメント](#13-yamlの記述スタイルとコメント)
    - [13.1 基本ルール](#131-基本ルール)
    - [13.2 記述例](#132-記述例)
  - [14. 削除・置換保護と変更管理](#14-削除置換保護と変更管理)
  - [15. セキュリティと機密情報](#15-セキュリティと機密情報)
  - [16. Git管理と品質チェック](#16-git管理と品質チェック)
    - [16.1 Git管理](#161-git管理)
    - [16.2 検証](#162-検証)
      - [16.2.1 Formatter・Linterの利用方針](#1621-formatterlinterの利用方針)
      - [16.2.2 規約と検証手段の対応](#1622-規約と検証手段の対応)
    - [16.3 実行例](#163-実行例)
      - [16.3.1 Transform・マクロの展開後検証](#1631-transformマクロの展開後検証)
    - [16.4 テストコード](#164-テストコード)
      - [16.4.1 記述例](#1641-記述例)
  - [17. プロジェクト固有規約との分担](#17-プロジェクト固有規約との分担)
  - [18. 参考資料](#18-参考資料)

## 1. 目的と適用方針

本規約は、CloudFormationによるInfrastructure as Code（IaC）の**可読性・保守性・再利用性・安全性・再現性**を確保することを目的とする。

**AWS公式のCloudFormationベストプラクティスおよびテンプレート仕様を基本とし、実務上の保守性・運用性を考慮して策定する。**

- テンプレート構造・論理ID・参照・パラメータ・変更セット・テストを一貫した基準で扱う。
- スタックやテンプレートの分割方法を特定のディレクトリに固定しない。
- 既存プロジェクトは既存の構成・運用との整合を優先し、合理的な理由なく分割・改名しない。
- AWS公式仕様による制約と、本規約による運用上の取り決めを区別する。

| 区分 | 意味                                                                     |
| ---- | ------------------------------------------------------------------------ |
| 必須 | CloudFormationの技術的制約、セキュリティ上の要請、または本規約の遵守事項 |
| 原則 | 標準的な方法。合理的な理由がある場合は例外を認める                       |
| 推奨 | プロジェクトの規模や運用に応じて適用することが望ましい                   |

## 2. スタックとテンプレートの構成

### 2.1 用語

| 用語                   | 定義                                                                        |
| ---------------------- | --------------------------------------------------------------------------- |
| テンプレート           | スタックで管理するAWSリソース・入力・条件・出力などを記述したYAMLまたはJSON |
| スタック               | CloudFormationが一体として作成・更新・削除するリソース群の管理単位          |
| ネストしたスタック     | 親テンプレートから`AWS::CloudFormation::Stack`で作成する子スタック          |
| StackSets              | 複数のAWSアカウント・リージョンへスタックを展開する仕組み                   |
| CloudFormation Modules | リソース構成を再利用可能にするCloudFormationの拡張単位                      |

**複数の`.yaml`ファイルを同一ディレクトリに置いても、CloudFormationはそれらを自動的に1つのテンプレートとして結合しない。** 機能別に分ける場合はスタック分割、ネストしたスタック、CloudFormation Modules等の採用を判断する。

### 2.2 許容する配置例

単一テンプレート：

```text
project/
├── template.yaml
├── parameters/
│   ├── prd.json
│   └── stg.json
├── rules/
│   └── network.guard
└── README.md
```

複数スタック：

```text
project/
├── templates/
│   ├── network.yaml
│   ├── compute.yaml
│   └── security.yaml
├── parameters/
│   ├── prd/
│   └── stg/
├── rules/
└── README.md
```

※ `templates/`、`parameters/` などは整理用の任意の名前であり、CloudFormationの必須ディレクトリではない。複数テンプレートの実行順序・参照関係は別途定義する。

### 2.3 構成ルール

| 区分 | 項目         | ルール                                                                                   |
| ---- | ------------ | ---------------------------------------------------------------------------------------- |
| 必須 | 管理単位     | テンプレートと対応するスタック、およびデプロイ先のアカウント・リージョンを明確にする     |
| 原則 | スタック分割 | ライフサイクル・所有者・変更頻度・独立性を基準とし、機械的に細分化しない                 |
| 原則 | 共通化       | 必要に応じてネストしたスタックやCloudFormation Modulesを使用する                         |
| 原則 | 複数環境     | 同一テンプレートと環境別パラメータによる再利用を優先し、差異が大きい場合は分割を検討する |
| 必須 | 依存管理     | スタック間の入出力・参照・デプロイ順・削除制約を明示する                                 |
| 原則 | 分割の過剰化 | 大量のExport/ImportValueによる密結合や、不必要に深いネストを避ける                       |

## 3. テンプレートファイルの命名と配置

| ファイル・ディレクトリ            | 主な用途                                     | 扱い         |
| --------------------------------- | -------------------------------------------- | ------------ |
| `template.yaml`                   | 単一スタックの定義                           | 原則         |
| `network.yaml`、`compute.yaml` 等 | 機能別スタックの定義                         | 必要に応じて |
| `parameters/<env>.json`           | 環境別のスタック入力値                       | 必要に応じて |
| `rules/*.guard`                   | CloudFormation Guardによるポリシー検証       | 推奨         |
| `tests/*_tests.yaml`              | Guardルールの単体テスト                      | 推奨         |
| `.cfnlintrc` / `.cfnlintrc.yaml`  | cfn-lintの共通設定                           | 必要に応じて |
| `README.md`                       | 構成、スタック・ファイル対応、検証・実行手順 | 推奨         |

- ファイル名は小文字の`snake_case`または`kebab-case`から、プロジェクト内で1つの表記を採用する。
- テンプレートの相対パスだけで対象スタックが分かるよう整理する。
- 拡張子は`.yaml`を原則とする。既存の`.yml`も許容する。

## 4. 識別子・命名規則

| 区分 | 対象                              | ルール                                                                     |
| ---- | --------------------------------- | -------------------------------------------------------------------------- |
| 必須 | リソース論理ID                    | 英数字のみとし、同一テンプレート内で一意にする                             |
| 原則 | リソース論理ID                    | `PrivateSubnet`、`ApplicationSecurityGroup`等の`PascalCase`で役割を表す    |
| 原則 | Parameters / Conditions / Outputs | 用途が理解できる`PascalCase`を使用する                                     |
| 原則 | 物理名                            | 必要がなければ固定せず、CloudFormationの生成名を活用する                   |
| 原則 | Tags                              | 共通キー・環境名・所有者等をプロジェクト固有規約で統一する                 |
| 必須 | 論理ID変更                        | 既存リソースの変更・置換・削除に影響し得るため、変更セットで影響を確認する |

**CloudFormationのリソース論理IDには英数字のみを使用する。** `PrivateSubnet`は使用できるが`private_subnet`は使用できない。

## 5. テンプレートのセクション

| 区分 | 項目                               | ルール                                                                  |
| ---- | ---------------------------------- | ----------------------------------------------------------------------- |
| 原則 | `AWSTemplateFormatVersion`         | `'2010-09-09'` を明示する                                               |
| 必須 | `Description`                      | テンプレートの目的・管理範囲を説明する。本規約では記載を必須とする      |
| 原則 | `Parameters`                       | スタック作成・更新時に変える必要がある値だけを公開する                  |
| 原則 | `Mappings`                         | 環境やリージョンによる固定対応表に使用し、過度に複雑化しない            |
| 原則 | `Conditions`                       | リソース作成の有無や条件による値の切り替えに使用する                    |
| 必須 | `Resources`                        | 少なくとも1つの管理対象リソースを定義する（通常のスタックテンプレート） |
| 原則 | `Outputs`                          | 呼び出し元・運用・他スタックが必要とする値のみ公開する                  |
| 任意 | `Metadata` / `Rules` / `Transform` | 目的がある場合のみ追加し、使用意図を明示する                            |

AWSのテンプレート仕様に従い、`Description`は`AWSTemplateFormatVersion`の後に配置する。存在しないセクションを空で記載する必要はない。

## 6. 入力パラメータと環境別設定

| 区分 | 項目             | ルール                                                                                                                |
| ---- | ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| 必須 | 型               | 各`Parameter`に`Type`を指定する                                                                                       |
| 必須 | 説明             | 各`Parameter`に`Description`を指定する（本規約）                                                                      |
| 原則 | 制約             | `AllowedValues`、`AllowedPattern`、`MinLength`、`MaxValue`等で有効範囲を制限する                                      |
| 原則 | 制約違反時の説明 | 制約を設定する場合は`ConstraintDescription`を記載し、入力者が許容値・形式・範囲を理解できるメッセージにする（本規約） |
| 推奨 | AWS専用型        | VPC ID等で適切な`AWS::EC2::VPC::Id`などのAWS専用パラメータ型を使用する                                                |
| 原則 | デフォルト       | 無害で汎用的な値がある場合のみ`Default`を設定する                                                                     |
| 原則 | 環境値           | 環境別ファイル・実行基盤の変数管理に集約し、テンプレート内に個別環境の値を大量に埋め込まない                          |
| 必須 | 秘密値           | テンプレート・Git管理ファイルに平文で含めない。必要に応じてSecrets Manager等を参照する                                |
| 原則 | `NoEcho`         | 秘密値をパラメータとして受け取る場合は`NoEcho: true`を使用する。ただし全出力を秘匿するものではない                    |

`Parameters`はテンプレートのトップレベルセクションに記述する。CLIでパラメータファイルを使用する場合は、実行コマンドが要求するJSON等の形式に従う。

`ConstraintDescription`はCloudFormationの仕様上は任意の属性だが、本規約では制約を定義したパラメータでの記載を原則とする。単なる制約式の繰り返しではなく、利用者がどのような値を入力すべきかを示す。

```yaml
Parameters:
  Environment:
    Type: String
    Description: デプロイ先の環境区分
    AllowedValues:
      - dev
      - stg
      - prd
    ConstraintDescription: dev、stg、prdのいずれかを指定する
```

## 7. 値の参照・条件・出力

| 対象                           | 標準的な方法                                                          |
| ------------------------------ | --------------------------------------------------------------------- |
| 同一スタック内のリソースIDなど | `!Ref`、`!GetAtt`                                                     |
| 文字列の組み立て               | `!Sub`（必要に応じて`!Join`）                                         |
| AWSアカウント・リージョン      | `AWS::AccountId`、`AWS::Region`等の疑似パラメータ                     |
| 環境別の固定値表               | `Mappings`と`!FindInMap`                                              |
| 条件付きリソース・値           | `Conditions`、`!If`、`AWS::NoValue`                                   |
| ネストしたスタックの出力       | `!GetAtt ChildStack.Outputs.OutputName`等                             |
| 独立スタック間の公開値         | `Outputs`の`Export`と`!ImportValue`（結合・更新・削除制約を確認する） |

- 同一テンプレート内で作成するリソースは、その論理IDを直接参照する。
- 参照により依存関係が推論される場合、`DependsOn`を機械的に追加しない。
- 参照できない外部の既存リソース識別子は、必要に応じて`Parameters`などで受け取る。
- 秘密値を`Outputs`へ出力しない。

### 7.1 Outputsの記述ルール

| 区分 | 項目          | ルール                                                                 |
| ---- | ------------- | ---------------------------------------------------------------------- |
| 必須 | `Description` | 定義する各Outputに、出力内容と用途が分かる説明を記載する（本規約）     |
| 必須 | `Value`       | 対象の値を適切な参照・式で定義する                                     |
| 原則 | 出力範囲      | 他スタック、運用、確認作業で利用する値に限定し、不要な出力を増やさない |
| 原則 | `Export`      | スタック間共有が必要な場合だけ使用し、命名と参照依存を整理する         |

`Description`はCloudFormationの仕様上は任意だが、本規約では必須とする。`Description`は組み込み関数を使用しないリテラル文字列で記載し、AWS公式仕様の長さ制限を守る。

```yaml
Outputs:
  PrivateSubnetId:
    Description: 内部用途のサブネットIDを運用確認に使用する
    Value:
      Ref: PrivateSubnet
```

## 8. リソースの定義とプロパティの網羅性

### 8.1 定義方法

| 区分 | 項目         | ルール                                                                                     |
| ---- | ------------ | ------------------------------------------------------------------------------------------ |
| 必須 | `Type`       | 正しいAWSリソースタイプを指定する                                                          |
| 必須 | `Properties` | 使用するリソースタイプで必要なプロパティを漏れなく定義する                                 |
| 原則 | 参照         | 同一スタックで作成したリソースに対して固定IDを埋め込まず、`!Ref`等を使用する               |
| 原則 | `DependsOn`  | 暗黙的な参照だけでは表現できない依存関係に限定して使用する                                 |
| 原則 | Tags         | タグに対応するリソースにはプロジェクトのタグ付け方針を適用する                             |
| 原則 | デフォルト   | セキュリティ・可用性・性能・運用に影響する値は省略時動作を確認し、重要な値は明示を検討する |

### 8.2 プロパティの網羅性

各`Resources`の定義にあたっては、**対象リソースタイプのAWS公式リファレンスにある設定可能な`Properties`およびネストしたプロパティを網羅的に把握したうえで、設定要否を判断する。**

| 区分 | 項目                  | ルール                                                                                                                                    |
| ---- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 必須 | 仕様の確認            | 新規作成時と重要な設定変更時に、対象リソースの最新公式仕様を確認する                                                                      |
| 必須 | 必須プロパティ        | 必須項目および条件付き必須項目を漏れなく定義する                                                                                          |
| 原則 | 任意プロパティ        | 設定要否を判断し、必要なものだけを明示する                                                                                                |
| 原則 | 省略時の動作          | デフォルト動作・セキュリティ・コスト・置換影響を確認する                                                                                  |
| 必須 | 更新時の影響          | 公式仕様の`Update requires`（No interruption / Some interruptions / Replacement）を確認する                                               |
| 原則 | 設計記録              | 設定要否の判断結果は必要に応じて設計書・パラメータシートへ記録する                                                                        |
| 必須 | Transform適用後の確認 | `Fn::ForEach`やマクロでリソースを生成・変更する場合、展開後の各リソースについても必須・任意プロパティ、デフォルト動作、更新影響を確認する |

**網羅的に確認することと、すべての任意プロパティをテンプレートに記載することは区別する。**

## 9. ネストしたスタック・再利用

| 区分 | 項目               | ルール                                                                               |
| ---- | ------------------ | ------------------------------------------------------------------------------------ |
| 原則 | ネストしたスタック | 独立した責務・共通化・管理境界がある場合に検討する                                   |
| 原則 | インターフェース   | 子スタックへの入力は`Parameters`、返却値は`Outputs`を中心に設計する                  |
| 必須 | `TemplateURL`      | ネストしたスタックで参照するテンプレートの配置・配布・バージョン管理方法を明確にする |
| 原則 | リソース共有       | Export/ImportValueの依存関係を把握し、削除順序・変更制約を文書化する                 |
| 原則 | モジュール         | CloudFormation Modulesの採用は登録・配布・バージョン管理方法を含めて判断する         |
| 推奨 | StackSets          | マルチアカウント／マルチリージョンで共通構成を展開するときに検討する                 |

## 10. Transform・条件分岐・繰り返し

### 10.1 基本ルール

| 区分 | 項目                  | ルール                                                                                                                                                                   |
| ---- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 原則 | `Conditions`          | 複数環境の構成差分や作成条件を簡潔に表現する。条件名と評価結果の意味を明確にする                                                                                         |
| 原則 | `Fn::If`              | 過度なネストを避け、`AWS::NoValue`を含む条件付きプロパティの有無を確認する                                                                                               |
| 必須 | Transformの宣言       | 使用するTransformをトップレベルの`Transform`に正しく宣言する。複数指定時は処理順序と依存関係を確認する                                                                   |
| 必須 | `Fn::ForEach`の前提   | `Fn::ForEach`を使用する場合は`Transform: AWS::LanguageExtensions`を宣言する                                                                                              |
| 必須 | `Fn::ForEach`の配置   | `Resources`（そのプロパティを含む）、`Conditions`、`Outputs`のサポート対象箇所だけで使用する                                                                             |
| 必須 | `Fn::ForEach`の識別子 | ループ論理名と展開後のキー・リソース論理IDが一意になることを確認する。論理IDはCloudFormationの制約に従う                                                                 |
| 原則 | 繰り返し対象          | 同じ構造の定義を簡潔に表現できる場合に使用し、理解しにくい多重ループや複雑な動的生成を避ける                                                                             |
| 必須 | 展開結果              | 生成リソース数、論理ID、プロパティ、`Ref`・`Fn::GetAtt`等の参照、条件、Outputs、リソース間の依存関係を確認する                                                           |
| 必須 | クォータ              | 展開後のテンプレートサイズ・リソース数などがCloudFormationのクォータ内に収まることを確認する                                                                             |
| 原則 | カスタムマクロ        | AWS提供Transformと独自マクロを区別し、変換の入出力、Lambda実行権限、変更管理、検証方法を明確にする                                                                       |
| 必須 | Transform利用時の更新 | `AWS::LanguageExtensions`を使用するスタックの更新では元のテンプレートを再指定し、`--use-previous-template`等による古い値の再利用を避ける                                 |
| 必須 | 変更確認              | マクロ・Transformで構成を生成・変更する場合、原則として処理済みテンプレートと変更セットを確認する。AWSの制約で変更セットを使用できない場合は、例外手順と事前承認を定める |

`Fn::ForEach`は`AWS::LanguageExtensions`によって提供される組み込み関数であり、`Fn::ForEach::LoopLogicalName`、置換用識別子、反復対象のコレクション、テンプレート断片で定義する。ループ論理名はテンプレート内で一意とし、`Resources`の論理IDと競合させない。反復対象の指定には配列または`CommaDelimitedList`パラメータへの`Ref`など、公式仕様でサポートされた方法を使用する。

`Fn::ForEach`は`AWSTemplateFormatVersion`、`Description`、`Metadata`、`Transform`、`Parameters`、`Mappings`、`Rules`、`Hooks`では使用できない。また、`AWS::LanguageExtensions`固有関数に`!ForEach`や`!Length`等のYAML短縮記法は使用せず、`Fn::ForEach::...`、`Fn::Length`などの正式な記法を使用する。

`Fn::ForEach`における置換対象は通常`${Identifier}`で表す。反復値に英数字以外の文字を含む場合は`&{Identifier}`も使用できるが、**展開後にリソース論理IDとして成立するか**を別途確認する。入力値によって出力キーが衝突しないようにする。

### 10.2 `Fn::ForEach`の記述例

以下は、2つのSQSキューを繰り返し定義し、それぞれのARNをOutputsに公開する**独立したCloudFormationテンプレートの例**。物理名を固定しないため、CloudFormationによる名前生成を利用する。

`examples/foreach.yaml`

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Example of repeated SQS queues using Fn::ForEach.
Transform: AWS::LanguageExtensions

Resources:
  # ============================================================
  # SQS Queue
  # ============================================================

  # 用途ごとにSQSキューを作成する。反復値は論理IDの一部になる。
  "Fn::ForEach::ApplicationQueues":
    - QueueSuffix # 置換用識別子。リソース論理IDに埋め込む
    - [Orders, Billing] # 作成対象。キューは2つ生成される
    - "${QueueSuffix}Queue":
        Type: AWS::SQS::Queue
        Properties:
          VisibilityTimeout: 30 # メッセージをほかのコンシューマーから隠す時間（秒）

Outputs:
  # ============================================================
  # SQS Queue ARN
  # ============================================================

  # 各キューのARNを出力する。参照先は展開後のリソース論理ID。
  "Fn::ForEach::QueueArns":
    - QueueSuffix # 置換用識別子
    - [Orders, Billing] # Resourcesと同じ反復対象を使用する
    - "${QueueSuffix}QueueArn":
        Description: ARN of the generated queue.
        Value:
          Fn::GetAtt:
            - "${QueueSuffix}Queue" # 展開後のキューの論理ID
            - Arn # 出力する属性
```

展開後は`OrdersQueue`、`BillingQueue`の2つのリソースと、`OrdersQueueArn`、`BillingQueueArn`の2つの出力が定義される。テンプレートを繰り返しで短縮しても、**各リソースの設定要否・コメント・設計意図を省略してよいわけではない**。

なお、この例はAWSの公式仕様に基づく記述例であり、実際のCloudFormation変更セットによる展開確認は実施していない。検証は[16.3.1 Transform・マクロの展開後検証](#1631-transformマクロの展開後検証)に従う。

### 10.3 独自マクロ・複数Transform

- **独自マクロの役割**：Lambda関数と`AWS::CloudFormation::Macro`によって実現するテンプレート変換処理。テンプレート全体の`Transform`、または一部の断片を対象とする`Fn::Transform`から呼び出す。
- **処理順序**：複数Transformを指定する場合はリストで宣言し、変換の順序を確認する。独自マクロとAWS提供Transformを併用するときは、公式仕様に従い依存関係に適した順序を設定する。
- **権限・安全性**：マクロ実行用Lambdaのコード・変更履歴・IAM権限・呼び出し権限・ログ・エラー時の挙動を確認する。変更セット作成時にもマクロが実行され得るため、意図しない副作用を避ける。
- **入力・出力の検証**：元のテンプレートと処理済みテンプレートを区別し、生成されたリソース、プロパティ、IAM権限、参照関係とクォータを確認する。マクロによる変換で元のレビュー対象が大きく変わる場合は、処理済み内容を重点的にレビューする。
- **再現性**：マクロのバージョン、デプロイ先リージョン、必要なパラメータ、外部依存を明確にする。複数環境・StackSetsで利用する場合は、対応状況と制約を個別に確認する。
- **変更セットの優先**：原則として変更セットを作成し、展開後の差分を確認してから適用する。変更セットを介さずマクロ付きテンプレートを直接作成・更新する場合に必要となる`CAPABILITY_AUTO_EXPAND`は、通常の手順の代替として安易に使用しない。

`CAPABILITY_AUTO_EXPAND`は、マクロを含むテンプレートを変更セットによる事前レビューなしで直接作成・更新する場合の承認指定であり、**変更セットを作成しただけで要求されるものではない**。IAMリソースを含む場合には、これとは別に必要なIAM系Capabilityを確認する。

CloudFormationでサポートされない一部のマクロとネストしたスタックの組み合わせなど、変更セットを使用できないケースでは、AWS公式の制約を確認し、`CAPABILITY_AUTO_EXPAND`による直接実行の可否と追加レビュー・承認方法を個別に定める。StackSetsへのマクロ適用にも利用形態による制約があるため、対象環境でのサポート状況を確認する。

AWS提供の`AWS::LanguageExtensions`を使う場合もマクロ処理が関与するが、Lambda関数を自ら実装する独自マクロとは運用・権限管理が異なる。独自マクロを実装する場合のコード規約やテスト方法は、プロジェクト固有規約に定める。

## 11. 実行環境・権限・リージョン

| 区分 | 項目           | ルール                                                                    |
| ---- | -------------- | ------------------------------------------------------------------------- |
| 必須 | 実行先         | AWSアカウント・リージョン・スタック名をデプロイ前に確認する               |
| 必須 | 認証           | 長期固定キーや秘密鍵をテンプレート・リポジトリへ埋め込まない              |
| 原則 | 実行権限       | CloudFormationサービスロールやCI/CD実行ロールに最小権限を付与する         |
| 必須 | `Capabilities` | IAMなど該当リソースを作成するとき、必要なCapabilityを確認・指定する       |
| 原則 | 環境値         | CLIプロフィール・リージョン・スタック名をREADMEや実行基盤設定で明確にする |

## 12. 変更セットとスタックのライフサイクル

| 区分 | 項目           | ルール                                                                                       |
| ---- | -------------- | -------------------------------------------------------------------------------------------- |
| 必須 | 変更確認       | 本番反映前に変更セットで作成・更新・削除・置換の差分を確認する                               |
| 原則 | 変更承認       | 重要リソースの置換・削除にはレビュー・承認手順を設ける                                       |
| 必須 | スタック外変更 | 原則としてCloudFormation管理リソースを手動変更せず、ドリフト発生時は原因と修復方針を確認する |
| 原則 | 大規模更新     | 変更範囲と依存関係を把握して段階的に実施する                                                 |
| 原則 | ロールバック   | 障害時の回復方針、手動復旧の必要性、データ退避を事前確認する                                 |

変更セットはスタックの更新前に変更内容と置換・削除の影響を確認するための仕組みであり、変更セットを確認しても更新の実行成功が保証されるわけではない。

## 13. YAMLの記述スタイルとコメント

### 13.1 基本ルール

| 区分 | 項目             | ルール                                                                                                                        |
| ---- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 必須 | インデント       | 半角スペース2文字を原則とし、タブを使用しない                                                                                 |
| 原則 | キー順           | `AWSTemplateFormatVersion`、`Description`、必要なトップレベルセクション、`Resources`、`Outputs`のように一貫した並びで記述する |
| 原則 | 引用符           | 文字列の意味が変化し得る日付・数値風の値は引用符で囲む。YAMLの型変換に注意する                                                |
| 必須 | YAML仕様         | CloudFormationが非対応のエイリアス・ハッシュマージ等を使わない                                                                |
| 原則 | コメント記法     | `#`を使用する                                                                                                                 |
| 原則 | 見出しコメント   | サービス名・リソース種別を区切り線付きで記載する                                                                              |
| 原則 | リソースの説明   | 各リソースの直前に何を定義するか・用途・設計意図・制約を説明する                                                              |
| 原則 | プロパティの説明 | 各プロパティに役割と設定方針を簡潔にコメントする。複雑な値は直前に説明してよい                                                |
| 原則 | 動的な値         | `!Ref`、`!GetAtt`、`!Sub`等の参照元と設計方針を説明し、参照先依存の具体値を固定しない                                         |
| 必須 | コメント整合性   | 設定変更時に関連コメントを更新する                                                                                            |
| 原則 | 可読性           | 網羅性と可読性が競合する場合は、設定内容と設計意図を正確に理解できることを優先する                                            |

なお、設定可能なプロパティの網羅性および設定要否の判断については、[8.2 プロパティの網羅性](#82-プロパティの網羅性) に従う。

### 13.2 記述例

以下は**単体のテンプレートとして成立する構成例**。VPCを作成し、その中に新規インスタンスへのパブリックIPv4アドレスの自動割当を無効にしたサブネットを作成する。外部通信全体の制限を実現するには、ルートテーブル・セキュリティグループ等を別途設計する。

`template.yaml`

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: VPC and private subnet example.

Parameters:
  Environment:
    Type: String # 環境名。タグの識別子に使用する
    Description: Environment name.
    Default: dev
    AllowedValues:
      - dev
      - stg
      - prd
    ConstraintDescription: dev、stg、prdのいずれかを指定する
  VpcCidr:
    Type: String # VPC用のCIDR。デプロイ時に変更できる
    Description: IPv4 CIDR block for the VPC.
    Default: 10.0.0.0/16
  SubnetCidr:
    Type: String # サブネット用のCIDR。VPCの範囲内で指定する
    Description: IPv4 CIDR block for the subnet.
    Default: 10.0.1.0/24

Resources:
  # ============================================================
  # VPC
  # ============================================================

  # サブネットを収容するVPCを定義する。
  Vpc:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: # VPCのIP範囲。パラメータから取得する
        Ref: VpcCidr
      EnableDnsSupport: true # VPCのDNS解決を有効にする
      EnableDnsHostnames: true # VPC内でDNSホスト名を利用可能にする
      Tags:
        - Key: Name
          Value: # 環境名を含めた識別用タグ
            Fn::Sub: "${Environment}-vpc"

  # ============================================================
  # Subnet
  # ============================================================

  # 内部用途のサブネットを定義する。
  # インスタンス起動時のパブリックIPv4アドレス自動割当を無効化する。
  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: # 所属VPC。同一テンプレートで作成したVPCを参照する
        Ref: Vpc
      CidrBlock: # IP範囲。環境ごとのパラメータから指定する
        Ref: SubnetCidr
      MapPublicIpOnLaunch: false # 新規インスタンスへのパブリックIPv4自動割当を無効化する
      Tags:
        - Key: Name
          Value: # 用途と環境を識別する名前
            Fn::Sub: "${Environment}-private-subnet"

Outputs:
  PrivateSubnetId:
    Description: ID of the private subnet.
    Value: # 作成したサブネットのIDを出力する
      Ref: PrivateSubnet
```

※ `VpcCidr`と`SubnetCidr`の包含関係は、上記のテンプレートだけでは完全に検証できない。実環境の構成要件に合わせて制約を追加する。また、本例はGuardによるYAML検証との互換性を考慮し、`!Ref`・`!Sub`の短縮記法ではなく`Ref`・`Fn::Sub`の完全記法を使用する。CloudFormation自体は短縮記法もサポートする。

## 14. 削除・置換保護と変更管理

| 区分 | 項目                  | ルール                                                                         |
| ---- | --------------------- | ------------------------------------------------------------------------------ |
| 原則 | `DeletionPolicy`      | データ喪失の影響が大きいリソースについて、Retain・Snapshot等の必要性を判断する |
| 原則 | `UpdateReplacePolicy` | プロパティ変更によるリソース置換時のデータ保護を検討する                       |
| 原則 | Stack保護             | 重要スタックにはtermination protectionやStack policy等を検討する               |
| 必須 | 置換                  | 更新が`Replacement`になるプロパティ変更は事前に影響を評価する                  |
| 必須 | 論理ID変更            | リソース再作成につながり得ることを認識し、変更セットと移行方針を確認する       |
| 原則 | 削除順序              | ネスト、Export/ImportValue、その他参照関係がある場合は削除制約を明示する       |

`DeletionPolicy`と`UpdateReplacePolicy`は用途が異なるため、必要に応じて両方を設計する。

## 15. セキュリティと機密情報

- パスワード、秘密鍵、APIトークン、アクセスキーをテンプレートやGit管理するパラメータファイルに埋め込まない。
- AWS Secrets ManagerやSystems Manager Parameter Storeの適切な参照方式を利用する。
- `NoEcho`は`Metadata`や`Outputs`等の秘密値を隠さない。秘密値をそれらへ出力しない。
- 動的参照をリソースの主要識別子等に含めると値が露出し得るため、対象属性を確認する。
- IAMポリシーとCloudFormationサービスロールは最小権限を基本とする。
- 必要に応じてCloudFormation Guard、CloudFormation Hooks、組織のPolicy as Codeを併用する。

## 16. Git管理と品質チェック

### 16.1 Git管理

| 区分 | 対象                                           | 方針                                                 |
| ---- | ---------------------------------------------- | ---------------------------------------------------- |
| 必須 | テンプレート、非機密パラメータ、README、テスト | バージョン管理する                                   |
| 必須 | 秘密値を含むファイル                           | Git管理しない                                        |
| 原則 | 変更履歴                                       | 重要な構成変更はPull Requestとレビューを経由する     |
| 推奨 | `.gitignore`                                   | 機密ファイル、生成物、ローカル専用ファイルを除外する |

### 16.2 検証

| 区分 | 検証                                   | 目的                                                                                                                                                              |
| ---- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 原則 | フォーマットチェック                   | 採用したFormatterでYAMLの記述形式を確認する。採用ツールと適用方法はプロジェクトで定める                                                                           |
| 必須 | `cfn-lint`                             | リソース仕様、参照、構文、プロパティ等の静的解析                                                                                                                  |
| 推奨 | `cfn-validate`                         | 追加のローカル検証（利用するツールバージョンと適用ルールを管理）                                                                                                  |
| 原則 | `aws cloudformation validate-template` | AWS APIによるテンプレート検証。AWS認証情報が必要                                                                                                                  |
| 必須 | Transform・マクロの展開後検証          | 展開によりリソース構成が変わる場合、原則として変更セット生成後の処理済みテンプレートと差分を確認する。AWS側の制約で変更セットを利用できない場合は例外手順を定める |
| 原則 | `cfn-guard validate`                   | 組織・設計ルールへの適合確認                                                                                                                                      |
| 原則 | `cfn-guard test`                       | Guardルール自体の正常系・異常系の単体テスト                                                                                                                       |
| 必須 | Change Setsレビュー                    | 本番反映前の実リソースの作成・変更・置換・削除の確認                                                                                                              |
| 推奨 | 実環境テスト                           | 専用環境へのデプロイ後、通信・アクセス・運用・復旧などを確認する                                                                                                  |
| 推奨 | CI/CD                                  | 静的解析・ポリシー検証・承認・デプロイを自動化する                                                                                                                |

**静的検証・Guardテスト・変更セットは、実環境での動作保証と同一ではない。** ツールにより検証範囲が異なるため、組み合わせて使用する。

#### 16.2.1 Formatter・Linterの利用方針

| 区分 | 項目               | ルール                                                                                                 |
| ---- | ------------------ | ------------------------------------------------------------------------------------------------------ |
| 原則 | Formatterの選定    | CloudFormation固有のYAML記法に対応し、継続的な保守が見込めるツールをプロジェクトごとに選定する         |
| 原則 | フォーマットの確認 | CI/CDやレビューで、採用したFormatterによる整形差分の有無を確認する                                     |
| 必須 | 静的解析           | `cfn-lint`でテンプレートを検査し、検出した問題を解消または理由を記録したうえで例外扱いとする           |
| 原則 | 設定の共通化       | `cfn-lint`の対象テンプレート・リージョン・適用ルール等を、必要に応じて`.cfnlintrc`で共通管理する       |
| 原則 | バージョン管理     | 利用する検証ツールのバージョンと実行方法をREADMEまたはCI/CD設定で管理する                              |
| 必須 | 整形後の確認       | 自動整形によって組み込み関数、文字列の型、コメント、参照式や設定値の意味が変わっていないことを確認する |

`rain fmt`は既存プロジェクトのFormatterとして利用してよいが、**Rainは開発元から非推奨・保守終了が案内されている**ため、新規プロジェクトでの標準採用を必須としない。既存で利用する場合も、導入済みのバージョンと整形差分を確認する。

`cfn-lint`はAWSリソース仕様の静的解析を担当する。命名・コメント・`ConstraintDescription`・`Outputs.Description`など、本規約独自のルールがすべて標準検査で担保されるわけではないため、必要に応じてカスタムルールやレビューで補う。

#### 16.2.2 規約と検証手段の対応

| 検証対象                                                    | 主な検証手段                                                                | 留意点                                                                                           |
| ----------------------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| YAMLの整形・インデント・記述順                              | 採用したFormatter、コードレビュー                                           | 自動整形前後で意味やコメントが変わらないことを確認する                                           |
| CloudFormationの構文・リソース・プロパティ・参照            | `cfn-lint`、`aws cloudformation validate-template`                          | ツールの検証範囲とAWS側での実行可能性は同一ではない                                              |
| `Fn::ForEach`の宣言・ループ構造                             | `cfn-lint`、レビュー                                                        | 静的解析で確認できる範囲は使用するツールのバージョンに依存する                                   |
| マクロ・Transformの展開結果と参照関係                       | 変更セット、`get-template --template-stage Processed`、`cfn-lint`、レビュー | AWS上で処理されたテンプレートのリソース数、論理ID、プロパティ、参照、Outputsとクォータを確認する |
| Parametersの`ConstraintDescription`、Outputsの`Description` | 規約レビュー、必要に応じた`cfn-lint`カスタムルール                          | 本規約で必須・原則にした項目は、標準検証のみでは十分に確認できない場合がある                     |
| Propertiesの網羅的な設定要否・設計意図・コメント            | AWS公式リファレンス、設計資料、人間・生成AIによるレビュー                   | 未指定プロパティの要否や設計判断は静的解析だけでは確定できない                                   |
| セキュリティ・可用性などの構成ポリシー                      | CloudFormation Guard、関連テスト、コードレビュー                            | 対象ルールとそのテストケースを別途定義する                                                       |
| 機密情報・認証情報の埋め込み                                | シークレット検査、コードレビュー                                            | `cfn-lint`だけで機密情報の混入を防げるわけではない                                               |
| 実リソースの作成・変更・削除・置換                          | 変更セットのレビュー                                                        | 変更の安全性や成功を保証するものではない                                                         |
| デプロイ後の通信・サービス動作                              | 専用環境での統合・運用テスト                                                | 静的解析やGuardでは確認できない事項を検証する                                                    |

**自動検証で判定できる事項と、設計判断が必要な事項を区別する。** 人間・生成AIによるレビューでは本規約を参照し、違反箇所・根拠・修正案を具体的に示す。特定のAIツールやSKILLの利用は必須としない。

### 16.3 実行例

以下はリポジトリのルートから実行するCLIの例。`cfn-lint`、`cfn-guard`、AWS CLIは事前インストールが必要。Formatterはプロジェクトが採用したものを使用する。

```bash
# 既存プロジェクトでrainを使用している場合のみ、整形差分を検出（AWS認証は不要）
rain fmt --verify template.yaml

# リソーススキーマ・参照等の静的解析（AWS認証は不要）
cfn-lint --regions ap-northeast-1 -t template.yaml

# ポリシールールとテンプレートの照合（AWS認証は不要）
cfn-guard validate --rules rules/network.guard --data template.yaml --type CFNTemplate

# Guardルール自体の単体テスト（AWS認証は不要）
cfn-guard test --rules-file rules/network.guard --test-data tests/network_tests.yaml

# AWS APIでテンプレート構造を検証（AWS認証・適切なリージョンが必要）
aws cloudformation validate-template --template-body file://template.yaml --region ap-northeast-1
```

`rain fmt --verify`はRainを使用している既存プロジェクト向けの参考例であり、新規導入を求めるものではない。採用したFormatterのチェックコマンドへ置き換えてよい。

本番の作成・更新では、対象アカウント・リージョン・スタックを確認したうえで、変更セットを作成・レビューし、必要な承認後に実行する。検証コマンドだけではAWSリソースは作成しない。

#### 16.3.1 Transform・マクロの展開後検証

`Fn::ForEach`やマクロを使用する場合は、**展開前のテンプレート**と**CloudFormationが変換した処理済みテンプレート**を区別して確認する。前者の`cfn-lint`や`validate-template`だけでは、展開後のリソース構成やマクロ処理の安全性を完全には保証できない。

次は、[10.2の記述例](#102-fnforeachの記述例)を新規スタック用の**未実行の変更セット**に登録し、展開後のテンプレートを取得する例。AWS CLI・AWS認証・対象リージョンでのCloudFormation利用権限が必要。スタック名・変更セット名は専用の検証用名称に変更する。

```bash
# 事前の静的解析。CloudFormationへの接続は不要
cfn-lint --regions ap-northeast-1 -t examples/foreach.yaml

# 変更セットを作成する（リソースの作成・更新はまだ実行されない）
aws cloudformation create-change-set \
  --stack-name cfn-foreach-example \
  --change-set-name foreach-review \
  --change-set-type CREATE \
  --template-body file://examples/foreach.yaml \
  --region ap-northeast-1

# 変更セットの生成を待ち、変更内容を確認する
aws cloudformation wait change-set-create-complete \
  --stack-name cfn-foreach-example \
  --change-set-name foreach-review \
  --region ap-northeast-1
aws cloudformation describe-change-set \
  --stack-name cfn-foreach-example \
  --change-set-name foreach-review \
  --region ap-northeast-1

# マクロ・Transformの処理済みテンプレートを取得する
aws cloudformation get-template \
  --stack-name cfn-foreach-example \
  --change-set-name foreach-review \
  --template-stage Processed \
  --query TemplateBody \
  --output text \
  --region ap-northeast-1 > processed-template.json

# 展開後のリソース構成を追加で静的解析する
cfn-lint --regions ap-northeast-1 -t processed-template.json

# 変更セットを実行せず削除する
aws cloudformation delete-change-set \
  --stack-name cfn-foreach-example \
  --change-set-name foreach-review \
  --region ap-northeast-1
```

この例は**まだ存在しない検証用スタックを対象にする場合**の`CREATE`変更セット。既存スタックでは`UPDATE`等、実際の状態に合う方法を選択する。変更セットを作る操作はスタックのリソースを作成しないが、**独自マクロを含む場合は、変更セットの作成中にLambdaが実行され得る**。事前に権限・費用・副作用を確認する。

処理済みテンプレートに対し、少なくとも以下を確認する。

- 反復対象の件数と生成されたリソース数、ループ論理名・展開後論理IDの一意性
- 各リソースの`Type`、`Properties`、`Condition`、`DependsOn`および参照関係
- `Outputs`の値と参照対象、生成時のプロパティ既定値、リソース置換・削除の影響
- テンプレートサイズ・リソース数などのクォータ、必要なIAM権限・Capability
- 変更セットに表示される作成・更新・置換・削除と、設計書・パラメータシートとの整合性

`get-template`の`Original`は送信したテンプレート、`Processed`はTransform・マクロの処理後のテンプレートを表す。処理済みテンプレートは、変更セットの生成が完了してから取得する。確認には変更セットを**実行する必要はない**。

### 16.4 テストコード

CloudFormationの構成ポリシーを検証する場合は、**Guardの`.guard`ファイルと、そのルールを検証するYAML/JSONテストデータ**を用いる。Guardは主として構成ポリシーを検証する。リソース間の実際の到達性・サービス挙動・ロールバックなどは別途統合テストで確認する。

| 区分 | 項目         | ルール                                                           |
| ---- | ------------ | ---------------------------------------------------------------- |
| 原則 | テスト対象   | セキュリティ、可用性、公開設定、運用上の重要な制約を優先する     |
| 原則 | ファイル配置 | `rules/`と`tests/`を分離し、対応関係がわかる名前にする           |
| 必須 | 期待結果     | `PASS`・`FAIL`・必要に応じて`SKIP`を明示する                     |
| 原則 | 異常系       | 禁止すべき設定のサンプルが失敗することを確認する                 |
| 必須 | 実行可能性   | 例示するテストには必要なルール・入力データ・実行コマンドを揃える |
| 原則 | 保守         | テンプレートや設計方針の変更時に関連ルールとテストを見直す       |

Transform・マクロを含むテンプレートでは、展開前の構文検証に加えて、反復数を変えた場合の生成リソース数、論理IDの衝突、条件分岐、参照先、必須プロパティ、削除・置換の影響をテスト観点に含める。独自マクロについては、Lambda変換処理の入力・出力、異常系、権限、再現性をプロジェクト固有のテスト計画で扱う。

CloudFormation Guardのルールを**処理済みテンプレート**に適用すれば、生成されたリソースの設定値についてポリシー検証を行える。ただしGuard単体ではTransform実行を保証せず、展開結果の取得にはAWS上の変更セット処理等が必要となる。

#### 16.4.1 記述例

次の例は、上記の`template.yaml`内のサブネットでパブリックIPv4の自動割当が無効であることを検証する。**これら2つのファイルと上記`template.yaml`を同じリポジトリに配置**する。

`rules/network.guard`

```guard
# ============================================================
# Subnet Policy
# ============================================================

# サブネットのパブリックIPv4自動割当が有効化されていないことを検証する。
let subnets = Resources.*[ Type == 'AWS::EC2::Subnet' ]

rule SUBNET_DISABLE_PUBLIC_IP when %subnets !empty {
  %subnets.Properties.MapPublicIpOnLaunch == false
}
```

`tests/network_tests.yaml`

```yaml
---
- name: PrivateSubnetPass
  input:
    Resources:
      PrivateSubnet:
        Type: AWS::EC2::Subnet
        Properties:
          MapPublicIpOnLaunch: false
  expectations:
    rules:
      SUBNET_DISABLE_PUBLIC_IP: PASS

- name: PublicSubnetFail
  input:
    Resources:
      PublicSubnet:
        Type: AWS::EC2::Subnet
        Properties:
          MapPublicIpOnLaunch: true
  expectations:
    rules:
      SUBNET_DISABLE_PUBLIC_IP: FAIL

- name: NoSubnetSkip
  input:
    Resources: {}
  expectations:
    rules:
      SUBNET_DISABLE_PUBLIC_IP: SKIP
```

実行方法：

```bash
cfn-guard test --rules-file rules/network.guard --test-data tests/network_tests.yaml
cfn-guard validate --rules rules/network.guard --data template.yaml --type CFNTemplate
```

これらのテストは**AWSリソースを作成せずに実行する構成**。CloudFormation Guardのインストールが必要。上記のサンプルは構文と想定動作を公式資料に基づいて記載したものであり、作成時点で実際の`cfn-guard`実行結果は未確認。Guardの検証は当該ポリシーへの適合性を確認するものであり、VPC全体の通信制御や実環境の挙動を保証するものではない。

## 17. プロジェクト固有規約との分担

本規約で固定しない事項は、`project-conventions.md`またはREADMEで明示する。

| 分野             | プロジェクトで定義する例                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------- |
| スタック構成     | テンプレート分割、ネスト、デプロイ順序、削除順序                                                        |
| 命名             | スタック名・論理ID・物理名・タグの接頭辞と環境識別                                                      |
| ファイル配置     | テンプレート・パラメータファイル・Guardルールの保存場所                                                 |
| リソース設計     | 採用するリソースタイプ、セキュリティ・可用性などの基準                                                  |
| 設計資料との対応 | パラメータシート・設計書と`Resources`・`Properties`の対応                                               |
| 実行環境         | AWSアカウント、リージョン、AWS CLIプロフィール、サービスロール                                          |
| パラメータ       | 環境別入力値、パラメータファイルとSecret管理                                                            |
| 変更管理         | 変更セットの承認、Stack policy、ロールバック、ドリフト検出                                              |
| 品質管理         | Formatterの選定、`cfn-lint`の適用リージョン・カスタムルール、Guardのポリシー、レビュー・CI/CDの実施方法 |

## 18. 参考資料

- [AWS CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [CloudFormation Template Sections](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)
- [CloudFormation Template Format](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-formats.html)
- [CloudFormation Resources Syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html)
- [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-template-resource-type-ref.html)
- [CloudFormation Parameters Syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)
- [CloudFormation Outputs Syntax](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html)
- [AWS::LanguageExtensions Transform](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/transform-aws-languageextensions.html)
- [Fn::ForEach](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-foreach.html)
- [Fn::ForEach - Resources Examples](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-foreach-example-resource.html)
- [CloudFormation Macros Overview](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-macros-overview.html)
- [GetTemplate API（Original / Processed）](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_GetTemplate.html)
- [CloudFormation Quotas](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)
- [CloudFormation Change Sets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)
- [Validate Templates with cfn-lint](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-lint.html)
- [cfn-lint Configuration](https://github.com/aws-cloudformation/cfn-lint#configuration)
- [Rain（非推奨・保守終了のお知らせ）](https://github.com/aws-cloudformation/rain)
- [Validate Templates with cloudformation-validate](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-validate.html)
- [Validate Templates with Guard](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-guard.html)
- [Testing CloudFormation Guard Rules](https://docs.aws.amazon.com/cfn-guard/latest/ug/testing-rules.html)
