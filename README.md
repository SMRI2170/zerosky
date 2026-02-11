# zerosky

## 1. 研究背景・目的（Introduction）

### 解決したい課題
本プロジェクトは、IoTデバイス（特にドローン）が生成する位置情報や関連データの信頼性とプライバシー保護という二重の課題を解決することを目的としています。具体的には、以下の問題に取り組んでいます。
*   **位置情報のプライバシー問題:** デバイスが収集した生の位置情報がそのまま公開されることによる個人の行動履歴や機密情報の露呈リスク。
*   **データ信頼性の問題:** 収集された位置情報が改ざんされていないことの検証の難しさ、およびその検証プロセス自体の透明性の確保。
*   **分散型システムにおける検証コスト:** ブロックチェーンのような分散型台帳技術（DLT）にデータを記録する際の高いトランザクションコストとスケーラビリティの限界。

### 背景（社会問題・技術課題）
近年、ドローンやその他のIoTデバイスの普及に伴い、膨大なデータが生成されています。これらのデータ、特に位置情報は、物流、監視、自動運転など多岐にわたる分野で利用される一方で、その正確性やプライバシー保護に関する懸念が高まっています。ゼロ知識証明（ZKP）は、情報の具体的な内容を開示することなく、その情報が真実であることを証明できる革新的な技術であり、この課題に対する強力な解決策として期待されています。

### なぜこのプロジェクトが必要か
本プロジェクトは、ZKP技術をIoTデータ検証に応用することで、以下を実現します。
*   **非公開証明:** ドローンが特定の位置にいたことや、特定の経路をたどったことを、詳細な緯度・経度情報を開示せずに証明する。
*   **改ざん防止:** ブロックチェーン上にZKPを記録することで、データの真正性と非改ざん性を保証する。
*   **効率的な検証:** スマートコントラクト上でZKPを検証することで、複雑なデータ処理をオフチェーンで行いつつ、オンチェーンでの検証コストを削減する。

### 想定ユースケース
*   **ドローンの飛行経路の監査:** 特定のエリアでドローンが飛行したことを第三者に非公開で証明する。
*   **サプライチェーンの透明性確保:** 物流における荷物の位置情報をZKPで証明し、追跡の信頼性を高める。
*   **IoTデバイスの認証:** デバイスが物理的に特定の場所に存在することを証明し、セキュリティを強化する。

## 2. システム概要（Overview / Abstract）

### システムの要約
本システムは、Flutter製のモバイルクライアントアプリケーションとNode.js製のバックエンドサーバーで構成され、ゼロ知識証明（ZKP）技術を用いてIoTデバイス（ドローン）の位置情報の信頼性を検証し、その証明をブロックチェーンに記録するエンドツーエンドのソリューションです。クライアントが取得した位置情報に基づき、サーバーがZKPを生成し、IPFSに保存されたドローンデータと照合しながら、結果をブロックチェーンに永続化します。

### 主な機能
*   **モバイルクライアント（Flutter）:**
    *   カメラによる画像キャプチャ
    *   GPSによる位置情報追跡と履歴管理
    *   Bluetooth LEによるドローンのID検出
    *   取得データに基づくローカルでのPoseidonハッシュ生成と性能評価（ベンチマーク）
    *   セキュリティチェック（Root化検出、開発者モード検出など）
    *   サーバーへのデータ提出
    *   ローカルデータベースへの記録保存と管理
*   **バックエンドサーバー（Node.js）:**
    *   クライアントからのデータ（画像、ハッシュ、ドローンID）受信と保存
    *   IPFSへのデータアップロードと監視（IPNS経由）
    *   Circom回路を用いたZKP（Groth16）の生成
    *   生成されたZKPのブロックチェーン（Ethereum Sepolia）への記録
    *   提出状況の管理とトラッキング

