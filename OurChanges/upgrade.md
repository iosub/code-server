# Upgrade code-server from source

This repository is a patched code-server fork. Installing a stock release will
replace the fork, so build and install a Debian package from this checkout.

The code-server version follows the embedded VS Code version. For example,
VS Code `1.133.2` should be packaged as code-server `4.133.2`.

## 1. Prepare the checkout

Start from the repository root and replace the example versions below:

```bash
cd /root/code-server
git switch miotest
git pull --ff-only origin miotest
git submodule update --init --recursive

export VSCODE_VERSION=1.133.2
export VERSION=4.133.2
export VSCODE_TARGET=linux-x64
```

Install the Linux build dependencies. The native `kerberos` and
`native-keymap` modules will fail to compile if `libkrb5-dev` or
`libxkbfile-dev` is missing.

```bash
sudo apt-get update
sudo apt-get install -y \
	build-essential pkg-config quilt rsync jq gettext-base curl \
	libx11-dev libx11-xcb-dev libxkbfile-dev libnotify-bin libkrb5-dev
```

Install the repository-pinned Node version. With `fnm`:

```bash
fnm install "$(cat .node-version)"
fnm use "$(cat .node-version)"
node --version
```

## 2. Update VS Code and patches

The update script checks out the requested VS Code tag, refreshes the quilt
patches, updates `.node-version`, regenerates CSP hashes, and adds a changelog
entry.

```bash
VERSION="$VSCODE_VERSION" ./ci/build/update-vscode.sh
```

Review `.cache/checklist`, resolve any rejected patches, and verify the
changelog. Confirm that every patch is applied before building:

```bash
quilt applied
quilt push -a
git status --short
git -C lib/vscode describe --tags --exact-match
```

Changes inside `lib/vscode` after `quilt push -a` are expected. Commit the
updated gitlink and files under `patches/` in the parent repository; do not
commit the patched VS Code working tree as a separate VS Code commit.

## 3. Install dependencies

VS Code's preinstall script must run before the root install so native modules
use the correct V8 headers.

```bash
cd lib/vscode/build
npm ci
cd ..
source ./build/azure-pipelines/linux/setup-env.sh
node build/npm/preinstall.ts
cd ../..
npm ci
```

If `npm ci` reports `gssapi/gssapi.h` missing, install `libkrb5-dev`. If it
reports `xkbfile.pc` missing, install `libxkbfile-dev` and `pkg-config`.

## 4. Build and package

Install the repository-pinned `nfpm` version if it is not already available:

```bash
mkdir -p "$HOME/.local/bin"
curl -sSfL \
	"https://github.com/goreleaser/nfpm/releases/download/v2.3.1/nfpm_2.3.1_$(uname -s)_$(uname -m).tar.gz" \
	| tar -C "$HOME/.local/bin" -xz nfpm
export PATH="$HOME/.local/bin:$PATH"
```

Build code-server and the minified VS Code payload, include native modules in
the release, and create the Debian package:

```bash
npm run build
VERSION="$VERSION" VSCODE_TARGET="$VSCODE_TARGET" npm run build:vscode
KEEP_MODULES=1 VSCODE_TARGET="$VSCODE_TARGET" npm run release
npm run package
```

Do not run `npm run clean` in a dirty checkout without reviewing it first. The
clean script runs `git clean -Xffd` and deletes ignored build artifacts.

## 5. Verify and install

Check the standalone release and package metadata before replacing the running
installation:

```bash
./release/bin/code-server --version
dpkg-deb --info "release-packages/code-server_${VERSION}_amd64.deb"
sudo dpkg -i "release-packages/code-server_${VERSION}_amd64.deb"

dpkg-query -W -f='${Package} ${Version}\n' code-server
/usr/bin/code-server --version
```

For ARM, use the package filename generated in `release-packages/` instead of
the `amd64` example.

## 6. Restart the service

On this server, root's instance is named `code-server-root.service`:

```bash
sudo systemctl restart code-server-root.service
sudo systemctl status code-server-root.service --no-pager
/usr/bin/code-server --version
```

The generic instance name is `code-server@$USER`. Find the active name when
needed with:

```bash
systemctl list-units 'code-server*' --all --no-pager
```

After reconnecting, hard-refresh the browser with `Ctrl+Shift+R` if About still
shows the previous version. Restarting without installing the new `.deb` only
restarts the old `/usr/bin/code-server` binary.

## 7. Commit and push

Review parent-repository changes before committing. A dirty `lib/vscode`
working tree is normal while quilt patches are applied.

```bash
git diff --check
git status --short
git add CHANGELOG.md .node-version lib/vscode patches
git commit -m "chore: update Code to ${VSCODE_VERSION}"
git push origin miotest
```

Only stage files that actually changed. Run the focused tests for any custom
source changes before committing them.
