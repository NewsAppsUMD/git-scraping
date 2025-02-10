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
4. Here's some sample text to put in that file (copy all of it):

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
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
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

Ok, so this should now run every time we push to the repository. What if we wanted it to run every day? For that, we can change that `on:` section at the top of our YAML file, which controls when the workflow is run. Linux systems like what we're using in codespaces and for this Action use a utility called `cron` that [handles scheduled jobs](https://en.wikipedia.org/wiki/Cron). It has a specific syntax, too, but you can generate the output for it using [web-based generators like this one](https://crontab-generator.org/).

At the top of your YAML script, replace the `on:` section with this:

```
on:
  push:
    push:
  schedule:
    - cron: "0 12 * * *"
  workflow_dispatch:
```

That will run the workflow once a day at 12 p.m. UTC (8 a.m. EDT). The action will also run every time you push to the repository, which is a good way to test it out.