### 技術スタック
*   **フロントエンド:** Flutter (Dart)
*   **バックエンド:** Node.js, Express.js
*   **ゼロ知識証明:** Circom, `snarkjs` (Groth16)
*   **ブロックチェーン連携:** `ethers.js` (Ethereum Sepolia)
*   **分散ストレージ:** IPFS
*   **データベース:** SQLite (クライアント側), ファイルシステム (サーバー側)
*   **セキュリティ:** `safe_device` (Flutter)

### 想定ユーザ
*   ドローン運用企業: ドローンの飛行証明や監査を必要とする企業
*   規制当局: ドローンの不正利用防止や飛行経路の検証に関心のある組織
*   ZKP開発者: ZKPの実世界適用に関心のある技術者・研究者
*   プライバシーを重視するユーザー: 自身の位置情報が安全に扱われることを求める個人

## 3. アーキテクチャ設計（Architecture）

### システム構成図
```
+-------------------+           +-------------------+           +-----------------------+
|  Mobile Client    |           |  Backend Server   |           |    Blockchain Network |
| (Flutter App)     |           |  (Node.js/Express)|           |    (Ethereum Sepolia) |
+-------------------+           +-------------------+           +-----------------------+
        |                                 |                                 ^
        | 1. Image/Location/Drone ID      |                                 |
        |    Capture + Hash Generation    |                                 |
        |                                 | 3. Proof Data + Tx              |
        v                                 v                                 |
+-------------------+             +-------------------+             +-----------------------+
| Local Database    |             |  IPFS Gateway     |<------------|  Smart Contract       |
| (SQLite)          |<----------->|  (Data Storage)   |             |  (ZKP Verifier)       |
+-------------------+             +-------------------+             +-----------------------+
        ^                                 ^                                 ^
        |                                 | 2. ZKP Generation + IPFS Update |
        |                                 |                                 |
        |                             +-------------------+                 |
        |                             |  ZKP Circuit      |                 |
        |                             |  (Circom/snarkjs) |                 |
        |                             +-------------------+                 |
        |                                 ^                                 |
        |                                 |                                 |
        +---------------------------------+---------------------------------+
```
<!-- (Note: This is a textual representation. A proper diagram would be an image.) -->

### データフロー
1.  **データ取得:** モバイルクライアントはカメラで画像を撮影し、GPSから位置情報を取得し、Bluetooth LEで特定のドローンIDを検出します。
2.  **ローカル処理:** 取得した位置情報と撮影時刻に基づき、複数の時間ウィンドウでPoseidonハッシュを生成し、その性能をベンチマークします。
3.  **サーバー提出:** クライアントは画像ファイル、ドローンID、生成されたハッシュをバックエンドサーバーの `/submit` エンドポイントに送信します。
4.  **サーバー側データ保存:** サーバーは提出されたデータを `uploads/` ディレクトリに一時保存し、提出状況を `submission-status.json` に記録します。
5.  **IPFSへのアップロード:** サーバーは提出ディレクトリ全体をIPFSにアップロードし、そのCIDを記録します。
6.  **IPFSウォッチャー:** サーバーは定期的にIPFS/IPNSを監視し、特定のドローンIDに関連するデータ更新（例えば、ドローンの公開鍵やその他のメタデータ）を検出します。
7.  **ZKP生成:** IPFSから取得したドローンデータとクライアントから提出されたハッシュ（Private Input）を用いて、Circom回路 (`merkle_js/merkle.wasm`) と `snarkjs` (`merkle_final.zkey`) を介してGroth16証明を生成します。
8.  **ブロックチェーン記録:** 生成されたZKPの公開入力と証明（calldata）は、`ethers.js` を用いて、スマートコントラクトの `recordProof` 関数を通じてEthereum Sepoliaブロックチェーンに記録されます。トランザクションハッシュが発行され、提出状況が更新されます。

