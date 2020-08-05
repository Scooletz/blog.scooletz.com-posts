---
layout: post
title: "Designing a better recruitment process"
date: 2020-08-10 11:00
author: scooletz
permalink: /2020/08/10/designing-better-recruitment-process
image: /img/2020/08/hire.png
categories: ["business", "company", "recruitment"]
tags: ["business", "company", "recruitment"]
whitebackgroundimage: true
twitter: true
---

This is my personal story about designing a better recruitment process. I'm far from being a HR expert or a recruiter. At the same time, discouraged from a traditional approach by various sources, I took some steps to make it much better and, hopefully, hire better people for my company, [Dotnetos](dotnetos.org). If you think that it'll be another sad post about **cognitive biases** that includes self-blame and stating that _that's the way things are_, it won't. It will be about **actions and processes used to overcome** them.

## Fear

The first came fear of failing greatly. **If you think that hiring people is easy**, let me provide you with the following sources:

1. [Job Interviews Don’t Work](https://fs.blog/2020/07/job-interviews) from Farnam Street. Long read about interviews being probably the worst tool if you want to use an unstructured, non-repeatable gut-feeling-driven interviews. They don't work.
1. [Thinking Fast and Slow](https://www.goodreads.com/book/show/11468377-thinking-fast-and-slow) by Daniel Kahneman. A story of two systems that makes you. If you don't feel interested, just take a look at [Linda Problem](https://en.wikipedia.org/wiki/Conjunction_fallacy) at wikipedia. This should get you interested.
1. [Jeff Hunter: Embracing Confusion](https://fs.blog/knowledge-project/jeff-hunter/) from the Knowledge Project. Jeff Hunter was leading HR in Bridgewater Associates, the company founded and led by Ray Dalio[^RayDalio]

Even with the whole knowledge available, human beings are still cocky, unreasonable and biased. The knowledge does not make it better in any way. The only approach that looks reasonable is process based. Having a **repeatable scheme**, that one can apply to every person, especially if some tests could be done in blind, would be great.

## Screening

The first thing was screening candidates. Initially, before putting an ad online I went to LinkedIn to search for some profiles, peoples, ideas that I could work with. It was interesting that in the search window I had no option to disable display of photos or other personal information. Maybe with a custom tooling I could do this, but I had no idea how to do it on LinkedIn itself. I even asked their Twitter profile whether it's possible to alter the way profiles are displayed:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Hi Szymon, thanks for reaching out! We do not have this functionality available currently but I&#39;ll make sure to pass this feedback to the relevant team here! Have a nice day, TB</p>&mdash; LinkedIn Help (@LinkedInHelp) <a href="https://twitter.com/LinkedInHelp/status/1281159219421548544?ref_src=twsrc%5Etfw">July 9, 2020</a></blockquote>

Eventually we used a different platform for hiring. It provided full view as well (including photos). The initial search was made broad enough to get over 100 people from over 1000 candidates. If a person had some matching points with the description, they might be a fit. I hope this narrowed a potential for **false negative** screening.

## Survey and the first score

The next step was totally **asynchronous** for candidates and for us. All people that passed the initial screening, were given **a short survey with a few tasks** and estimated amount of time that it should take to solve. All the answers were collected in a spreadsheet. The asynchronous nature of it was beneficial in two ways. The first is that I work in an async remote environment and this enables processing answers when there's a time for it. The second is that it shows the asynchronous nature of the company.

The scoring part was designed to remove outliers. Every person scoring answers, did it in a blind way. They were **scoring answers without knowing** who the responder was. At the same time, the fact that all the data were provided in a really simple plain text form was helping to remove any influences. The last but not least, the final score for a specific task for a specific person was based on **median of all the scores**. This **removed outliers** and prevented that one person having a bad day (let's assume it's me) from destroying the scoring process.

The outcome of this step was a single combined number assigned to each candidate. All the people that were above the median (upper half) were asked to **schedule an interview video call**.

## The interview

All the interviews were based on the same list of questions. The questions were gathered internally. Good questions are not enough though. You need to know some points that you're looking for to **make them comparable**. With this it was much easier to convey the interviews.

You may ask now, that during a call the interviewer could have a bad day too. To amortize this, all the interviews were recorded and watched by every person in the process. It was not required to watch the full video, but it was somewhat encouraged. Even with the 1.5x multiplier it might be hard to do it. I think that getting any insight, beside the interviewer was important. Again, the very same **median of all the scores** was used in this case.

## The final step

The final score was a weighed combination of the survey score and the interview score. Both of these component were based on median removing outliers. It ensured that **interviews are not more important than other parts**. With this final score, it was an easy task to select TOP N and to proceed with the selection of candidates that we want to hire.

## Summary

The process consisted of:

1. an initial screening:
    - potentially biased
    - a high number of people passed through to minimize `false negative` cases
1. a survey/test
    - scored in blind, without knowledge who is the author
    - the output score is the median out of the scores to remove outliers
1. an interview performed according to a scenario
    - slightly biased (you can ask same question with a slightly different intonation)
    - the output score is the median out of the scores to remove outliers
1. total score
    - a formula combining scores above
    - ensures that interviews are not more important than anything else
1. the final decision
    - based on TOP N from the scores

One can say, that the whole process is **depersonalizing** and that we're loosing humanity between all the scores and numbers. I think that this is the only way to be human, fight biases (instead of talking about them all the time) and make the process **repeatedly good** for every single candidate.

[^RayDalio]: Ray Dalio is the author of [Principles](https://www.goodreads.com/review/show/2254565243), an amazing book about a principal approach to life and work. I highly encourage to read it.
