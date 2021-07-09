# PWA始めの一歩(create-react-app)


## cra-template-pwa-typescriptテンプレートでreactのひな形を作成する

```bash
npx create-react-app pwa-proto --template cra-template-pwa-typescript
```

`unregister()` ⇒ `register()`に変更（Service workerを利用する)
```typescript
// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://cra.link/PWA
// serviceWorkerRegistration.unregister();
serviceWorkerRegistration.register();
```

## github pagesへデプロイするためのymlファイルを作成する

* package.jsonにhomepageのURL(github pages)を追加する 

```json
  "homepage": "https://murasuke.github.io/pwa-proto/",
```

* deploy用のキー生成

```bash
$ ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f pwa-proto -N ""
```

* githubのSettingから`Deploy keys`を選択し、公開鍵を登録する

* githubのSettiongから`Secrets`を選択し、秘密鍵を登録する（名前は`MY_ACTIONS_DEPLOY_KEY`)

* github actionsでpagesへデプロイするためのymlファイルを作成する(./.github/workflows/build.yml) 

```yaml
# ワークフローの名前
name: Release-GitHub-Page

# 起動のタイミング 
# 今回はmasterブランチへのpush
on:
  push:
    branches:
      - master

# ジョブの定義
jobs:
  build:
    # 実行するインスタンス
    runs-on: ubuntu-latest

    # nodeのバージョン一覧
    strategy:
      matrix:
        node-version: ['16.x']

    # 各ステップの実行
    steps:

    # チェックアウト
    - uses: actions/checkout@v1

    # 使用するnodeのバージョンを指定
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}

    # パッケージのインストールとアプリのビルド
    - name: install and build
      run: |
        npm ci
        npm run build
      env:
        CI: true

    # gh-pagesを使って公開
    - name: deploy
      uses: peaceiris/actions-gh-pages@v2
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.MY_ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: gh-pages
        PUBLISH_DIR: ./build

```


## githubへpush する

* pushするとactionsがバックグラウンドでビルドしてpagesheへデプロイする
