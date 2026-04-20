# Publishing

Maintainer-only doc. Covers the recurring release loop. One-time account setup (Azure DevOps PAT, Eclipse Contributor Agreement, Open VSX namespace) is documented in the project plan and the official registry docs linked below.

## Prerequisites (set up once)

- `VSCE_PAT` — Azure DevOps Personal Access Token with **Marketplace (Manage)** scope, organization set to **All accessible organizations**. Created at <https://dev.azure.com>. Publisher `utilitybend` must exist at <https://marketplace.visualstudio.com/manage>.
- `OVSX_PAT` — Open VSX access token from <https://open-vsx.org/user-settings/tokens>. Namespace `utilitybend` must exist; an Eclipse Contributor Agreement must be signed.
- Both secrets stored in GitHub repo -> Settings -> Secrets and variables -> Actions.

## Recurring release loop

```bash
# 1. Update CHANGELOG.md with the new version section.
# 2. Bump version (creates a git tag vX.Y.Z).
npm version patch   # or minor / major

# 3. Push commit + tag.
git push && git push --tags
```

GitHub Actions (`.github/workflows/publish.yml`) then:

1. Packages `utilitybend-neon.vsix`.
2. Publishes to the VS Code Marketplace via `@vscode/vsce`.
3. Publishes to Open VSX via `ovsx`.
4. Creates a GitHub Release with the `.vsix` attached and auto-generated notes.

Propagation: Marketplace ~1-5 min, Open VSX ~5-15 min.

## Fallback: manual publish (CI down)

```bash
npx --yes @vscode/vsce package --out utilitybend-neon.vsix
npx --yes @vscode/vsce publish --packagePath utilitybend-neon.vsix --pat "$VSCE_PAT"
npx --yes ovsx publish utilitybend-neon.vsix --pat "$OVSX_PAT"
```

## Local smoke-test before tagging

```bash
npx --yes @vscode/vsce package
cursor --install-extension utilitybend-neon-*.vsix
# or: code --install-extension utilitybend-neon-*.vsix
```

Then Cmd+K Cmd+T -> **Utilitybend Neon** and eyeball a TSX + CSS + JSON + Markdown file.

## Gotchas

- A published version cannot be reused. If a publish fails, bump the patch version and retry.
- README images must use absolute `https://raw.githubusercontent.com/...` URLs — the Marketplace strips relative image paths.
- PAT scoped to a single Azure DevOps org instead of "All accessible organizations" is the most common `vsce publish` failure.
