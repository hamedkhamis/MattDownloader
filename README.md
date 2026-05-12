# MattDownloader GitHub Relay

This repository is a GitHub Actions template for **MattDownloader**. It downloads files on GitHub runners, stores the result in your fork under `jobs/<job_id>/`, and lets the Android app fetch the prepared file later.

The idea is inspired by [soheilditf5-svg/downloader](https://github.com/soheilditf5-svg/downloader): use GitHub Actions as a stable relay when direct phone downloads are slow, blocked, or unreliable.

## What this repo contains

- `.github/workflows/` relay workflows.
- `README.md` and `.gitignore`.
- No mobile app source code.
- No APK in the git tree. Download the MattDownloader APK from this repository's **Releases** page.

Actions create `jobs/<job_id>/` on your fork after successful downloads. That folder contains the manifest and downloaded file data that the app reads.

## How to use

> **Use a separate GitHub account for this**, not your main one. Forking, workflow runs, and downloaded job files all live under whichever account you use. A throwaway or secondary account keeps your primary account, repos, and reputation isolated from any content the relay pulls.

1. **Fork this repository.** Sign in with your secondary account, then click **Fork** and create your own copy. Your fork can be public or private.
2. **Copy your fork repo URL.** Use the repository you forked, not the original template. It will look like `https://github.com/yourname/your-fork` or `yourname/your-fork`.
3. **Enable Actions on your fork.** Open your fork's **Actions** tab and allow workflows if GitHub asks.
4. **Install MattDownloader.** Download the latest APK from this repository's **Releases** page, install it, and open the app.
5. **Create a fine-grained GitHub token.** Open [github.com/settings/personal-access-tokens](https://github.com/settings/personal-access-tokens) and click **Generate new token**. Configure it as follows:
   - **Token name**: anything you like (for example `mattdownloader-relay`).
   - **Expiration**: pick whatever you prefer; you can regenerate later.
   - **Resource owner**: the account that owns your fork.
   - **Repository access**: choose **Only select repositories** and pick your fork.
   - **Repository permissions**:
     - **Contents**: **Read-only** (the app reads `manifest.json` and prepared file blobs from your fork; the workflow itself commits using GitHub's built-in `GITHUB_TOKEN`).
     - **Actions**: **Read and write** (so the app can dispatch workflows and check run status).
     - **Metadata**: **Read-only** (auto-selected by GitHub).
   - Click **Generate token** and copy the value once. You will not see it again.
6. **Add the token and fork URL to the app.** In MattDownloader settings, paste the token and enter your fork as `owner/repo` (or the full URL). Save.
7. **Start downloading.** Give MattDownloader a file URL. The app dispatches the workflow on your fork, waits for `manifest.json`, then downloads the prepared file from GitHub.

If setup fails, check that Actions are enabled, the token is valid, and the repo URL points to your fork instead of the upstream template.

## Why use it

- GitHub Actions handles the heavy HTTP download instead of your phone.
- Files live in your own fork, under your GitHub account.
- Large files are split into chunks of about 95 MB so the GitHub API remains practical.
- Jobs can be deleted from the app when you are finished.

## Workflows

- **`download-split.yml`**: accepts `file_url` and `job_id`, downloads the file, writes `jobs/<job_id>/manifest.json`, splits large files, and commits the result.
- **`delete-job.yml`**: accepts `job_id`, removes `jobs/<job_id>/`, and commits the cleanup.
- **`cleanup-expired-jobs.yml`**: scheduled cleanup for old `jobs/*` folders.

## App contract

After `download-split.yml` succeeds, the app reads `jobs/<job_id>/manifest.json`.

- `suggested_filename`: safe filename for saving on device.
- `mode: "single"`: fetch `jobs/<job_id>/file.bin`.
- `mode: "chunked"`: fetch `jobs/<job_id>/chunks/*` in order and merge on device.

`job_id` must be 6-64 characters and use only `A-Z`, `a-z`, `0-9`, `_`, or `-`.

## Security note

Use a fine-grained personal access token limited to only your fork. Avoid classic tokens with broad account access unless you understand the risk.

## Donate

If this template or MattDownloader helps you, optional donations are welcome. Always verify the chain and address in your wallet.

| Asset | Network | Address |
|-------|---------|---------|
| **USDT** | Polygon | `0xacef000D72eb46Af92F32252fe2bCE146aa9c2E0` |
| **USDT** | TRON (TRC20) | `TFpeKzEakmSXXi7MXBt1VZVfErB3kXHyL7` |

## License

Released under the [MIT License](./LICENSE).
