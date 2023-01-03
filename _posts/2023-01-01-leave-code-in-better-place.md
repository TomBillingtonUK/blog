---
layout: post
title: The one habit all software engineers should have
subtitle : And it will only take you five minutes to implement
image: assets/img/posts/2023-01-01-leave-code-in-better-place/social-media-post.png
imageWidth: 1080
imageHeight: 1080
tags: [Software Engineering, Principles]
author: Thomas Billington
comments : False
---
As software engineers, we typically build software within the confines of the project management triangle. We can never deliver perfect code with all the features our customers want in the timeframes and budget that keep the management happy. Quality dips when the triangle sways too much in one direction, and we create technical debt.
<br/>
<br/>
![Project Management Triangle](/assets/img/posts/2023-01-01-leave-code-in-better-place/project-management-triangle.png)
<br/>
<br/>
Life as a software engineer is also a path of constant learning. The team members around you will also accelerate your knowledge, so there will be a continuous flow of new ideas and approaches into the code base. The output will create inconsistency in the code base as your code will look different today compared to twelve months ago.
<br/>
<br/>
As engineers, we will never have perfect code, zero technical debt, or consistency. It's impossible to do as our customers will always want new features and management will only give us the time and money to achieve it.
<br/>
<br/>
To reduce the inconsistency and tech debt I've seen teams:
- Dedicate a percentage of a sprint to technical improvements (Usually a 60/20/20 split Features/Bugs/Tech Debt)
- Spend an entire sprint to technical improvements
- Only committing engineers to a 60% workload so the other 40% can be used for meetings, personal development and technical improvements.

<br/>
There is one problem with these ideas, for most engineers, these decisions are out of our control. On top of that, these initiatives are also subject to the project management triangle.
<br/>
<br/>
So what can you do that you can control? The answer is easy: following a simple principle and building a habit to ensure you follow it.
<br/>
<br/>
![Always leave the code in a better state than when you started.](/assets/img/posts/2023-01-01-leave-code-in-better-place/quote.png)
<br />
<br/>
I'm not saying spend hours refactoring every time you go near a code file. What I'm suggesting is that before you put up a Pull Request, spend five minutes making some small tweaks and improvements, such as:
- Renaming variables/functions
- Improve the logging (Operations will love you)
- Adding a comment if you had to work something out to save the next person doing the same.
- Restructure a file

<br/>
If you are unsure what to tweak, ask yourself how to improve the experience for the next engineer who visits this code.
<br/>
<br/>
Imagine how much better your code base would become if you had ten engineers, each spending that 5 minutes, two or three times a sprint over time.
<br/>
<br/>
So before you put that pull request up for review, spend 5 minutes improving the experience for the next engineer.