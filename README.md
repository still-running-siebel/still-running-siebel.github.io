# stillrunningsiebel.com

Hugo static site for Still Running Siebel, deployed to GitHub Pages via Actions.

## Local development

    hugo server        # live preview at localhost:1313
    hugo --minify      # production build into ./public

Hugo extended v0.148+ required. Content lives in `content/`, templates in `layouts/`, styles in `assets/css/main.css`.

## Publishing

Push to `main`. The workflow in `.github/workflows/deploy.yml` builds and deploys automatically. In the repo settings, set **Pages → Source → GitHub Actions** once.