### コンポーネント説明
*   **Mobile App (Flutter):** ユーザーインターフェースを提供し、物理デバイスからのデータ（カメラ、GPS、BLE）を収集・前処理します。ローカルデータベースとセキュリティチェック機能を持ちます。
*   **Backend Server (Node.js):** クライアントとブロックチェーン/IPFSの中継役。データ提出の受け入れ、ZKP生成処理のオーケストレーション、IPFSへの公開、ブロックチェーンへの記録を担当します。
*   **Blockchain (Ethereum Sepolia):** 不変の台帳としてZKPの検証結果を永続的に記録します。ZKP検証用のスマートコントラクトがデプロイされています。
*   **ZKP Circuit (Circom/snarkjs):** サーバー上で実行され、提供された入力が特定の条件（例：特定のハッシュがドローンデータと一致する）を満たすことを証明するゼロ知識証明を生成します。
*   **IPFS (InterPlanetary File System):** 分散型ストレージシステム。ドローンデータや提出物の証拠データを非中央集権的に保存するために利用されます。

## 4. 技術仕様（Technical Details / Method）

### 使用技術
*   **クライアント:**
    *   **Flutter (Dart):** UIフレームワーク
    *   `camera`: カメラ制御
    *   `location`: GPSアクセス
    *   `flutter_blue_plus`: Bluetooth LEスキャン
    *   `sqflite`: ローカルDB
    *   `poseidon`: Poseidonハッシュ関数
    *   `safe_device`: デバイスセキュリティチェック
*   **サーバー:**
    *   **Node.js:** ランタイム環境
    *   `Express.js`: Webアプリケーションフレームワーク
    *   `multer`: ファイルアップロード処理
    *   `ethers.js`: Ethereum連携
    *   `dotenv`: 環境変数管理
    *   `ipfs-http-client`: IPFS連携
    *   `child_process`: 外部コマンド実行 (`snarkjs`)
*   **ZKP:**
    *   **Circom:** ZKP回路開発言語
    *   **`snarkjs`:** ZKPプロバー/ベリファイアツール (Groth16実装)
    *   `merkle_js/merkle.wasm`: Witness計算用のWASMファイル
    *   `merkle_final.zkey`: Groth16証明生成用の最終的なZKey
*   **データストア:**
    *   クライアント: SQLite (`zkp_verifier_v3.db`)
    *   サーバー: ファイルシステム (`uploads/`, `submission-status.json`, `ipns-names.json`)

### API仕様
#### サーバーエンドポイント
*   **`POST /submit`**
    *   **説明:** クライアントから ZKP 生成用のデータ（画像、ハッシュ、ドローンID、撮影時刻、期間）を受け付けます。
    *   **形式:** `multipart/form-data`
    *   **フィールド:**
        *   `image`: 画像ファイル (`req.file`)
        *   `hash`: ハッシュ値の改行区切り文字列 (`req.body.hash`)
        *   `droneId`: ドローンID (`req.body.droneId`)
        *   `captureTime`: 撮影時刻 (ISO 8601形式文字列, `req.body.captureTime`)
        *   `durationSeconds`: 選択された期間 (`req.body.durationSeconds`)
    *   **レスポンス:** `JSON { message: '申請を受け付けました。', submissionId: <string> }`
*   **`GET /submission-status/:submissionId`**
    *   **説明:** 特定の提出IDの処理ステータスを返します。
    *   **形式:** `GET`
    *   **パスパラメータ:** `submissionId`
    *   **レスポンス:** `JSON { status: <string>, ... }` (例: `submitted`, `processing`, `proof_generated`, `completed`, `failed`)
*   **`POST /record-on-chain`**
    *   **説明:** サーバーで生成された ZKP をブロックチェーンに記録します。ZKP生成はバックグラウンドプロセス (`processAllSubmissions`) で行われるため、このエンドポイントは実際には `processAllSubmissions` から呼び出されるコントラクト呼び出しをトリガーするもので、クライアントから直接呼び出されるものではありません。
    *   **形式:** `POST`
    *   **レスポンス:** `JSON { transactionHash: <string> }`
