# reinGraph 文档与官网

[reinGraph](https://github.com/MaLunan/rein-graph)(图编排框架)的文档与官网源。

- `docs/` —— 内部开发讲解(`code-guide.md`:逐模块大白话,讲清复用了 rein 的哪条机制)
- `website/` —— 官网与文档站(MkDocs Material)

## 本地预览

```bash
pip install mkdocs-material
mkdocs serve   # → http://127.0.0.1:8000
```

## 部署

push 到 main 后,GitHub Actions(`.github/workflows/docs.yml`)自动 `gh-deploy` 到 GitHub Pages。
