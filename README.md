# So I Write

Personal blog at https://shelper.github.io/, built with Hugo 0.100.1.

## Write and publish

Add or edit Markdown in `content/posts/`, retaining its YAML front matter (title, date, tags, categories). Push to `main`; GitHub Actions builds the blog and publishes `public/` to `gh-pages`. Existing post URLs, RSS, taxonomy pages, and Disqus identifiers are retained.

## Preview locally

```sh
git submodule update --init --recursive
hugo server -D
```

The notebook design lives in `layouts/` and `static/notebook/`. The PaperMod submodule supplies existing supporting templates. Run `hugo --minify` with Hugo 0.100.1 to reproduce the deployment build.