*   **`GET /get-proof-events`**
    *   **説明:** スマートコントラクトから `ProofRecorded` イベントのログを取得し、フォーマットして返します。
    *   **形式:** `GET`
    *   **レスポンス:** `JSON Array of { transactionHash, blockNumber, timestamp, pubSignals }`

### データ構造
*   **`LocalRecord` (クライアント側 - SQLite):**
    *   `id`: INTEGER PRIMARY KEY AUTOINCREMENT
    *   `capture_time`: TEXT (ISO 8601)
    *   `duration_seconds`: INTEGER (最大期間)
    *   `drone_id`: TEXT
    *   `image_path`: TEXT (ローカルファイルパス)
    *   `hashes`: TEXT (JSON形式で `Map<int, List<String>>` を格納)
*   **`ProcessLog` (クライアント側 - UI表示用):**
    *   `name`: `ProcessName` (hashGeneration, applicationSubmission, chainRecording)
    *   `status`: `ProcessStatus` (pending, inProgress, completed, error)
    *   `duration`: `Duration`
    *   `errorMessage`: `String?`
    *   `submissionId`: `String?`
*   **`submission-status.json` (サーバー側):**
    *   キー: `submissionId` (文字列)
    *   値: `JSON Object { status: <string>, ipfsCid: <string>, transactionHash: <string>, blockNumber: <number>, performance: <object>, error: <string>, lastUpdated: <ISO 8601 string> }`
*   **ZKP入力 (Circom `input.json`):**
    *   `hashes`: クライアントから提出されたハッシュの配列（プライベート入力の一部）
    *   `droneData`: IPFSから取得したドローンデータ（公開入力の一部）
        *   例: `{ publicKey: "...", sensorData: {...} }` (詳細は `ipfs/retrieve-data.mjs` および `prepare_inputs.js` に依存)

### プロトコル設計
1.  **クライアント-サーバー通信:** HTTPS (SSL/TLS) を使用して暗号化された通信路を確立します。自己署名証明書 (`key.pem`, `cert.pem`) が使用されています。
2.  **ZKP生成プロトコル (Groth16):**
    *   **Setup:** `snarkjs groth16 setup` で生成された `merkle_final.zkey` を使用。
    *   **Witness計算:** `snarkjs wtns calculate <wasm> <input.json> <witness.wtns>`
    *   **Proof生成:** `snarkjs groth16 prove <zkey> <witness.wtns> <proof.json> <public.json>`
    *   **Calldata生成:** `snarkjs zkey export soliditycalldata <public.json> <proof.json>`
3.  **ブロックチェーン連携:** `ethers.js` を介してEthereum RPCに接続し、スマートコントラクトの `recordProof` 関数を呼び出します。イベント監視には `queryFilter` を使用します。
4.  **IPFS/IPNS:** `ipfs-http-client` を使用してIPFSノードと連携。IPNSを用いて特定のドローンIDに関連する最新のコンテンツハッシュ（CID）を追跡します。

### アルゴリズム
*   **Poseidon Hash:** クライアント側で位置情報データ（緯度、経度、タイムスタンプ）をZKPフレンドリーなハッシュ関数でハッシュ化します。このハッシュは複数のポイント（中心点と周囲の点）に対して計算されます。
*   **Groth16:** サーバー側で、Circomで定義された回路に基づいてGroth16プロトコルを用いてゼロ知識証明を生成します。具体的な回路ロジックは `merkle_js/merkle.circom` （または関連ファイル）に実装されていますが、提出されたハッシュとIPFS上のドローンデータとの照合ロジックが含まれていると推測されます。

## 5. セットアップ方法（Setup / Installation）

本プロジェクトを実行するには、クライアント（Flutter）とサーバー（Node.js）の両方の環境設定が必要です。

