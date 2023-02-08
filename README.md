# Git Scraping

[Based on Simon Willison's description](https://simonwillison.net/2020/Oct/9/git-scraping/) of the process of automating scraping or retrieval jobs using GitHub Actions, Git scraping includes a wide variety of actions. Check out the [Git scraping topic on GitHub](https://github.com/topics/git-scraping) to see some examples.

In this repository, we'll cover a couple of examples of doing this. First, let's read up on [GitHub Actions](https://docs.github.com/en/actions/learn-github-actions). Any GitHub repository can be used to automate retrieval or scraping jobs. To connect a repository to GitHub Actions you need a set of instructions in a YAML file (.yml) in a special directory in your repository. That directory is called .github/workflows/. [Here's an example](https://github.com/dwillis/wvu-projects/blob/master/.github/workflows/scrape.yaml) of a project that runs three different scrapers and also assembles a Datasette app from one of them, deploying it to fly.io.

YAML has some very specific rules for a valid file, and GitHub Actions will be cranky about enforcing both those and a specific terminology for the steps in a workflow file. You can mostly think of the commands you use as bash shell commands like we've run before, surrounded by some information that tells GitHub what's about to happen. So you can run Python scripts (or R scripts), bash commands and other types of commands. You can install libraries, even use specific operating systems.

Writing these workflow YAML files is ... kind of a chore. Thus, I actually recommend that you use both existing examples and AI tools like ChatGPT to do so. Do NOT write these from scratch unless you've done it many, many times.

Using the repository you created for the first Web scraper assignment, open it up in codespaces and follow along:

### First Steps

1. A .github directory should already exist, so cd into it: `cd .github`.
2. Create a worflows directory: `mkdir workflows` and `cd` into that.
3. Create a blank YAML file in that directory called scrape.yaml: `touch scrape.yaml`
4. Here's some sample text to put in that file (copy in between the backticks):

```
name: Scrape

on:
  push:
    branches:
      - '*'

jobs:
  scheduled:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install dependencies
        run: pip install requests bs4

      - name: run scraper
        run: python scrapers/legislature/scrape.py

      - name: "Commit and push if it changed"
        run: |-
            git config user.name "Automated"
            git config user.email "actions@users.noreply.github.com"
            git add -A
            timestamp=$(date -u)
            git commit -m "Latest data: ${timestamp}" || exit 0
            git push
```

You might need to change the path to your scraper if you didn't follow the instructions in the tutorial exactly.

5. Save that file.
6. Get back to your main directory: `cd ..` twice.
7. Commit your changes and sync them with your GitHub repository.
8. Go to your repository on GitHub and click on the Actions tab, then click on the latest job.
