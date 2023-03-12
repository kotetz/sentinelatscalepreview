# Sentinel at Scale

## Microsoft Cloud Security Benchmark と NIST Cybersecurity Framework

リスクが効果的に管理された状態のためには包括的なセキュリティ コントロールが組織に展開されている必要があります。セキュリティ コントロールは多岐にわたるため、適切なセキュリティ運用が行われていなくて既存の指針がないような環境でセキュリティ コントロールを展開しようとすると、幅が広すぎて何から手をつければよいかわからない、という状況になります。

Microsoft はクラウドを運用する際に必要となるセキュリティ コントロールを Microsoft Cloud Security Benchmark (MCSB) として公開しています。コントロールは適度にメンテナンスしやすい粒度になっているのでセキュリティ コントロールのベースとしてお勧めです。Defender for Cloud のセキュリティ態勢管理（無償機能）はこのコントロールに基づいて環境のリソースのセキュリティ状態を評価するため、クラウド サービスをお使いであれば導入しやすいセキュリティ基準です。Defender for Cloud については FTA Live でセッションがあるため、詳しく知りたい方はご参加ください。

[FTA Live - Defender for Cloud](https://github.com/Azure/fta-japan/blob/main/FTALive/DefenderForCloud/Pre-requisites.md)

Cybersecurity Framework (CSF) は組織が実装すべきセキュリティ コントロールをコンパクトに整理していて、粒度が細かすぎず、荒すぎず、シンプルで使いやすいという特徴があります。このため、当初は重要インフラを保護することを目的に作られましたが、様々な組織で利用されるようになっています。

組織のセキュリティ機能を Identify、Protect、Detect、Respond、Recover の 5 つのカテゴリーに分類していて、このカテゴリーは Identify から順番に実装することで効果が出やすいため、セキュリティ運用が存在しない中でセキュリティ製品を展開する必要があったり、セキュリティ運用に自信がない場合などに、多くの組織にお勧めできるフレームワークです。よく使われているセキュリティ標準（ISO 27001 / NIST SP 800-53 / CIS CSC など) へのマッピングを持つため、これらの標準を既に使っている場合には重ね合わせて使うこともできます。

