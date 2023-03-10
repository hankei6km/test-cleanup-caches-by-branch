# test-cleanup-caches-by-branch

[Force deleting cache entries](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#force-deleting-cache-entries) のサンプルのテスト。

2023-01-21 時点のサンプルは下記のようなコードになっている。

```yaml
name: cleanup caches by a branch
on:
  pull_request:
    types:
      - closed
  workflow_dispatch:

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Cleanup
        run: |
          gh extension install actions/gh-actions-cache

          REPO=${{ github.repository }}
          BRANCH=${{ github.ref }}

          echo "Fetching list of cache key"
          cacheKeysForPR=$(gh actions-cache list -R $REPO -B $BRANCH | cut -f 1 )

          ## Setting this to not fail the workflow while deleting cache keys. 
          set +e
          echo "Deleting caches..."
          for cacheKey in $cacheKeysForPR
          do
              gh actions-cache delete $cacheKey -R $REPO -B $BRANCH --confirm
          done
          echo "Done"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

しかし、`BRANCH=${{ github.ref }}` だと大体はデフォルトブランチになる。これに関しての issue も立っている。

https://github.com/github/docs/issues/22727

このリポジトリで試した場合だと下記のことがわかった。

- `actions: write` の permission が必要
- ~~`$BRANCH` は `github.head_ref` を使うとプルリクエストのブランチのキャッシュ消せた~~
- `$BRANCH` の指定はキャッシュを作ったときの event によって変化する
  - `pull_request` の場合は[上記 issue](https://github.com/github/docs/issues/22727) の通り
  - `push` の場合は `github.head_ref`
  - 他の event や type だとまた違うかもしれない

試したときのコードは `clean.yml` に記述してある。

https://github.com/hankei6km/test-cleanup-caches-by-branch/blob/main/.github/workflows/cleanup.yml