### 必要環境
*   **Flutter SDK:** 最新安定版
*   **Node.js:** v18以上 (LTS推奨)
*   **npm または Yarn:** Node.jsパッケージマネージャー
*   **Git:** ソースコード管理
*   **IPFS Daemon (Optional):** ローカルのIPFSノードと連携する場合
*   **EthereumウォレットとSepoliaテストネットのETH:** ブロックチェーンへのトランザクション発行用

### 依存関係
#### クライアント (Flutter)
`pubspec.yaml` に記載されている依存関係は、`flutter pub get` コマンドで自動的に解決されます。

#### サーバー (Node.js)
`package.json` に記載されている依存関係は、`npm install` または `yarn install` でインストールされます。

### インストール手順
1.  **リポジトリのクローン:**
    ```bash
    git clone [このリポジトリのURL]
    cd poseidon_hash_zkp/poseidon_client
    ```
2.  **クライアントのセットアップ:**
    ```bash
    cd lib
    flutter pub get
    # iOSの場合 (初回のみ)
    # cd ios && pod install && cd ..
    ```
3.  **サーバーのセットアップ:**
    ```bash
    cd server
    npm install # または yarn install
    ```
4.  **ZKP回路のコンパイルとTrusted Setup (開発時のみ):**
    ZKP回路（例: `circom/merkle.circom`）は事前にコンパイルされ、`merkle_js/merkle.wasm` と `merkle_final.zkey` が生成されている必要があります。これらのファイルがない場合、`circom` と `snarkjs` をインストールし、以下の手順を実行します。
    ```bash
    # 例: Circomとsnarkjsのインストール
    # npm install -g circomlibjs snarkjs
    
    # Circom回路のコンパイル
    # circom circom/merkle.circom --wasm --r1cs -o circom/
    # mv circom/merkle_js/merkle.wasm merkle_js/
    
    # Trusted Setup (pot19_final.ptau は別途取得)
    # snarkjs groth16 setup circom/merkle.r1cs pot19_final.ptau merkle_final.zkey
    ```

### 環境変数
サーバーの `server/.env` ファイルを作成し、以下の変数を設定してください。
*   `SEPOLIA_RPC_URL`: SepoliaテストネットのRPCエンドポイントURL (例: Alchemy, Infura)。
*   `PRIVATE_KEY`: Ethereumウォレットの秘密鍵。トランザクションの署名に使用されます。**本番環境では絶対に直接コードに含めないでください。**

## 6. 使用方法（Usage）

### サーバーの起動
プロジェクトのルートディレクトリから以下のコマンドを実行してサーバーを起動します。
```bash
node server.cjs
```
サーバーは `https://localhost:3000` でリッスンを開始します。自己署名証明書を使用しているため、クライアント側でHTTPオーバーライド設定が必要です (例: `lib/main.dart` の `MyHttpOverrides` クラス)。

### クライアントアプリの実行
Flutterプロジェクトのルートディレクトリで以下のコマンドを実行し、アプリをデバイスまたはエミュレータにデプロイします。
```bash
flutter run
```

### デモ手順
1.  **クライアントアプリの起動:** デバイスでアプリを開きます。セキュリティチェックがパスすると、カメラ画面が表示されます。
2.  **ドローンの検出:** アプリはBluetooth LEで特定のドローンID (`_targetDroneId` = `D8:3A:DD:E2:55:36`) を検索します。ドローンが検出されるまで待ちます（表示ステータス: "接続中: D8:3A:DD:E2:55:36"）。
3.  **GPS捕捉:** アプリはGPS位置情報を捕捉します（表示ステータス: "GPS捕捉中..."）。
4.  **写真撮影:** 画面下部のシャッターボタンをタップします。
5.  **性能評価:** アプリは複数の期間（10秒〜60秒）で30回ハッシュ生成を行い、平均処理時間を計測・表示します。この間、"性能評価実行中..."のダイアログが表示されます。
6.  **性能評価レポート:** 性能評価が完了すると、結果がダイアログとして表示されます。CSV形式でコピーするオプションもあります。
7.  **検証画面への遷移:** レポートダイアログで「次へ進む」をタップし、検証画面に進みます。
8.  **証明期間の選択:** 検証画面で、サーバーに提出するハッシュの期間（例: 30秒）を選択します。
9.  **サーバーへ提出:** 「サーバーへ提出」ボタンをタップします。画像と選択された期間のハッシュがサーバーに送信されます。
10. **ブロックチェーン記録:** サーバーでのZKP生成が完了し次第、「ブロックチェーン記録」ボタンが有効になります。これをタップすると、生成されたZKPがEthereum Sepoliaに記録されます。
11. **トランザクション確認:** 記録が完了すると、トランザクションハッシュが表示され、Etherscanで詳細を確認できます。

