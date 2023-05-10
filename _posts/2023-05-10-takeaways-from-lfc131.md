---
layout: post
title: Takeaways from LFC131 - Green Software from Practitioners
subtitle :
image: assets/img/posts/2023-03-05-github-actions-storage-limits/social-media-post.png
imageWidth: 1080
imageHeight: 1080
tags: [GitHub Actions, DevOps]
author: Thomas Billington
comments : False
---
I've recently taken the [Linux Foundation Green Software for Practitioners course](https://training.linuxfoundation.org/training/green-software-for-practitioners-lfc131/), which explores a set of green principles for building and deploying applications while providing some practical tips to make your application greener.
<br/>
<br/>
In this blog, I'll run through my two takeaways from this training course on how to apply green principles next time I build/design something.
<br/>
<br/>

## Learning 1 – Server Utilisation

I have a habit of provisioning a larger resource than I need; for example, one of my current websites runs on a Windows server that is only 25% utilised. Doing this isn't good for my wallet or the environment.
<br/>
<br/>
Most people assume that a server uses a percentage of the maximum power based on the server's busyness. As per the image below:
<br/>
<br/>
![Linear power consumption example]](/assets/img/posts/2023-05-10-takeways-from-lfc131/linear-power-consumption.png)
<br/>
<br/>
The above calculations are incorrect; Google did some research back in 2007 into [Energy Proportional Computing](https://research.google/pubs/pub33387/), which shows that a server just being turned on will use approximately 50% of the maximum power, as per the graph below:
<br/>
<br/>
![Energy proportional computing graph]](/assets/img/posts/2023-05-10-takeways-from-lfc131/energy-proportionality-graph.png)
<br/>
<br/>
That means that the actual usage calculations would be:
<br/>
<br/>
![Energy proportional computing example]](/assets/img/posts/2023-05-10-takeways-from-lfc131/energy-proportionality.png)
<br/>
<br/>
When building applications, we should attempt to use as little energy as possible. Based on Google's findings, I should be actively trying to keep all my services running at a high utilisation with auto-scaling enabled to deal with spikes. As a bonus, I'm avoiding overprovisioning, which means I'm potentially reducing my monthly cloud bills.
<br/>
<br/>

## Learning 2 – Where and when to run my code.

When deploying an application into the cloud, I usually pick the cheapest or closest to me. However, that's not always the most environmentally friendly decision. 
<br/>
<br/>
I could strategically position or run automated jobs, batch jobs, and data processing tasks to make the most use of green electricity. I would need to define the schedule so the code runs when the power grid has the most carbon-free electricity to do that.
<br/>
<br/>
If our data centre is somewhere with lots of sunshine and uses solar power, scheduling the jobs during the day would be better for the environment. If you can't change the time you need to run the job, you could run the job in a data centre with the highest level of carbon-free electricity for that time of day.
<br/>
<br/>
Say I was deploying on [Google Cloud](https://cloud.google.com/sustainability/region-carbon) to ensure I was always using carbon-free energy; I could deploy to Montréal, which uses 100% green power. I would avoid places like Singapore which is only 4%
<br/>
<br/>
The same with [AWS](https://sustainability.aboutamazon.co.uk/environment/the-cloud?energyType=true); currently, 11 out of 31 regions are powered by 95% renewable energy.
<br/>
<br/>
Over time, the cloud providers will move most of their operations to renewables, and as engineers can ensure we are running our code in the greenest places possible.
<br/>
<br/>

## Conclusion

I've spent a day looking into green computing and how I can make more environmentally friendly decisions when building applications. From now on, I'll be checking to see if:
<br/>
<br/>
I'm provisioning the correct infrastructure instead of going up a tier in case I need it.
I'm deploying and running my code in sustainable locations.
<br/>
<br/>
If you have a spare hour or two, I recommend the [Linux foundations course](https://training.linuxfoundation.org/training/green-software-for-practitioners-lfc131/), and if you like collecting things, you also get a nice badge on Credly.