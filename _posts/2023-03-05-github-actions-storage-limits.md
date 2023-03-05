---
layout: post
title: Avoiding GitHub Actions Artifact Storage Quota
subtitle : As a GitHub pro member, you get 2GB of artifact storage. I hit those limits and this is what I put in place to avoid the same issue going forward.
image: assets/img/posts/2023-03-05-github-actions-storage-limits/social-media-post.png
imageWidth: 1080
imageHeight: 1080
tags: [GitHub Actions, DevOps]
author: Thomas Billington
comments : False
---
When writing and testing my [last blog](/2023/02/26/automated-dependency-upgrades-with-dependabot.html), I hit the following error on my GitHub Pro Account. 
<br/>
<br/>
> Create Artifact Container failed: Artifact storage quota has been hit. Unable to upload any new artifacts.

<br/>
I checked my billing account, and I was over my free 2GB limit:
<br/>
<br/>
![Image of GitHub storage limit](/assets/img/posts/2023-03-05-github-actions-storage-limits/storage-limit.png)
<br/>
<br/>
There is no way I'm paying our Microsoft any more than I should, so it was time to work out a way around this and ensure I don't hit the limits again.
<br/>
<br/>
## Clearing out existing artifacts

The first job was to clear out all existing artifacts. To do this, I created a new GitHub Action to delete all the artifacts I had created. There are already multiple actions in the marketplace that other people have made, so I opted to use one. This was my yaml file:
<br/>
<br/>
```yml
name: Delete all build artifacts
on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Artifact Cleanup
        uses: jimschubert/delete-artifacts-action@v1
        with:
          log_level: 'error'
          min_bytes: '0'
```
<br/>
<br/>
The job ran successfully; however, it took around a day for GitHub to recognise that I was no longer at the storage limits.
<br/>
<br/>
![Image of GitHub storage limit](/assets/img/posts/2023-03-05-github-actions-storage-limits/storage-limit-after-deleting.png)
<br/>
<br/>
## Deleting after every job

That's the initial problem solved, but I don't want to be in this situation again. I could run the delete job on a CRON schedule, but that wouldn't be enough. I needed my deployment job to clean up after itself. 
<br/>
<br/>
The first step is creating an access token for the GitHub Action, enabling it to view and delete artifacts. To do this:
<br/>
<br/>
1. Go to GitHub Settings -> Developer Settings -> Personal Access Tokens -> Fine Grained Tokens
2. Click Generate New Token
3. Give the token a name, description and expiry date
4. Limit the token to the repository that is creating the artifacts you want to delete
5. Give the token Read-write access to Actions on the repository 
6. Click Generate Token and copy the toke.

<br/>
After creating the token, we must store it as a secret in the repository so that the action can use it. To do this:
<br/>
<br/>
1. Navigate to the repository 
2. Go to Settings -> Secrets and variables -> Actions 
3. Click on "new repository secret."
4. Enter a name; we called ours DELETE_ARTIFACT_TOKEN
5. Paste in your personal access token
6. Click "Add secret."

<br/>
We now have access sorted. All that is left is to update my deployment pipeline action with an extra stage for cleaning up the artifacts. My full pipeline can be seen below:
<br/>
<br/>
```yml
name: Deploy
on:
  push:
    branches: ['main']
  workflow_dispatch:

env:
  AZURE_WEBAPP_NAME: test
  AZURE_WEBAPP_PACKAGE_PATH: '.'
  NODE_VERSION: '16.x'

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'yarn'

      - name: Install dependencies
        run: yarn install

      - name: Build package
        run: yarn run build

      - name: Run lint
        run: yarn run lint

      - name: Run tests
        run: yarn run test

      - name: Remove files
        run: |
          rm -r .husky
          rm -r assets
          rm -r src
          rm -r tests
          rm jest.config.js
          rm readme.md
      - name: Zip artifact for deployment
        run: zip release.zip ./* -r

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: node-app
          path: release.zip

  deploy:
    permissions:
      contents: none
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'Production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: unzip artifact for deployment
        run: unzip release.zip

      - name: 'Deploy to Azure WebApp'
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
    
  cleanup:
    runs-on: ubuntu-latest
    continue-on-error: false
    needs: deploy
    steps:
      - uses: actions/checkout@v1
      - name: Artifact Cleanup
        uses: jimschubert/delete-artifacts-action@v1
        with:
          GITHUB_TOKEN: ${{ secrets.DELETE_ARTIFACTS_TOKEN }}
          artifact_name: 'node-app'
          log_level: 'error'
          min_bytes: '0'
```
<br/>
<br/>
I ran the job, and it succeeded, meaning I no longer need to worry about all those artifacts causing me to hit the storage limits.
<br/>
<br/>
![Image showing stages in GitHub Action](/assets/img/posts/2023-03-05-github-actions-storage-limits/successful-pipeline-run.png)
<br/>
![Image showing stages logs of cleanup job](/assets/img/posts/2023-03-05-github-actions-storage-limits/cleanup-job-logs.png)
<br/>
<br/>
If you are facing the same issue, I hope this blog gives you some inspiration on how to get around this issue. There are many potential solutions to this problem; however, this was good enough for my use case for now. Thank you for reading.
<br/>
<br/>
Thomas Billington