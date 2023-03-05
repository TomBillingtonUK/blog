---
layout: post
title: Automated Dependency Upgrades with Dependabot
subtitle : Dependency management can be a massive time sink for teams, and most fail to keep them updated. Automation can help us improve this, and Dependabot is an easy tool to use to get started.
image: assets/img/posts/2023-02-26-automated-dependency-upgrades-with-dependabot/social-media-post.png
imageWidth: 1080
imageHeight: 1080
tags: [DevOps, Time Savers, GitHub Actions]
author: Thomas Billington
comments : False
---
When building applications, we want to focus on providing value to our customers and users. Managing dependencies isn't sexy, and as I've seen over the last few weeks, it's easy to fall behind. In this blog, I look at how I added Dependabot to attempt to manage upgrades automatically in one of my projects.
<br/>
<br/>
## The initial state
I'm starting with an in-progress node project, which already has:
<br/>
<br/>
* A protected "main" branch
* A pipeline which builds, lints and runs tests on a pull request before merging into the "main" branch
* A release pipeline which deploys the application into Azure on any commit into the "main" branch

<br/>
This project has a single maintainer myself and has been active for over two years. So there is going to be lots of stuff that needs updating.
<br/>
<br/>
## Introducing Dependabot
Anyone who has worked on GitHub over the last few years will be aware of dependabot. It started as a bot which would let you know when your packages have a security vulnerability. These days it can also update all out-of-date packages.
<br/>
<br/>
You first need to make sure Dependabot is enabled on your repository. You can do this by visiting the Code security and analysis settings and turning on the bot by clicking the correct buttons. The page should look something like this:
<br/>
<br/>
![GitHub Code security and analysis settings page](\assets\img\posts\2023-02-26-automated-dependency-upgrades-with-dependabot\code-security-and-analysis-settings.png)
<br/>
<br/>
To get dependabot to update packages, you must create a dependabot.yml file in the .github folder, like the one below:
<br/>
<br/>
```yml
version: 2
updates:
  - package-ecosystem: 'npm'
    directory: '/'
    schedule:
      interval: 'daily'
      time: '06:00'
      timezone: 'Europe/London'
    rebase-strategy: 'auto'
    target-branch: 'main'
    commit-message:
      prefix: 'chore'
      prefix-development: 'chore dev:'
      include: 'scope'
    open-pull-requests-limit: 3
```
<br/>
<br/>
As you can see, I've told it at 6:00 am every day to look for out-of-date dependencies and allowed it to create up to 3 pull requests with updates. I went to sleep and woke up to see:
<br/>
<br/>
![Dependabot pull request list](\assets\img\posts\2023-02-26-automated-dependency-upgrades-with-dependabot\list-of-pull-requests.png)
<br/>
<br/>
I had three pull requests ready to approve, one was successfully built, and two had failed. So my checks had worked; I triggered auto-merge on the successful one, and the pipelines did the rest. For the ones that failed, I was able to address the breaking changes and handle them manually.
<br/>
<br/>
## Automating the merges
I'm not too fond of having multiple pull requests every morning to merge manually. To fix this, I created a GitHub action to set the pull request to auto-merge.
<br/>
<br/>
{% raw %}
```yml
name: Dependabot auto-merge
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: ${{ github.actor == 'dependabot[bot]' }}
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Enable auto-merge for Dependabot PRs
        run: gh pr merge --auto --rebase "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```
{% endraw %}
<br/>
<br/>
With the action set up and merged, every time the pull request is updated, the action will check to see if it's Dependabot, and if it is, set the pull request to merge automatically.
<br/>
<br/>
![Dependabot pull request list](\assets\img\posts\2023-02-26-automated-dependency-upgrades-with-dependabot\automatic-merge-pending.png)
<br/>
<br/>
Once all the checks have passed, the pull request automatically merges, and we have an automated dependency upgrade system.
<br/>
<br/>
![Dependabot pull request list](\assets\img\posts\2023-02-26-automated-dependency-upgrades-with-dependabot\automatic-merge-complete.png)
<br/>
<br/>
## Conclusions

Based on my experiences over the last few weeks, I think all projects would benefit from an automatic dependency upgrade system. My perfect system would:
<br/>
<br/>
* Automatically deal with all non-breaking updates
* Tell me when there is an upgrade which breaks something
* Keep me informed about deprecated packages.

<br/>
Dependabot will do two out of three, so it's a good starting point. After spending a few hours working on this, my current thoughts are:
<br/>
<br/>
* A single pull request per dependency caused issues with jest, lint and typescript, where multiple packages must be updated simultaneously.
* You must have robust automated checks already in place if you are going all the way to production automatically. After one deployment, my production environment broke, so I needed to fix that gap in the automation.
* With all the deployments, I hit the GitHub Actions 2GB storage limit,   so you need to ensure your builds efficiently use the available resources.

<br/>
I'll watch how dependabot performs over the next few weeks to see if I will stick or try another tool.