### API使用例 (サーバー)
サーバーの `/submit` エンドポイントは、`multipart/form-data` 形式でPOSTリクエストを受け付けます。
```javascript
// 例: Node.js fetch API を使用
const FormData = require('form-data');
const fs = require('fs');

const formData = new FormData();
formData.append('image', fs.createReadStream('/path/to/your/image.jpg'));
formData.append('hash', 'hash1\nhash2\nhash3'); // クライアントが生成したハッシュ
formData.append('droneId', 'D8:3A:DD:E2:55:36');
formData.append('captureTime', new Date().toISOString());
formData.append('durationSeconds', '30');

fetch('https://localhost:3000/submit', {
    method: 'POST',
    body: formData,
    // 自己署名証明書の場合、Node.jsで無視する設定が必要になることがあります
    // agent: new (require('https').Agent)({ rejectUnauthorized: false })
})
.then(res => res.json())
.then(data => console.log(data))
.catch(error => console.error(error));
```

## 7. 実験・評価（Evaluation / Benchmark）

### 性能評価
クライアントアプリケーションには、ZKP生成の入力となるPoseidonハッシュの計算時間を計測する機能が組み込まれています。
*   **計測内容:** 指定された期間（例: 10秒、20秒、...、60秒）ごとに、位置情報に基づくPoseidonハッシュを30回生成し、その平均処理時間（ミリ秒）を計測します。
*   **表示形式:** アプリケーション内で表形式のレポートとして表示され、CSV形式でクリップボードにコピー可能です。
*   **目的:** 異なる期間設定がハッシュ生成の計算コストにどのように影響するかを評価し、最適なパラメーター設定やデバイスの処理能力の把握に役立てます。

### 実験環境
*   **クライアント:**
    *   デバイス: (例: Androidスマートフォン Pixel 7, iPhone 14)
    *   OS: (例: Android 13, iOS 16)
    *   CPU: (例: Tensor G2, A15 Bionic)
*   **サーバー:**
    *   OS: (例: macOS Sonoma, Ubuntu 22.04 LTS)
    *   CPU: (例: Apple M1, Intel Core i7)
    *   RAM: (例: 16GB)
*   **ネットワーク:** Wi-Fiまたは有線LAN (サーバーとクライアント間の通信用)
*   **ブロックチェーン:** Ethereum Sepoliaテストネット

### 結果
(このセクションは、実際の実験結果に基づいて記述する必要があります。例として、以下のような内容が考えられます。)
*   「30秒間のハッシュ生成（30回平均）は、[デバイス名]上で約 [X] ms で完了しました。」
*   「サーバー上でのGroth16証明生成は、[Y] 秒かかり、ブロックチェーンへの記録には約 [Z]秒を要しました。」
*   「クライアント側のセキュリティチェックのオーバーヘッドは無視できるレベルでした。」

### 比較
(このセクションも、既存の研究や他手法との比較に基づいて記述する必要があります。)
*   「従来のブロックチェーンへの直接データ記録と比較して、ZKPを用いることでオンチェーントランザクションのガス代を [N]% 削減できました。」
*   「類似のZKPベースの位置情報証明システムとの比較において、本システムはクライアント側の処理性能とオフチェーンデータの効率的な管理において優位性を示しました。」

