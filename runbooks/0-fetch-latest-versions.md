# Runbook: Fetch latest versions and SHAs for the marketplace

Refresh the per-skill `version` in `catalog.json` and the per-plugin `sha` in
`.claude-plugin/marketplace.json` so they point at each skill's **latest GitHub
release**.

## Goal / contract

For every skill listed in the marketplace:

- `catalog.json` → `skills[].version` = the latest release's version (tag minus
  any prefix, e.g. `v1.2.0` → `1.2.0`).
- `.claude-plugin/marketplace.json` → `plugins[].source.sha` = the **commit
  SHA** that the latest release tag points to.

The SHA must be a commit SHA, not the annotated-tag object SHA — Claude clones
the repo and checks out that commit.

## Critical: use GitHub, not local clones

Do **not** read versions/SHAs from local git checkouts (`git rev-parse`,
`version.txt`, `.release-please-manifest.json`, local tags). Local clones are
frequently stale or ahead of what is actually released, which produces wrong
values. Always query GitHub's release API as the source of truth.

## Steps

### 1. List the repos to refresh

The set of repos comes from `.claude-plugin/marketplace.json` → each
`plugins[].source.url` (strip the trailing `.git`). They are all under the
`shivdeepak` owner. Today that is: `speakeasy`, `self-documenting`,
`knowledge-base-builder`, `skillship`.

### 2. Fetch the latest release tag + commit SHA per repo

Preferred (if `gh` is installed and authenticated):

```bash
for repo in speakeasy self-documenting knowledge-base-builder skillship; do
  tag=$(gh release view -R "shivdeepak/$repo" --json tagName -q '.tagName')
  sha=$(gh api "repos/shivdeepak/$repo/commits/$tag" -q '.sha')
  echo "$repo  tag=$tag  sha=$sha"
done
```

Fallback with `curl` + the REST API (no auth needed for public repos; this is
what was used last time because `gh` was unavailable). It resolves annotated
tags down to the underlying commit:

```bash
for repo in speakeasy self-documenting knowledge-base-builder skillship; do
  tag=$(curl -fsSL "https://api.github.com/repos/shivdeepak/$repo/releases/latest" \
    | grep -m1 '"tag_name"' | sed -E 's/.*"tag_name": *"([^"]+)".*/\1/')
  ref=$(curl -fsSL "https://api.github.com/repos/shivdeepak/$repo/git/ref/tags/$tag")
  otype=$(echo "$ref" | grep -m1 '"type"' | sed -E 's/.*"type": *"([^"]+)".*/\1/')
  osha=$(echo  "$ref" | grep -m1 '"sha"'  | sed -E 's/.*"sha": *"([^"]+)".*/\1/')
  if [ "$otype" = "tag" ]; then
    sha=$(curl -fsSL "https://api.github.com/repos/shivdeepak/$repo/git/tags/$osha" \
      | grep -A3 '"object"' | grep -m1 '"sha"' | sed -E 's/.*"sha": *"([^"]+)".*/\1/')
  else
    sha=$osha
  fi
  echo "$repo  tag=$tag  sha=$sha"
done
```

Notes:

- Tag prefixes vary: some repos tag as `v1.2.0`, others as
  `<name>-v1.2.0` (e.g. `speakeasy-v1.1.0`, `skillship-v1.2.3`). Use the tag
  exactly as returned for the API lookup; strip everything up to and including
  the last `v` to get the catalog `version`.
- `releases/latest` reflects the latest **published** (non-draft,
  non-prerelease) release.

### 3. Derive the catalog version from the tag

`version` = the tag with any leading `<name>-` and the `v` removed.

| Tag | version |
| --- | --- |
| `v1.2.0` | `1.2.0` |
| `speakeasy-v1.1.0` | `1.1.0` |
| `skillship-v1.2.3` | `1.2.3` |

### 4. Apply the updates

- In `.claude-plugin/marketplace.json`, set each `plugins[].source.sha` to the
  commit SHA from step 2 (match by repo URL / plugin `name`).
- In `catalog.json`, set each `skills[].version` to the value from step 3
  (match by `name` / `repo`).

Only `version` and `sha` change — leave descriptions, categories, tags, etc.
untouched.

### 5. Validate

Confirm both files still parse as JSON:

```bash
node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/marketplace.json','utf8')); JSON.parse(require('fs').readFileSync('catalog.json','utf8')); console.log('both JSON valid')"
```

### 6. Report

Print a table of `skill | old → new version | new sha` so the change is easy to
review. Call out any skill whose unreleased local work is **not** yet reflected
in a GitHub release (it should keep pointing at the last released tag until a
new release is cut).
