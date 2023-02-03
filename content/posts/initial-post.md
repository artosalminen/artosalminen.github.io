---
title: "How to set up a blog with Hugo and GitHub Pages in 10 minutes"
date: 2023-02-03T15:04:11+02:00
draft: false
---

So you want to set up a blog. Here is how to do it in 10 minutes or less. For me, it took a little more time, just to make sure for you it would not. And actually, even for me, most of the time was used to write this post.

So here we go, you have 40 for each step.

0. Create GitHub account if you do not have one yet
1. Create new GitHub repository named `<your GitHub username>.github.io`
2. On your local development environment, create a folder to work in, run `npm install hugo-bin --save-dev` to install Hugo CLI
3. Clone your GitHub repository under the same directory `git clone https://github.com/<your username>/<your username>.github.io.git`
4. Run `npx hugo new site <your username>.github.io`
5. Navigate to the repository root directory `cd <your username>.github.io`
6. Add a theme as Git Submodule: `git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod`
7. Update `config.toml` by adding the `theme = 'PaperMod'` option (and add a name for the site while at it)
8. Also update `config.toml` with your own baseUrl, e.g. `baseURL = 'https://<your GitHub username>.github.io/'`
9. Run `npx hugo new posts/initial-post.md`
10. Edit the initial post and when ready to publish, remove the draft tag
11. Add the following into `.github\workflows\gh-pages.yml` file in the repository
    ```
    name: github pages

    on:
    push:
        branches:
        - main  # Set a branch that will trigger a deployment
    pull_request:

    jobs:
        deploy:
            runs-on: ubuntu-22.04
            steps:
            - uses: actions/checkout@v3
                with:
                submodules: true  # Fetch Hugo themes (true OR recursive)
                fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

            - name: Setup Hugo
                uses: peaceiris/actions-hugo@v2
                with:
                hugo-version: 'latest'
                # extended: true

            - name: Build
                run: hugo --minify

            - name: Deploy
                uses: peaceiris/actions-gh-pages@v3
                if: github.ref == 'refs/heads/main'
                with:
                github_token: ${{ secrets.GITHUB_TOKEN }}
                publish_dir: ./public
    ```
11. Create a branch named `gh-pages` in GitHub
12. Check which branch is used for GitHub pages publishing in your GitHub repository, set it to `gh-pages`
13. Set in **GitHub Settings > Actions > General**: Workflow permissions to `Read and write permissions`
14. Push to Git remote using the main branch
15. Make sure the deployment went smoothly, and enjoy the result in `https://<your GitHub username>.github.io`