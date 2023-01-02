---
layout: post
title: Why the size of your package matters
subtitle : And you need check how big it is
tags: [Devops, Time Savers]
author: Thomas Billington
comments : False
---
I needed to deploy a node web application into Azure; as always, I like to keep costs low, so I used the free GitHub action minutes I have as a GitHub pro member. And I was in for a shock, with the first build taking 41 minutes and 30 seconds when using the default template for deploying a Web Application to Azure.
<br/>
<br/>

| Task | Time |
|--|--|
| Setup Job | 2 seconds  |	
| Run actions/checkout@v3 | 2 seconds  |	
| Set up Node.js | 2 seconds  |
| Yarn install, build and test | 1 minute 28 seconds |	
| Upload artifact for deployment  job | 39 minutes 38 seconds  |	
| Post set up Node.js | 2 seconds  |	
| Post Run actions/checkout@v3 | 0 seconds  |	
| Complete job | 0 seconds  |	

<br/>
There is no way I am going to wait 40 minutes for a build, so it was time to dive into what the default workflow was doing.
<br/>
<br/>
## Improving the upload job speed

GitHub Actions was uploading each file one by one, all 60,000 of them. The cynical side of me thought it was one way for GitHub to make money as you pay per minute.
<br/>
<br/>
 I could get around that by zipping the code and making the following changes to the pipeline:
<br/>
<br/>
 ```
- name: Zip artifact for deployment
    run: zip release.zip ./* -r

- name: Upload artifact for deployment job
    uses: actions/upload-artifact@v3
    with:
    name: node-app
    path: release.zip
 ```

 <br/>
 With this change, I now zip all the files together and then tell the upload artifact action to only upload the one zipped file. I also needed to update the release pipeline to unzip the file later. So what were the results:
 <br/>
 <br/>

| Task | Time |
|--|--|
| Setup Job | 2 seconds  |	
| Run actions/checkout@v3 | 5 seconds  |	
| Set up Node.js | 1 seconds  |
| Yarn install, build and test | 1 minutes 33 seconds |	
| Zip artifact | 31 seconds  |	
| Upload artifact for deployment  job | 1 minutes 32 seconds  |	
| Post set up Node.js | 12 seconds  |	
| Post Run actions/checkout@v3 | 0 seconds  |	
| Complete job | 0 seconds  |	

<br/> 
Total time: 4 minutes and 2 seconds, a 90.28% decrease.
<br/>
<br/>
## Reducing the size of the package

Now that I am only uploading one file, the next step is to ensure I am only uploading the files I need. I created a new task which will delete unnecessary files:
<br/>
<br/>
 ```     
 - name: Remove files
    run: |
    rm -r .husky
    rm -r assets
    rm -r src
    rm -r tests
    rm jest.config.js
    rm readme.md
 ```

<br/>
That leaves me with the built code, node modules and the package.json file deployed into Azure. The time savings are below:
<br/>
<br/>

| Task | Time |
|--|--|
| Setup Job | 1 seconds  |	
| Run actions/checkout@v3 | 2 seconds  |	
| Set up Node.js | 6 seconds  |
| Yarn install, build and test | 50 seconds |
| Delete files | 0 seconds  |		
| Zip artifact | 11 seconds  |	
| Upload artifact for deployment  job | 30 seconds  |	
| Post set up Node.js | 0 seconds  |	
| Post Run actions/checkout@v3 | 0 seconds  |	
| Complete job | 0 seconds  |

<br/>
Total time: 1 minute 48 seconds,  a 55.37% decrease.
<br/>
<br/>
## Bonus tidy up
 
As I was in the area, I also checked to see what the build package looked like after it was built. It contained the source code and the tests. I made a tweak to the tsconfig.json and removed even more files.
<br/>
<br/>
There was no drastic reduction in build time; however, it felt nice to know the build was clean.
<br/>
<br/>
## Conculsion

In this extreme example, I shaved around 40 minutes off the build process, mainly because the default GitHub action template isn't performant. There is some advice I would give to those reading which is:
<br/>
<br/>
![Check the size of your package today](/assets/img/posts/2023-01-08-why-package-size-matters/check-size-of-package-today.png)
<br/>
<br/>
An efficient build process means that you are waiting around less. A shorter feedback loop will help you build and test code faster. Think back to 2022; if each build you did took 30 seconds less, how many hours would you have saved?