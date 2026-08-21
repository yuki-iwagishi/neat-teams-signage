# neat-teams-signage

**日本語** ｜ [English](#english)

Microsoft Teams Rooms（Neat デバイス）のデジタルサイネージで表示する Web コンテンツです。
Azure Static Web Apps（Free）でホスティングし、その URL を Teams Rooms Pro 管理ポータルの
Custom（Web URL）ソースに登録して使います。

## 構成

```
├── index.html                 # サイネージ本体（表示するページ）
├── staticwebapp.config.json   # レスポンスヘッダー設定（CSP frame-ancestors）
└── .github/workflows/         # Azure Static Web Apps への自動デプロイ（GitHub Actions）
```

## 要点：CSP frame-ancestors

Teams Rooms は指定した URL を **サンドボックス iframe 内で読み込む**ため、ページ側で
`teams.microsoft.com` からの埋め込みを許可する必要があります。これを `staticwebapp.config.json`
で付与しています（`<meta>` では効かないため、レスポンスヘッダーで返す）。

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "frame-ancestors https://teams.microsoft.com/"
  }
}
```

## デプロイ

`main` に push すると GitHub Actions が走り、自動で Azure Static Web Apps に反映されます。
発行される URL は Azure Portal の Static Web App「概要」で確認できます（以下の
`<YOUR-APP>` は自分の環境の値に読み替えてください）。

### 注意：ビルドをスキップする

素の静的 HTML のため、ビルド工程は不要です。ワークフローがビルドを試みて失敗する場合は、
`.github/workflows/azure-static-web-apps-*.yml` の `Azure/static-web-apps-deploy@v1` の
`with:` ブロックに次を追加します。

```yaml
          skip_app_build: true
```

## 動作確認

```bash
# ヘッダーが返るか
curl -sI https://<YOUR-APP>.azurestaticapps.net/ | grep -i content-security-policy
# → content-security-policy: frame-ancestors https://teams.microsoft.com/

# 本体が返るか
curl -s https://<YOUR-APP>.azurestaticapps.net/ | head -5
```

ブラウザの InPrivate ウィンドウで開き、認証なしで表示されることも確認します。

## 使い方（Pro 管理ポータル）

Teams Rooms Pro 管理ポータル → Settings → Digital signage → Add source → **Custom** に
発行された URL を登録し、対象の Neat デバイス（ルーム）へ割り当てます。

## メモ

- 本リポジトリは検証用の構成です。本番運用は、組織で所有・管理できる Azure サブスクリプション
  （またはそれに準じるホスティング）へ載せ替えてください。
- サイネージは公開・認証不要が前提のため、機密情報は載せないこと。
- コンテンツ差し替え時も `staticwebapp.config.json` は残すこと（ヘッダーが消えると表示されません）。

---

# English

[日本語](#neat-teams-signage) ｜ **English**

Web content shown as digital signage on Microsoft Teams Rooms (Neat devices).
It is hosted on Azure Static Web Apps (Free), and its URL is registered as a
Custom (web URL) source in the Teams Rooms Pro Management portal.

## Structure

```
├── index.html                 # The signage page itself
├── staticwebapp.config.json   # Response header config (CSP frame-ancestors)
└── .github/workflows/         # Auto-deploy to Azure Static Web Apps (GitHub Actions)
```

## Key point: CSP frame-ancestors

Teams Rooms loads the given URL **inside a sandboxed iframe**, so the page must
allow being framed by `teams.microsoft.com`. This is set via `staticwebapp.config.json`
(it must be a response header — a `<meta>` tag does not work for `frame-ancestors`).

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "frame-ancestors https://teams.microsoft.com/"
  }
}
```

## Deploy

Push to `main` and GitHub Actions deploys automatically to Azure Static Web Apps.
Find the generated URL in the Azure Portal under the Static Web App "Overview"
(replace `<YOUR-APP>` below with your own value).

### Note: skip the build

This is plain static HTML, so no build step is needed. If the workflow tries to
build and fails, add the following to the `with:` block of
`Azure/static-web-apps-deploy@v1` in `.github/workflows/azure-static-web-apps-*.yml`:

```yaml
          skip_app_build: true
```

## Verify

```bash
# Header present?
curl -sI https://<YOUR-APP>.azurestaticapps.net/ | grep -i content-security-policy
# → content-security-policy: frame-ancestors https://teams.microsoft.com/

# Body served?
curl -s https://<YOUR-APP>.azurestaticapps.net/ | head -5
```

Also open the URL in a browser InPrivate window and confirm it loads without sign-in.

## Usage (Pro Management portal)

Teams Rooms Pro Management portal → Settings → Digital signage → Add source →
**Custom**, register the generated URL, then assign it to the target Neat device (room).

## Notes

- This repository is a test setup. For production, move it to an Azure subscription
  (or equivalent hosting) that the organization can own and manage.
- Signage must be public and require no authentication — do not include confidential content.
- Keep `staticwebapp.config.json` when replacing content (without the header it won't display).
