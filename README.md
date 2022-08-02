# data-analysis
データ分析実施時に必要になる前処理、モデリングやモデル評価などの知見についてまとめていきます。  
# ドキュメント
[./doc](https://github.com/kato-masaya/data-analysis/tree/main/doc)配下に各ドキュメント配置しています。

# Inner PROXY Settings
## ✒Writing environment 
- [.devcontainer/devcontainer.json](https://github.com/kato-masaya/data-analysis/tree/main/.devcontainer/devcontainer.json)
  - **"TODO: "** 以下に利用している Proxy 設定を入力

```bash
"build": {
  "args": {
    // TODO: Proxy設定。利用者の環境に合わせて定義すること
    "PROXY": "http://xxxxxxxxxxxxxxx:yyyy"
  }
},
```

- [.devcontainer/Documentation_Plugin_Dockerfile](https://github.com/kato-masaya/data-analysis/tree/main/.devcontainer/Documentation_Plugin_Dockerfile)
  - **"# Proxy"** 以下のコメントアウト解除

```bash
# Proxy
# ARG PROXY=${PROXY}
# ENV http_proxy=${PROXY}
# ENV https_proxy=${PROXY}
# ENV HTTP_PROXY=${PROXY}
# ENV HTTPS_PROXY=${PROXY}

```

