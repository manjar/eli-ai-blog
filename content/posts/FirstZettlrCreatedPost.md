---
title: "Development log: using Apple Foundation Models"
date: 2025-08-20T15:00:00-07:00
draft: false
tags: ["zettlr", "hugo", "papermod"]
categories: ["blog"]
---

# Notes on using Apple Foundation Models in my "Fallacy Detector" app
The following are some notes on Apple Foundation models. Since I'm using the Apple on-device models for my logical fallacy detector app, I'm starting with some background that led to its development. If you're eager to get to the discussion of the models you should jump to "Quick app overview" or "First attempts using APIs".

## My history with logical fallacies
I've been interested in logical fallcies for many years. This was initially due to losing a lot of arguments, often in ways where it didn't seem like facts or logic carried the day, and probably also due to 'winning' some arguments in a similar manner.  I wanted to understand the various devices people used - consciously or unconsciously - to prevail in contentious discussions, often without really moving the conversation or understanding forward.

I do believe that most of us, most of the time, are sincerely seeking the truth, ideally some kind of shared truth. And many of us, myself definitely included, struggle with staying true to that search at least some of the time. I am optimistic that learning about logical fallacies, and other deeply studied and well-documented areas of human communication, can help us improve as a society.

Back to the story, over the years I have found various places to get information about these fallacies. With the public release of ChatGPT, however, I immediately sensed its capability to wade into fuzzy (from the perspective of someone used to algorithmic programming) areas like human language. I found myself prompting it repeatedly to "find the logical fallacies in the following text:", and the results I got back were generally useful. I found myself running these queries so often that I thought it might be better just to have an app.

A quick shout-out: on my learning path I found useful resources at [School of Thought](https://www.schoolofthought.org, ). They seem very committed to promoting understanding of these topics. And [Wikipedia](https://en.wikipedia.org/wiki/List_of_fallacies) has an extensive list of fallacies if you want to check that out.

## There oughtta be an app for that
Enter my new app, Fallacy Detector. It originally started the first time I sat down to try "vibe coding" which I'd heard so much about. Grasping for an idea to implement, my first thought was to create an app to detect fallacies. In the spirit of vibe coding, I went with that thought.

A couple of hours later I had a jumble of code that compiled and ran. It was a dead-end mess that I couldn't wait to abandon. But it worked just well enough to convince me that I actually wanted to develop a "real" (is that the right opposite of "vibe"?) fallacies app. So I set about to do that.

## Quick app overview
For the sake of discussion, let me briefly describe the intent of the app. (Though if you want to cut to the chase and become a [beta tester](https://testflight.apple.com/join/E7XdSWJW), please do that!)

The basic workflow in the app goes like this:
* Paste, drag or type some text into the app
* The app analyzes the text for logical fallacies
* Any fallacies found are highlighted, and the following additional information is provided:
    * A name and description of the fallacy
    * Suggestions about how to avoid using this fallacy, or how to counter it when you come across it
    * A link to the Wikipedia entry for that fallacy

It's a classic "database app" that also stores the fallacies and presents them in a table view, so you can go back to them later.

## First attempts using third-party web APIs for LLM inference
When I first started development of the app, Apple Foundation Models were not available (at least not outside of Apple to regular "customers" such as myself). My approach, therefore, was to use web APIs for inference. This immediately created three challenges:
1. Getting the API to reliably respond in a format that could be parsed and translated into programmatic objects
2. Dealing with the fact that these APIs are not free
3. Implicitly requiring the user to face the privacy implications of shipping their text to a third party

Regarding the first issue, it didn't take too many iterations before I could get the LLMs to reliably respond in well-formed JSON. These were readily decoded into in-memory objects, usually. It still felt squishy and hack-ish, but on an practical basis it was working out.

The second challenge was more vexing. I did not want to have to ship a subscription-based version of the app, mainly because I don't like subscription-based apps myself. Also, while I had an API key that I could use for my own development purposes, I did not want to fund an unbounded number of API calls from users' installs. My solution to this was to create additional UI and secure backend to support BYOK (bring your own key). With this,  the user could get their own API key directly from OpenAI or Gemini (for as little as $0), register it once with the app, and execute model calls on their own account from then on. Shortly after I finished my first implementation of this approach, the Apple Foundation Models became available to people like me.

## Switching to on-device models
The Apple Foundation Models were a "direct hit" against the issues I was facing using third-party APIs. Specifically:
1. Instead of coaxing JSON out of the language model, I could get fully-formed structured objects back right from the model. Errors could still occur, but it was in one presumably highly-tested place (instead of in my hand-written parsing or decoding code).
2. Since the models run on-device, I no longer have to worry about API keys and their associated costs. Meanwhile:
    * Latency seems to be in the same ballpark
    * The analysis could now take place without a network connection
3. No information leaves the device

## Challenges so far using the on-device models
I have encountered a few challenges working with the models so far. Unfortunately, I've long since removed the third-party adapters from the app, so I'm not in a position to very easily see whether these challenges are characteristic of the Apple models in particular.

One challenge I encountered was when trying to extend the behavior of the main analysis prompt. Having accidentally pasted some Swift code into the app, and having watched the app dutifully find false dichotomies, strawmen, and more in it, I realized I needed a front-end classification of the text into "analyzable prose" and various other categories. This would allow me to skip the analysis in such degenerate cases and even explain to the user what happened. My initial approach was to simply amend the main prompt to perform this initial classification, set a field in the returned object, and proceed to look for fallacies only if the text was analyzable.

The result was a lot of really weird behavior. Text would be classified as "sourceCode", yet the model would still go ahead and find various fallacies in it. After many attempts to engineer the prompt into submission (including few-shot examples and various rephrasings), I decided to simply break the classification task out as a separate LLM call. This immediately resolved the issue. It did add latency to the overall flow, however I would need to compare it to the latency impact of adding this classification task to the analysis task, if I could have gotten that to work. Nonetheless, I'll be thinking about ways to perform the classification with a more targeted and hopefully less latent model or algorithm.

## Current state of the app
The app is currently in [beta via TestFlight](https://testflight.apple.com/join/E7XdSWJW). Please do join! I'm eager to get feedback.