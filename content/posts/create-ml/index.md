---
title: "Creating ML"
date: 2025-09-01T13:11:00-07:00
tags: ["Create ML", "ai", "learning", "machine learning", "Core ML"]
draft: false
---

# Create ML - A Very Appl-y ML App
It's amazing and exciting that anyone with a computer can build and train an ML model. Having said that, it's also like using Core Audio was a decade or two ago - quite a lot of overhead just to get to "Hello, world!", especially if you're somewhat new to the process. Enter Create ML, Apple's answer to machine learning for the masses.

## Foundation models are great...
As I've written elswhere in this blog, my current "main" project is an app for identifying logical fallacies in text passages, and learning what to do about them. The "heavy lifting" is being done by Apple Foundation Models (AFM) which will ship for the first time in iOS 26 (still in beta at the time of this writing). AFM creates some amazing possibilities that were hitherto either impossible or impractical. What is the intersection of all these constraints, and did it even exist prior to AFM?

* "Real" LLM
* Fast enough for a lot of purposes
* Runs on device
* Tight integration with Swift/Xcode
* Doesn't bloat app footprint
* No licensing or subscription fee

![Aaka](AFMVenn.001.png)

So much to be excited about! I should note that "fast" is relative and subjective. For the purposes of my fallacy-detection app, fast means "fast enough", which in this case is ~2 seconds to when fallacy findings start streaming into the UI.

## ...but not the greatest for everything
It turns out that if you have a much narrower, ML-friendly task, you inference can execute a few orders of magnitude faster with a good-old ML model than with an LLM, while performing better in all sectors of the confusion matrix. A great example of such a task is the part of the fallacy-detection app that tries to answer "is this text something that can even be analyzed for fallacies?" This gate is needed because AFM, in all its compulsive glee to oblige, will steadfastly find fallacies in source code, random gibberish, etc.

Since this type of classification task is more or less a cliche, it's ideallly suited for the "on rails" workflow of Create ML. You just:

* Point it at your training data
* Choose some options from drop-down menus
* Press "go"
* Save the resulting model into your project

Yes, I'm eliding a lot of steps here. But compare the above to the workflow of using other tools (pytorch, tensorflow) to generate a model, and you'll recognize that those steps fall out of the comparision because they exist no matter what (e.g. deciding your output classes, preprocessing data, etc.). Those tools also require a lot more setup and boilerplate even to do pretty straightforward stuff.

My point isn't to say that one is better than the other. Needless to say, the open-ended tools are far more powerful in terms of the possibilities they offer for custom model creation. But it is really convenient to have something like Create ML for cases where all that power isn't needed.

Using the constraints above, we still have:

* Real ML
* Often blazingly fast
* Runs on device
* Tight integration with Swift/Xcode
* Adds a few MBs to app footprint (certainly acceptable in the case of my fallacy app)
* No licensing or subscription fee

## Random findings
* ChatGPT is an excellent sidekick when putting together a training data generation, augmentation, and post-processing pipeline
* BERT models don't seem to work in the Simulator, as they don't find their embeddings (they work great on device, thoug)
* fun fact: the amount of time it took to _train_ the Max Entropy version of the classifcation model was comparable to the amount of time it took to run classification inference via AFM