# Publishing updates for this Mochy fork

The app keeps the original Android application ID (`com.mochyy.mochymusic`) and checks this repository for releases:

- Repository: https://github.com/arzanmhd819-tech/Mochy-Music
- Stable release API: `https://api.github.com/repos/arzanmhd819-tech/Mochy-Music/releases`
- Required APK asset names: `mochy.apk` (legacy in-app update screen) and `mochy-universal.apk` (queue/update channel updater)

## One-time signing setup

The signing key must be created and backed up by the repository owner. Do not commit it, upload it to a public issue, or share it in chat. The same key must be used for every release from this fork.

On a trusted computer with Android `keytool` installed:

```bash
keytool -genkeypair -v \
  -keystore release.keystore \
  -alias mochyfork \
  -keyalg RSA -keysize 2048 -validity 10000
```

Keep `release.keystore`, its store password, alias, and key password in a secure password manager and at least one encrypted backup. If the key is lost, future APKs cannot update installs signed with it.

Convert the keystore to Base64 locally:

```bash
# Linux
base64 -w 0 release.keystore > release-keystore.base64

# macOS
base64 < release.keystore | tr -d '\n' > release-keystore.base64
```

In GitHub, open **Settings → Secrets and variables → Actions → New repository secret** and create:

- `RELEASE_KEYSTORE_BASE64` — contents of `release-keystore.base64`
- `STORE_PASSWORD` — keystore password
- `KEY_ALIAS` — `mochyfork` (or the alias you chose)
- `KEY_PASSWORD` — key password

Never commit the keystore or the Base64 file. The workflow writes the key to the ignored `app/keystore/` directory only on the GitHub runner.

## Publishing a release

1. Commit and push the source changes to `main`.
2. Increase the semantic version for each release. The first fork release should be `v8.3.3` (newer than the upstream `8.3.2` source version); use `v8.3.4`, `v8.3.5`, etc. for later releases.
3. Create and push a tag, for example:

   ```bash
   git tag v8.3.3
   git push origin v8.3.3
   ```

4. GitHub Actions builds the signed universal FOSS APK and creates a GitHub Release containing both updater-compatible APK asset names.
5. Existing fork installs check the fork's GitHub Releases and offer the update through Mochy's in-app updater.

The workflow calculates a monotonically increasing Android `versionCode` starting above the source's current value of 73. The Android `versionName` is taken from the tag without the leading `v`.

## Replacing the official app the first time

The official Mochy app is signed by its publisher's private key. Your fork cannot reuse that private key. To switch to this fork while keeping the same Android package ID:

1. Back up/export anything important from official Mochy.
2. Uninstall the official Mochy app.
3. Install the first APK from this fork's GitHub Release.
4. Future releases from this fork can install over it, provided the signing key above remains unchanged.

The application ID remains `com.mochyy.mochymusic`; it is not a separate parallel app.

## Scope and verification

- Stable GitHub Release updates are configured.
- The workflow publishes `mochy.apk` for the existing update popup and `mochy-universal.apk` for the newer updater.
- Nightly builds are not configured by this workflow. The beta/nightly updater path will only work after a separate nightly workflow and matching nightly artifact are added.
- Android Gradle build and real-device update installation must be tested after the signing secrets are added.