## 8. セキュリティ設計（Security Consideration）

本プロジェクトは、ZKP、IoT、ブロックチェーンといったセキュリティが極めて重要な技術を組み合わせているため、多層的なセキュリティ設計を施しています。

### 脅威モデル
*   **悪意のあるクライアント:** 偽の位置情報、偽のドローンID、改ざんされたハッシュをサーバーに提出しようとする。
*   **サーバーの侵害:** サーバーがハッキングされ、ZKP生成プロセスやIPFS/ブロックチェーン連携が悪用される。
*   **ネットワーク攻撃:** クライアントとサーバー間の通信傍受、改ざん、リプレイ攻撃。
*   **デバイスの侵害:** クライアントデバイスのRoot化/ジェイルブレイクにより、アプリの動作が改ざんされる。
*   **ZKP回路の脆弱性:** 回路設計の不備により、誤った証明が生成されたり、プライバシーが漏洩したりする。
*   **スマートコントラクトの脆弱性:** コントラクトのバグにより、意図しない動作や資産の損失が発生する。

### 攻撃耐性
*   **ZKPによる非公開性:** 生の位置情報やドローンデータの詳細を公開することなく、特定の事実（例：ドローンが特定の時間帯に特定の範囲内にいた）を証明することで、プライバシーを保護し、情報の直接的な窃取に対する耐性を持ちます。
*   **ブロックチェーンによる不変性:** 一度ブロックチェーンに記録されたZKPは改ざん不可能であり、証明の信頼性を保証します。
*   **セキュアな通信:** クライアントとサーバー間の通信はHTTPS (`key.pem`, `cert.pem`) で暗号化されており、盗聴や中間者攻撃を防ぎます。
*   **デバイスのRoot化対策:** クライアントアプリは `safe_device` ライブラリを用いてデバイスのRoot化/ジェイルブレイクや開発者モードの有効化を検知し、安全でない環境での実行を拒否します。これにより、アプリのコード改ざんや機密情報（例：ローカルDB）への不正アクセスリスクを低減します。
*   **分離された責任:** クライアントはデータ収集と提出、サーバーはZKP生成とオンチェーン記録という形で責任が分離されており、単一障害点のリスクを軽減します。

### プライバシー設計
*   **ZKPによるデータ秘匿:** ユーザーの具体的な位置情報（緯度、経度）はブロックチェーン上に直接記録されず、ZKPを介して抽象化された事実のみが公開されます。これにより、個人の移動履歴が追跡されるリスクを最小限に抑えます。
*   **IPFS利用:** ドローンデータのような補助的な情報は、中央集権的なサーバーではなくIPFSに保存され、その可用性と耐検閲性を高めます。

## 9. 制限事項（Limitations）

本プロジェクトの現在の実装には、以下の制限事項があります。

*   **ZKP回路の複雑性:** 現在のZKP回路は特定の用途（例: 位置情報の範囲証明）に特化しており、より複雑なロジックや大規模な入力データに対応するには回路の再設計や最適化が必要です。
*   **スケーラビリティ:** サーバー側でのZKP生成は計算コストが高く、大量の同時提出があった場合、処理の遅延やサーバーリソースのボトルネックが発生する可能性があります。
*   **ネットワーク依存性:** IPFSやブロックチェーンネットワークへの接続性、およびそのパフォーマンスがシステム全体の応答時間に影響を与えます。
*   **自己署名証明書:** サーバーが自己署名証明書を使用しているため、クライアント側での証明書検証ロジックの追加や、信頼できるCAが発行した証明書への置き換えが必要です。
*   **ドローンデータの一般化:** 現在のドローンデータ取得はIPNS監視に依存していますが、多様なドローンモデルやデータ形式に対応するための一般化が必要です。
*   **バッテリー消費:** クライアントアプリのGPS追跡やBluetoothスキャンは、デバイスのバッテリーを比較的多く消費する可能性があります。
*   **環境構築の複雑さ:** Flutter、Node.js、IPFS、Circom/snarkjs、Ethereumといった多様な技術スタックを網羅するため、開発環境のセットアップが複雑になる可能性があります。

