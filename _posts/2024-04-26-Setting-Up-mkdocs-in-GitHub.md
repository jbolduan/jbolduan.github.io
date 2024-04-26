---
title: Setting Up mkdocs in GitHub
date: 2024-04-26 07:20:22 -0500
Categories: GitHub mkdocs python
---

1. Install Python 3

    ```bash
    sudo apt-get install python3 python3-venv python3-dev python3-pip
    ```

2. Install mkdocs

    ```bash
    pip install mkdocs
    pip install mkdocs-material
    ```

    If there is a warning about the `mkdocs` command not being found, add the following to your `$PATH` variable:

    ```bash
    PATH="$PATH:{path to the directory where the mkdocs command is located}"
    ```

3. Create a new mkdocs project

    ```bash
    mkdocs new my-project
    ```

4. Change the theme to `material` in the `mkdocs.yml` file

    ```yaml
    theme:
        name: material
    ```

5. Test the mkdocs site locally to make sure it builds correctly

    ```bash
    cd my-project
    mkdocs serve
    ```

    Open a browser and navigate to `http://127.0.0.1:8000/`

6. Create the GitHub workflow file

    Create a new directory called `.github/workflows` in the root of your project and create a new file called `mkdocs.yml` with the following content:

    ```yaml
    name: Deploy Docs

    on:
    push:
        branches:
        - main

    workflow_dispatch:

    permissions:
    contents: read
    pages: write
    id-token: write

    concurrency:
    group: "pages"
    cancel-in-progress: false

    jobs:

    # build job

    build:
        runs-on: ubuntu-latest
        steps:
        - name: Checkout
            uses: actions/checkout@v4

        - name: Setup Pages
            uses: actions/configure-pages@v5

        - name: Setup Python
            uses: actions/setup-python@v5.1.0
            with:
            python-version: '3.x'
        
        - name: Install reqired packages
            run: pip install -r requirements.txt
        
        - name: Setup caching
            uses: actions/cache@v4
            with:
            key: ${{ github.sha }}
            path: .cache
        
        - name: Build site (_site directory name is used for Jekyll compatiblity)
            run: mkdocs build --config-file ./mkdocs.yml --site-dir ./_site
        
        - name: Upload artifact
            uses: actions/upload-pages-artifact@v3

    # deployment job

    deploy:
        environment:
        name: github-pages
        url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        needs: build
        steps:
        - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4
    ```

7. Create a requirements.txt file

    Create a new file called `requirements.txt` in the root of your project with the following content:

    ```txt
    mkdocs-material[recommended, imaging]
    ```

8. Go to **Settings** > **Pages** in your GitHub repository and set the source to GitHub Actions.
9. Commit and push all the repo changes into the main branch.
10. Go to the GitHub Actions tab in your repository and check the status of the workflow.  It should complete successfully and you should now have a GitHub pages built on mkdocs! 🥳
