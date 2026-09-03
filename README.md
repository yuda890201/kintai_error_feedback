# kintai_error_feedback

勤怠エラー改善指導シート作成 & 集計アプリ（Firebase連携版）

`index.html` を開くだけで動く単一ファイルのWebアプリです。従業員マスターと月次の勤怠エラー指導記録を
Firebase（Firestore / Storage / Authentication）に保存するため、複数の店長・端末間でデータが自動的に同期されます。

## セットアップ手順

### 1. Firebaseプロジェクトを作成

1. [Firebaseコンソール](https://console.firebase.google.com/) にアクセスし、新しいプロジェクトを作成します。
2. 「Webアプリを追加」を選び、表示された `firebaseConfig` の値（apiKey, authDomain, projectId など）を控えます。

### 2. 各機能を有効化

- **Authentication** → Sign-in method で「匿名」を有効化します（ログイン画面なしでFirestore/Storageにアクセスするため）。
- **Firestore Database** を作成します（本番環境モードで作成してOK。ルールは下記で設定）。
- **Storage** を有効化します（ストコン画面の写真アップロード先）。

### 3. `index.html` に設定を反映

`index.html` 内の `firebaseConfig`（`<script type="module">` の先頭付近）を、手順1で控えた自分のプロジェクトの値に書き換えます。

```js
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
};
```

未設定のままだと、画面上部に「⚠ Firebase未設定」という警告バナーが表示されます。

### 4. セキュリティルールを設定

匿名認証済みユーザーのみ読み書きできるように設定する例です（社内利用が前提の簡易ルール）。

**Firestore ルール**

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

**Storage ルール**

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

## データ構造

- `employees`（Firestoreコレクション、ドキュメントID = 従業員番号）: `{ id, name, store }`
- `records`（Firestoreコレクション、ドキュメントID = `従業員番号_対象月`）: `{ empId, empName, month, errorCount, errors, imageUrl, updatedAt }`
- ストコン画面の写真は Storage の `stcon/` 配下にアップロードされ、そのダウンロードURLが `records` の `imageUrl` に保存されます。

## 使い方

1. `index.html` をブラウザで開きます（ローカルファイルとして開くか、任意のWebサーバーで配信します）。
2. 画面上部が「✅ Firebaseに接続済み」になれば準備完了です。
3. 左側パネルで従業員情報・対象月・発生エラー・ストコン画面の写真・特記事項を入力し、「AI総括文を作成・反映」→「この記録をデータ保存する」の順に操作します。
4. 「月次エラー集計」タブで、Firestoreに保存された指導記録・従業員一覧を確認できます（他の端末で保存したデータもリアルタイムに反映されます）。