## 10. 今後の展望（Future Work）

本プロジェクトは、以下の方向で改善・発展を計画しています。

*   **より高度なZKP機能の導入:**
    *   より複雑な地理空間クエリ（例: 特定の時間内に複数のチェックポイントを通過した証明）に対応するZKP回路の開発。
    *   ZK-RollupやStarkwareなどのスケーリングソリューションとの統合による、オンチェーン検証コストのさらなる削減。
*   **マルチチェーン対応:** Ethereumだけでなく、Polygon、Solanaなどの他のブロックチェーンネットワークへの対応。
*   **デバイスインテグレーションの強化:**
    *   より広範なIoTデバイスタイプ（例: 車載センサー、ウェアラブルデバイス）からのデータ収集サポート。
    *   ハードウェアレベルのセキュリティ機能（例: TEE (Trusted Execution Environment)）との連携による、データ取得時点での改ざん耐性向上。
*   **リアルタイム処理の最適化:** サーバー側でのZKP生成処理の並列化や分散化、WebAssemblyへのオフロードなどによる性能向上。
*   **ユーザーインターフェースの改善:** クライアントアプリのUX/UIをさらに洗練し、提出状況の視覚化やエラーハンドリングを強化。
*   **標準化への貢献:** IoTデバイスとZKP、ブロックチェーンを組み合わせたデータ検証の標準プロトコルやフレームワークの研究開発。
*   **エコシステム統合:** 既存のIoTプラットフォームやデータマーケットプレイスとの連携。

## 11. 貢献方法（Contributing）

本プロジェクトへの貢献を歓迎します。以下のガイドラインに従ってご協力ください。

*   **Issueの作成:** バグ報告、機能リクエスト、改善提案は、GitHubのIssueトラッカーを通じて行ってください。
*   **Pull Request (PR) の作成:**
    *   変更は意味のある小さな単位にまとめてください。
    *   各PRは一つの特定の課題または機能に対応するようにしてください。
    *   既存のコーディング規約とスタイルを尊重してください。
    *   PRを提出する前に、関連するテストを作成し、パスすることを確認してください。
    *   詳細なコミットメッセージとPR説明を含めてください。
*   **ブランチ戦略:** `main` ブランチは常に安定した状態を保ちます。新機能やバグ修正は `feature/your-feature` や `bugfix/issue-number` のようなトピックブランチを作成して作業し、`main` へのPRとして提出してください。

## 12. ライセンス（License）

(このプロジェクトのライセンス情報をここに記載してください。例: MIT License, Apache 2.0 Licenseなど)

## 13. 参考文献（References）

*   **ゼロ知識証明に関する論文:**
    *   [Groth16: On the Size of Pairing-based Non-interactive Arguments](https://eprint.iacr.org/2016/260)
    *   [Circom Documentation](https://docs.circom.io/)
    *   [snarkjs Documentation](https://docs.snarkjs.io/)
*   **Poseidon Hash 関数に関する情報:**
    *   [Poseidon: A New Hash Function for ZKP](https://www.poseidon-hash.info/)
*   **IPFS と IPNS:**
    *   [IPFS Documentation](https://docs.ipfs.tech/)
    *   [IPNS Documentation](https://docs.ipfs.tech/concepts/ipns/)
*   **Ethereum および `ethers.js`:**
    *   [Ethereum Documentation](https://ethereum.org/ja/developers/docs/)
    *   [ethers.js Documentation](https://docs.ethers.org/v6/)
*   **Flutter:**
    *   [Flutter Documentation](https://docs.flutter.dev/)
