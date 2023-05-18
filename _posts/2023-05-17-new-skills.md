---
layout: post
title:  "Picking Up New Skills: Career Fundamentals"
date:   2023-05-17 20:00:00 -0400
categories: career career-fundamentals
---

*How to learn new practical skills on the job with confidence.*

![a student practicing painting](/assets/img/new-skills/new-skills-header.png)

Ever since it was published, I can’t stop sharing Azeria’s blog post, “[The Importance of Deep Work & The 30-Hour Method for Learning A New Skill](https://azeria-labs.com/the-importance-of-deep-work-the-30-hour-method-for-learning-a-new-skill/)”. I’m always impressed at how well her post ties the concepts of deep work and deliberate practice back to cybersecurity. In it, she writes about the importance of choosing a focus, working without distraction, and forcing your brain into new habits that integrate regular feedback to improve your skills. This builds deep knowledge in a technical area over time through self study.

In my experience, while you can always learn from certifications and courses, this kind of focused learning is where the rubber hits the road. Applied self study is critical to accelerate your growth in cybersecurity. I’d like to build on this idea below, with how I have used deliberate practice to skill up and hone my craft in new work environments.

## The Process: Observe, Practice, Perform

Let’s look at how to take experience in self-study and apply it to the new, complex systems of work you’ll observe once you land a job in the field (or change to a new one!). How can we do this quickly? SOC and IR work, Penetration Testing, and Security Engineering have some of the same basics, but lead to very different day-to-day measures of success. Some of that success comes from training, and some in how you apply it.

## Why Do This?

When taking on a new technical role, there’s a lot to master fast. New work systems, team structures, processes, and not just skillsets… but how to apply those skillsets to get things done. If I run nmap, what do I do with the scan results? If I exploit a system, how do I report on it? Learning how this process works can take months, and be daunting.

I’ve been lucky to work in both “blue” and “red” team roles in my career. This has meant a few career pivots, which can be a struggle! This process was built to help combat this struggle. I didn’t realize I was using this system at first, but after the third or fourth go it seemed worth writing down.

## 1. Observe

![observe](/assets/img/new-skills/observe.png)

In each new role, you’ll be placed on a team of existing experts; your new team. Even if they don’t do *exactly* the same work as you, they still perform tasks similar to what you’ll need to do. Your work is likely to be measured similarly to theirs, even if your function is a little different. Thus, time interviewing and observing them is the quickest way to learn how *they* apply their expert skillset in a specific work environment to have an output.

When team members and mentors explain how things are done during training, listen closely. Take notes. Summarize what you’ve learned. Ask if there are reports, tickets, or other project outputs your new mentors would be able to share with you so you can see how they work. These notes and documents are inputs to the “Observe” phase.

### Steps for Observe

- ***Read through everything you can get your hands on:***
    - For penetration testing, this will might be past reports or (in internal roles) tickets. Are previous client reports available for review? What about sample reports?
    - For an internal security role, this may be alerts, tickets, or IR reports. If sensitive cases are not available for review, are redacted versions available?
- ***Review work as if it were your own. The key here is reading deeply, and observing:***
    - What process did the analyst follow to complete this work?
    - How do they document, write, and analyze?
    - What specific information did they include?
    - What questions are they answering with code snippets, tool output, and screenshots?
    - What is this analyst’s strength? What makes their work great?
- ***Write down steps you observe in a “cheatsheet” or personal methodology:***
    - This can be a list of steps you’d take to preform the same work, merging what you know from formal process documents with notes you’ve taken on how work is performed.
    - Writing your own reference will build your understanding of how things are done, and capture details that stick out specifically to you.
    - Continue reviewing and revising this methodology against additional documents. Once you feel confident you understand the process (or have reviewed 4-5 cases), move on to the next stage.

## 2. Practice

![observe](/assets/img/new-skills/practice.png)

My first “real” pentest, I felt like I’d been pushed into cold water. I had spent months training and carefully observing others’ work, and now I was independently accountable. Very suddenly, I wasn’t a student anymore… I was on the hook to deliver on time, with quality, to the client. Needless to say, this was stressful.

This next stage is meant to help avoid that “sink or swim” feeling by easing into things before you’re on the hook, and performing a few practice runs - either by thinking through or rerunning an assessment.

It’s key in this stage that you act as if this is *your project* as much as possible. Clients are waiting, computers are infected, and *you* need to take ownership of the situation. What do you do first? Good thing you just drafted a personal methodology!

### Steps for Practice

- ***For a chosen document, start from scratch:***
    - Choose a document (report, ticket, etc) to work with. Start with ones you’ve already reviewed, then move into ones you haven’t looked at yet.
    - Open a blank page alongside this document, and start writing…
- ***Think through the analysis fresh:***
    - For the given assessment (SOC alert, IR summary, pentest scope), think through and write down what steps you would perform for each step of your methodology.
    - Pretend you’re starting that work fresh as the lead for that task. What steps would you take? What questions need answering?
    - If you get stuck, briefly check against the past document and its notes as an answer key to help your understanding.
    - If you’re truly struggling, asking specific questions of your mentor (How do you do X?) may also help you get unstuck.
    - If it’s allowable, consider rewriting the assessment as much “from scratch” as possible. While you can’t re-run a pentest, can you review previous search or tool output? Can you spot the same ports and activity the original analyst spotted?
- ***Check your work:***
    - When done, check your analysis against the original document and see how your analysis, conclusions, and writing line up with the original. How did you do? What did you miss?
    - Pick another document, and go again! It’s alright if this takes several iterations to feel natural, as you’re deliberately pushing outside of your comfort zone here.
    - When you start to get the hang of it (or have done this 2-3 times), move on to the next stage.

## 3. Perform

![observe](/assets/img/new-skills/perform.png)

We’ve read a pile of documents, developed our own process, tested it against some more documents… when does this cross over to reality? This is where the real magic happens. At this stage, you’re ready to start working independently little by little. Instead of following the work of a mentor, now *you’re* going to do the work, and they’ll follow along to check if you’ve done it right.

This may already be a process in your environment. If it’s not, no worries! This is easy to take initiative on and should be well received by your peers, as it shows a lot of initiative. You may have heard this called “reverse shadowing”.

### Steps for Perform

- ***Take on a task:***
    - Choose a task you can assign to yourself and accomplish independently:
        - For pentesting, this may mean: requesting part of an assessment to work on (e.g. one part of an app to test), and drafting your findings.
        - For SOC work, this may mean: picking up a ticket, assigning it to yourself, and drafting your analysis.
    - Reach out to a peer with specific questions if you get stuck, but aim to work as independently as possible.
- ***Request a review:***
    - Once you’ve completed your work, reach out to a peer (or buddy, mentor, etc).
    - Request a review of your work with an in person or video call debrief, so you can get their full feedback on your work.
    - Complete any revisions they’ve recommended, and submit your work.
- ***Build independence:***
    - As you repeat this process, take on larger tasks. This may mean more of an application to pentest, or just working more independently on an investigation.
    - Complete these larger tasks on your own, and continue requesting feedback.
    - When you start getting positive feedback (or less suggested edits) from your peer or mentor, you’re likely ready to fly solo.

## Process Summary

![observe](/assets/img/new-skills/new-skills-recap.png)

As a reminder, here’s our new process overview for success:

**Steps for Observe**

- ***Read through everything you can get your hands on,*** such as previous reports, tickets, or alerts. If full documents are not available, can you review redacted documents?
- ***Review everything as if it were your own. The key here is reading deeply, and observing*** how the analyst completed their work and their strengths, writing style, formatting, and details. This is what you will emulate.
- ***Write down steps you observe in a “cheatsheet” or personal methodology*** that you can reference while performing this work. This summarizes what you’ve learned, reinforcing your observations on what works. Add details as you review additional documents.

**Steps for Practice**

- ***For a chosen document, start from scratch*** with a blank page to draft on.
- ***Think through the analysis fresh:*** What would you have done here? What steps would you take? Pretend this is *your* incident, report, or investigation.
- ***Check your work*** against the original analysis, and see how your work lines up. How did you score?

**Steps for Perform**

- ***Take on a task,*** such as an alert to investigate or a part of an application to pentest. Complete this work as independently as possible.
- ***Request a review*** from a peer or mentor, and ask them for honest feedback on how you did. Does this work match what they would do? Revise based on their feedback.
- ***Build independence*** by taking on larger tasks, completing them, and requesting feedback. When you start getting short, positive feedback (”Great job! Looks good.”), you know you’re ready to go solo!

## Conclusion: Why Deliberate Practice?

Why do this instead of (or along with) your employer’s onboarding? What’s the point in this process if you’re going to have training anyways?

For me, it’s because every role is different in a way that documentation can’t capture. I’m immensely thankful for each role I’ve held, both red and blue team. But each individual role has been different in culture, focus, and output. In each of these new environments, even if the process of the work is documented well, I’ve never seen exactly how others perform their work and succeed at it on paper. Everyone has a different style of success, and writing what works for one person as the required process for another wouldn’t work out well.

Additionally, cybersecurity is still relatively new. This means the teams you join may be less than 5 years old, or part of a new office, or at a startup. Your peers might have a lot to teach you, but no time to have written it down yet. In an ideal world, this would all be written down - but sometimes, you’re the first one writing it.

Let yourself become a reporter on your role. How do great incident responders do what they do? What’s the secret to your team’s greatest hacker? Try these techniques out, and see how they work for you. Along the way, this process will put career success more into your own hands, and help you find the process that works for you to succeed.  
  
<br/>
  
<sub><sup>*Image Credit: Unsplash, DALL-E (header)*</sup></sub>