---
layout: post
title:  "Home cooking apps with AI assistance"
date:   2024-12-19 17:30:00 -0400
categories: career reflections
image: /assets/img/homecooking-apps/header.png
---

> *This is a reflective end-of-year post on using AI to make our app wishes come true. Happy holidays! ❄️*

## Markets of one
I came across Robin Sloan's "[An app can be a home-cooked meal](https://www.robinsloan.com/notes/home-cooked-app/)" early this year, by way of Hrishi Olickel's "[Home cooked apps](https://olickel.com/my-home-cooked-meals)". Both are phenomenal reads. I highly recommend indulging in them. These posts have stuck with me, with their mutual sentiment of building more for ourselves and the people we love.

There are times when technology accommodates near-infinite customization: [WinAMP skins](https://skins.webamp.org/) are a classic in this arena. The more control a user has over customizing a product, the more wildly creative or specific to user interests it can be:

<img src="/assets/img/homecooking-apps/winamp-skins.png">
*(Source: Winamp Skin Museum)*

Regardless of what free time each theme took to make, that the product supported - even *embraced* - theming as a common user interest is part of what made it so delightful. Personal. Indulgent, even.

On the flip side, the more widely convenient a product is, the less uniqueness it can afford. Support for user-controlled code takes both time and risk to implement. Suddenly, something as simple as dark mode can become out of reach. Does this request directly help pay the bills? No? We don't have time for it.

There's nothing inherently **wrong** with this, economically: A wider target audience means lower product costs, which means more of us can afford to use said software. More practical features means more practical impact. Minimum viable product (MVP) development means building more market-valuable software faster.

***But you might be only person who sees value in your feature request.*** 

We spend so much time playing in others' walled gardens. Why shouldn't we build our own sometimes? 

Can't we build for our own markets of one?

## AI fueled tinkering
Let's change gears for a moment: AI has been heralded as a market changer. Anyone can become a startup... for everyone! Scaling to infinity! With AI's emergence, we're encouraged to see new markets to conquer, customers to serve, with ever more side hustles.

While these features are thrilling, there's a quieter side to this revolution. AI assistants remain free or low-cost to coach us on coding new features, or to write those features for ourselves. 

What apps have *you* wanted most in this world? What features have you needed, but not had time to fully code yourself? What little apps could make you *smile* every time you used them?

For those of us without a development background, it may not occur to us to create our own app. Isn't AI... *not so good* at full, complex development?

What makes AI ill-suited for production use makes it a great fit for personal projects:
- While it can't write complex projects yet, it excels at solving simple problems with code.
- If you have a plan, you can coach it through creating the pieces for you.
- It has access to tons of common solutions to frequent issues. (StackOverflow!)

Results can be sloppy, but does that matter if you need something basic? Code doesn't need to be production-ready. It just needs to meet our own needs.

Costs of this early AI usage are still pretty low, and it's not clear [how long](https://www.cnbc.com/2024/09/27/openai-sees-5-billion-loss-this-year-on-3point7-billion-in-revenue.html) [costs will stay](https://www.reuters.com/technology/artificial-intelligence/openai-remove-non-profit-control-give-sam-altman-equity-sources-say-2024-09-25/) [low enough](https://techcrunch.com/2024/09/27/openai-might-raise-the-price-of-chatgpt-to-22-by-2025-44-by-2029/) for individual tinkering.

So while the feature is available to us: We should do some wonderful, [useless things](https://kknowl.es/posts/useless-things/), things that have no market value to anyone but us.

## Example: Brevity, a home cooked editor
A few months ago, I wanted a local web editor to improve my writing. I sometimes use a certain web-based editor to refine my public writing and CFP submissions. The editor scans my text, and provides feedback on how to improve it.

However, it comes with drawbacks:
- It's an online application using my data for who-knows-what
- It tries to upsell me on features I don't need
- I can't update it with my own writing rules or vocabulary checks

Sam Williams' post "[How I reverse-engineered the Hemingway Editor](https://www.freecodecamp.org/news/https-medium-com-samwcoding-deconstructing-the-hemingway-app-8098e22d878d/)" caught my attention as more or less what I was looking for, but it didn't have a live-edit feature.

I gave my own version a shot with ChatGPT's Canvas feature.

Over the next couple hours I gave requirements, requested additional features, and tested the results. Some more complex issues, like scanning text repeatedly for errors, took time to iron out. The agent got confused occasionally, requiring questions like "Is there a less resource-intensive way to do this?" in order to refine towards a better method.

The code was a little sloppy! But perfect wasn't really the point, was it? The end result was cute, workable, and ran offline:
<img src="/assets/img/homecooking-apps/brevity.png">

Most importantly, I could add or change rules when I needed to. I could quickly update the theme, or tweak the highlight colors. I could add a Galaxy theme! This was truly *my* editor.

For the curious, I've shared my final results here:
- [☕️ Brevity: An offline, single-page app to improve your writing](https://github.com/siigil/brevity)

It's not the most resilient thing on the planet, but it does what I need and it's mine. I'm excited to keep building on it.

## Conclusion
When publicly accessible AI models exploded in 2022, there was a [groundswell of articles](https://www.cnn.com/2022/09/03/tech/ai-art-fair-winner-controversy/index.html) on its creative potential. In time, this potential has led to some [truly ridiculous slop](https://www.404media.co/facebook-is-being-overrun-with-stolen-ai-generated-images-that-people-think-are-real/). But there are still great ways we can use this technology to our own advantage.

I don't know how long this phase of accessible AI will stay. The market value of billions of computing resources eventually needs to be realized. There are more widely convenient products to build for more [widely convenient groups of people](https://nothinghuman.substack.com/p/the-tyranny-of-the-marginal-user), more side hustles to be realized, more market share to gain.

But for now: AI means that cozy, home-cooked feeling *also* possible for an individual to attain. The cookbook of apps is open to me, a relative novice, and as long as it's here I'll use it.

I like to think that this is what AI loves doing, too.


<sub><sup>*Header Image: Unsplash*</sup></sub>