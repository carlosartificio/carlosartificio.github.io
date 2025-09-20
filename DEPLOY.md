# Deployment and CI notes

This repository uses a secure GitHub Actions workflow to build and deploy the Jekyll site to GitHub Pages.

## How it works
- On push to `master`, GitHub Actions builds the site using the Ruby/Bundler environment in `Gemfile`.
- The built `_site` directory is uploaded as a Pages artifact with `actions/upload-pages-artifact`.
- The artifact is deployed to GitHub Pages using `actions/deploy-pages`, which requires only the `pages: write` permission.

This avoids pushing compiled HTML to a branch (like `gh-pages`) and minimizes token permissions.

## Why this is secure
- Least-privilege permissions: the workflow grants `pages: write` rather than `contents: write`.
- No permanent deploy token or personal access token is stored in the repo.
- Builds run in GitHub-hosted runners with transient credentials.

## Adding third-party secrets (example: Algolia)
If you enable `jekyll-algolia` indexing or other plugins that require API keys, add secrets to the repository.

1. Go to your repository on GitHub → Settings → Secrets and variables → Actions → New repository secret.
2. Add the secrets used by your plugin (example names):
   - ALGOLIA_APP_ID
   - ALGOLIA_ADMIN_KEY
   - ALGOLIA_INDEX_NAME

3. Update your workflow or add a separate indexing job that uses these secrets as environment variables.

Example step snippet to run indexing in CI:

```yaml
- name: Index to Algolia
  env:
    ALGOLIA_APP_ID: ${{ secrets.ALGOLIA_APP_ID }}
    ALGOLIA_ADMIN_KEY: ${{ secrets.ALGOLIA_ADMIN_KEY }}
  run: bundle exec jekyll algolia
```

## Local testing
To test locally (Windows PowerShell):

```powershell
gem install bundler
bundle install
bundle exec jekyll serve
```

Visit http://127.0.0.1:4000 to preview.

## Notes about minimal-mistakes
The theme supports a `_includes/head_custom.html` include which is automatically merged into the site head. This repo adds `_includes/head_custom.html` to inject `{% seo %}` provided by `jekyll-seo-tag`.

If you want me to add more CI steps (image optimization, automated accessibility checks, or Algolia indexing), say which features you'd like and I'll add them.