![Framework Version 1.1](https://www.nist.gov/sites/default/files/styles/220_x_220_limit/public/images/2019/10/18/framework_functions_wheel.png?itok=1KLGPsFQ)

- Identify：ビジネス状況と資産を特定し、リスク アセスメントを実施する
- Protect：アクセス制御と保護を展開する
- Detect：イベントの収集、監視を行い脅威を検知するプロセスを展開する
- Respond：インシデント検知時の対応やコミュニケーションを展開する
- Recover: 復旧し、学びを反映する

Microsoft Sentinel はこの機能の中で主に Detect と Respond を担当する製品です。脅威検知製品を活用することで、特定の組織や資産に依存しない脅威の検知を行うことができますが、Identify の機能が適切であれば企業固有の資産に対する具体的な脅威を見つけることが期待できますし、Protect が適切であれば監視すべき攻撃面を小さくすることができるので、ログのコスト効率と脅威検知の効率の両方を高めることができます。逆に Identify が行われていない場合には目的のないログが大量に保存され、コストを圧迫したり、必要なログが保存されていないなどの問題が発生します。

継続的にメンテナンスされている展も重要で、現在発行されているCSF は 1.1 ですが、今後リリースされる [2.0](https://www.nist.gov/system/files/documents/2023/01/19/CSF_2.0_Concept_Paper_01-18-23.pdf) は上の 5 つの機能に加えて、Govern が追加されるようです。リスク管理は組織としての意思決定が重要ですが、この点が強調される形です。

## Microsoft Sentinel の概要

### Security Information and Event Management (SIEM)

SIEM は組織のセキュリティに関するログを収集し、正規化と関連付けを行うことで検索を可能にし、分析を行うための機能です。セキュリティに関係するログは OS やアプリケーション、サービスやネットワーク アプライアンスなど様々なコンポーネントが生成し、その生成元によって文字コードやレコードの形式、日付時刻の書式などが異なるため、データを整形して関連付けを行い、検索可能な状態にする必要があります。

SIEM では収集されたログからセキュリティ アラートを発見する役割を持ちますが、発見されたアラートは管理され、対応される必要があります。
Microsoft Sentinel は発見されたセキュリティ アラートを `インシデント` としてライフサイクルを管理しながら調査を行うための機能を持っています。この機能の中には担当者をアサインし、調査の状況を記録するものや、ログの中から意味を持つ情報を `エンティティ` として抽出し、関係するセキュリティ アラートを可視化する機能が含まれています。

- データの収集
- 分析の容易さ
- データ容量に応じた拡張性

> **Log Analytics と KQL**  
 Microsoft Sentinel はデータストアとして Log Analytics ワークスペースを使用し、Kusto Query Language (KQL) でデータ検索や操作のクエリを記述します。
KQL は Log Analytics ワークスペースで Azure Monitor や Microsoft Sentinel のログを検索する他、リソース グラフで Azure リソースの状態を確認したり、Microsoft Defender Endpoit の高度な追及の中でハンティングを行うためにも使うことができます。

### Security Orchestration, Automation and Response (SOAR)

SOAR は繰り返し発生するセキュリティ オペレーションを自動化する機能です。担当者のアサインなど簡単なものであれば GUI を使用して設定することができますが、Logic Apps と連携して複雑なワーク フローを実行することができます。自動化を行うために特別なサーバーを構築、管理したり、自動化機能のための高額なライセンスや初期投資は必要ありません。

- 発生したインシデントに対する担当者の割り当て
- 条件に応じた担当者へのメッセージ通知、Teams への投稿
- 侵害されたリソースのネットワーク隔離
- VirusTotal などの SaaS サービスと連携し、インシデントに含まれるエンティティ情報のエンリッチメント

Logic Apps は使用量に応じた課金と専用インスタンスの確認の２つのオプションがあります。

## ログのインジェスト

Sentinel は Log Analytics ワークスペースで管理されるログに対して様々な機能を提供するソリューションなので Log Analytics ワークスペースの設計が必要になります。管理や検索を簡単にするためにワークスペースは 1 つであるほうが良いですが、ベストプラクティスには複数のワークスペースを検討するための主要な考慮点が記載されています。

[Microsoft Sentinel ワークスペース アーキテクチャのベスト プラクティス](https://learn.microsoft.com/ja-jp/azure/sentinel/best-practices-workspace-architecture)

より詳細な意思決定ツリーは次のドキュメントに含まれています。

[Microsoft Sentinel ワークスペース アーキテクチャを設計する](https://learn.microsoft.com/ja-jp/azure/sentinel/design-your-workspace-architecture)

### Azure AD テナントが複数である場合

Sentinel ではデータコネクタ（後述）を使用して様々なリソースからログを取り込みますが、Azure AD のテナントが異なる場合、ログの取り込みができないものが存在します。このため、Azure AD テナントが複数ある場合にはそれぞれのテナントで Log Analytics ワークスペースを持つことを検討してください。

 [Azure Lighthouse](https://learn.microsoft.com/ja-jp/azure/lighthouse/how-to/onboard-customer) を使用するとサブスクリプションやリソースに対する権限を別のテナントのユーザーに委任することができるため、複数のテナントに跨った Sentinel  を 1 箇所から管理できるようになります。

### データのコンプライアンス要件や規制が存在する場合

地域によっては個人情報を域外に持ち出す場合に特別な前提条件や認定が必要になる場合があります。[EU の一般データ保護規則](https://blogs.microsoft.com/eupolicy/2021/05/06/eu-data-boundary/) が有名ですが、セキュリティ ログを保管する場合、これらの規制の影響を含む情報が保存される可能性があるため、ログの生成元と同じリージョンの Log Analytics ワークスペースにデータを保持した方が良い場合があります。

### データのアクセス権を分割する必要がある場合

複数のセキュリティ チームがあり、アクセスできるログを分離する必要がある場合、Log Analytics ワークスペースを分割する場合があります。例えば複数の子会社に跨ってセキュリティ監視を検討する場合などは会社ごとにワークススペースを持つことは要件としてもよくあります。リソースの管理者が自身のリソースのログにアクセスしたり、あるユーザーやグループが特定の種類のリソースのログを閲覧する権限が必要である場合、リソースに基づいた RBAC や、テーブル レベルのアクセス権を使って制御するという選択肢もあります。詳細については次のドキュメントを参照してください。

[リソースによる Microsoft Sentinel データへのアクセスを管理する](https://learn.microsoft.com/ja-jp/azure/sentinel/resource-context-rbac#explicitly-configure-resource-context-rbac)

### セキュリティに関係がない大量のデータが存在する場合

Sentinel は Log Analytics ワークスペースに対して様々な機能を追加するため、Sentinel のコストは概ね Log Analytics ワークスペースのデータの量に比例します。運用の監視などを行っており、既に Log Analytics で大量のログが存在しているような環境でワークスペースに対して Sentinel を有効化すると、セキュリティ監視ではほとんど活用されない巨大なログに対して Sentinel の課金が追加で発生することになるため、分割を検討したほうが良い場合があります。

[Azure Sentinel の価格](https://azure.microsoft.com/ja-jp/pricing/details/microsoft-sentinel/)

## コンテンツハブ

Sentinel で利用する様々な機能は必要に応じて追加することができるようになっています。利用可能な機能はコンテンツハブで管理されており、機能の種類に応じて 8 種類に分類されています。

[Microsoft Sentinel コンテンツ ハブ カタログ](https://learn.microsoft.com/ja-jp/azure/sentinel/sentinel-solutions-catalog)

- **データコネクタ**：SIEM において最も重要なコンテンツで、様々なソースからデータを取り込みます。[データコネクタ] メニューに表示されます。
- **パーサー**：取り込まれたログを検索が容易になるように整形します。Log Analytics ワークスペースの中で使われます。
- **ブック**：ダッシュボード機能です。様々なソースのログの状態やアラートを可視化し、[ブック] メニューからアクセスすることができます。
- **分析ルール**：KQL でログを分析し、セキュリティ アラートやインシデントを生成する機能です。[分析ルール] メニューに表示されます。
- **ハンティング　クエリ**：インシデントの調査で組織が実行するクエリです。[ハンティング] メニューに表示されます。
- **Notebook**：Jupyter ノートブックです。ハンティングの手順と実施するアクションを１か所にまとめることができます。[ノートブック] に表示されます。
- **Watchlist**：ログの分析を行う際の検索条件や除外条件をテーブルとして保持するための機能です。[ウォッチリスト] に表示されます。
- **プレイブックとカスタムコネクタ**：SOAR 機能で使われる Logic Apps と、Logic Apps が必要なリソースにアクセスするためのコネクタです。[オートメーション] メニューからアクセスすることができるほか、Logic Apps のリソースから直接管理することもできます。

## データコネクタ

データコネクタは Sentinel の設計において最も重要で、ソースによって様々なものがあるため、

### ネイティブのログインジェスト

[Microsoft Sentinel データ コネクタ](https://learn.microsoft.com/ja-jp/azure/sentinel/connect-data-sources)

[Microsoft 365 Defender と Microsoft Sentinel の統合](https://learn.microsoft.com/ja-jp/azure/sentinel/microsoft-365-defender-sentinel-integration)

### Microsoft Monitoring Agent / Azure Monitor Agent

### Logstash

### ログ収集の順番

### ログのコスト

#### アーカイブ機能

#### Azure Data Explorer

#### ストレージアカウントなど

#### 取り込み時データ変換

[Microsoft Sentinel のカスタム データ インジェストと変換](https://learn.microsoft.com/ja-jp/azure/sentinel/data-transformation)

## よく使うログの取り込み (ハンズオン)

### **Microsoft 365 Defender**

次のドキュメントに従って Microsoft 365 Defender を接続します。

[Microsoft 365 Defender から Microsoft Azure Sentinel にデータを接続する](https://learn.microsoft.com/ja-jp/azure/sentinel/connect-microsoft-365-defender?tabs=MDE)

Microsoft 365 Defender のデータ コネクタは 3 種類の構成を含んでいます。

#### **インシデントとアラートを接続する**

「インシデントとアラートを接続する」を選択することで次の製品のインシデントとアラートを Sentinel に取り込むことができます。

- Microsoft Defender for Endpoint
- Microsoft Defender for Identity
- Microsoft Defender for Office 365
- Microsoft Defender for Cloud Apps
- Microsoft Defender のアラート
- Microsoft Defender 脆弱性の管理
- Microsoft Purview データ損失防止
- Azure Active Directory Identity Protection

接続されたインシデントの オープン / クローズなどの状態は双方向で同期されるため、Sentinel をインシデント管理の集中ダッシュボードとして使うことができるようになります。このログは [無料データソース](https://learn.microsoft.com/ja-jp/azure/sentinel/billing?tabs=commitment-tier#free-data-sources) に含まれるため、Sentinel のコストには影響を与えません。

#### **エンティティの接続**

「エンティティの接続」では [Microsoft Derfender for Identity](https://learn.microsoft.com/ja-jp/defender-for-identity/what-is) (オンプレミスの Windows Server Active Direcotry のセキュリティ監視を行う製品) と連携し、エンティティの情報を Sentinel に取り込みます。エンティティの情報は後程扱う UEBA 機能で処理され、ユーザーのふるまいに関するインサイトを提供します。UEBA 機能は独自のテーブルを作成するため、若干のコストがかかります。

#### **イベントの接続**

「イベントの接続」 では Microsoft Defender 製品の生ログを取り込みます。このログは課金対象となり、環境によってはそれなりのログの量になります。シナリオは[このドキュメント](https://learn.microsoft.com/ja-jp/azure/sentinel/microsoft-365-defender-sentinel-integration#advanced-hunting-event-collection)で紹介されていますが、次のような要求がある場合に活用することができます。

- Sentinel が管理する様々なログと、Microsoft Defender 製品のログを関連付けて分析を行いたい
- Microsoft Defender 製品のログの保存期間を越えてログを保存しておきたい

### Defender for Cloud の連携

[Microsoft Defender for Cloud アラートを Microsoft Sentinel に接続する](https://learn.microsoft.com/ja-jp/azure/sentinel/connect-microsoft-365-defender?tabs=MDE)

### Azure Active Directory

#### Signin Logs

#### Audit Logs

### Azure Activity

## 仮想マシンの接続

### Windows のログ

### Linux のログ

### ワークブック

### XDR の確認 （ハンズオン）

## 脅威の発見

### エンティティ

[Microsoft Sentinel でエンティティを使用してデータを分類および分析する](https://learn.microsoft.com/ja-jp/azure/sentinel/entities)

### ASIM

[正規化と Advanced Security Information Model (ASIM)](https://learn.microsoft.com/ja-jp/azure/sentinel/normalization)

### Anomary

### UEBA

### Threat Intelligence

## ログの連携

### Windows VM

### Linux VM

### Azure Arc

### NVA - Log Analytics Agent

## ログの分析（ハンズオン）

### MS

[Microsoft セキュリティ アラートからインシデントを自動的に作成する](https://learn.microsoft.com/ja-jp/azure/sentinel/create-incidents-from-alerts)

### 組み込みの脅威分析ルール

[難しい設定なしで脅威を検出する](https://learn.microsoft.com/ja-jp/azure/sentinel/detect-threats-built-in)

### NRT

[Microsoft Sentinel でほぼリアルタイム (NRT) の分析ルールを使用し、脅威をすばやく検出する](https://learn.microsoft.com/ja-jp/azure/sentinel/near-real-time-rules)

### 大規模なデータセットに対する調査

[大規模なデータセット内のイベントを検索して調査を開始する](https://learn.microsoft.com/ja-jp/azure/sentinel/investigate-large-datasets)

[Microsoft Sentinel でエンティティ ページを使用してエンティティを調査する](https://learn.microsoft.com/ja-jp/azure/sentinel/entity-pages)

### 偽陽性の処理

[Microsoft Sentinel での擬陽性の処理](https://learn.microsoft.com/ja-jp/azure/sentinel/false-positives)

## SOAR