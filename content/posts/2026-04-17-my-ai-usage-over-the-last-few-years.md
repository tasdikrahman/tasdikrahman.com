---
title: "My AI usage over the last few years"
description: "A personal changelog of how my AI tooling and workflows have evolved, from chat interfaces to Claude Code and agentic programming"
tags: [ai, llm, claude-code, productivity]
comments: true
share: true
cover_image: ''
---

This post is going to be mostly a reference check for myself on how my AI usage has evolved over the last few months and even last year. The tools are moving at quite a pace, and I just wanted to write this down to have a reference for myself on how my usage has evolved over time with AI tools. 

## Chat-based interfaces for searching something instead of default search engines 

This was the first interface that I had used for trying to do research on something instead of using a search engine first, and I must say it has evolved into quite something now. 

I wouldn't repeat what others have written already, but just to summarize my own thoughts. Instead of doing search engine-based research now, I lean more towards asking one of the frontier models, as of now it being Opus 4.6. Essentially, starting a chat and then I keep coming back to it. Since the chat also has memory of the whole conversation, which I've been carrying over for multiple weeks or even months sometimes, it has all the context it needs 

For example, if I was planning for a trip soon (probably one of the most used use cases), I would like to provide my inputs in terms of what I want to do. See the number of people joining, their preferences, budget in general, where we would want to stay, and what each person usually defaults to. The suggestions are usually a very good starting point on what you would want to do, but at the same time it's also important to note that these suggestions would be sometimes quite generic. So a travel guide book is still not completely something which you would not want to use. 

Some of these frontier models have this ability to group your chats into projects, which I find very intuitive and also useful, so that I can refer back to, inside each project, different chats which I've had. I think as of now, Gemini still is yet to add that, perplexity, as well as Claude, along with ChatGPT, have this for quite some time now when I use them.

## When did I start paying for this?

In the order of usage, I started using ChatGPT at first, and then I moved over to Gemini and Perplexity at the same time. But all of them in their free versions 

Sometime last year, I started using O3 from ChatGPT as one of the first paid subscriptions that I made. Honestly, the results were vastly different from the free models which I was using over the years.

I had briefly stopped paying for ChatGPT for a while because I got a Perplexity Pro subscription through a discount for a year.
But then I moved back to Claude simply because I wanted to start using Claude code for my personal usage also. That happened sometime in Feb this year.

## Building context over time with files

Quite honestly, I feel the CLI approach is pretty strong. Even for use cases when I'm not actually coding 

For example, I routinely use a lot of code to create some Excel sheets where I am running some numbers for some personal calculations. 

I'm pretty sure some other people also do the same, but what I have tried doing now is to insert memory or context inside my current work, which I have been doing. What I end up doing is I end up writing it to a markdown file or an Excel sheet if I'm running some numbers. Over time, the context builds up on either of these file types

This way I don't have to start from scratch each time I'm trying to prompt for something. This used to actually happen before when I would use the chat interface for some of these AI frontier labs. I would have to repeat what I had researched for quite a bit, or the memory would be lost, and I would have to redo a lot of these things. Wasted computation cycles also. Now, if I'm using a text file on CLI and it has all the context from the previous research, it just picks it up back from there, essentially saving a lot of cycles and also the knowledge which has been gained over the last few discussions. 

What I also like similarly in this approach is also for some of my personal Excel calculations. This works pretty well 

## On agentic use cases and beyond 

I haven't tried out openclaw or a similar setup yet, but I do feel that it is certainly something which will evolve and mature over time. For example, if I wanted the agent to remind me and do a re-prompt of something which I had told it to do for me at a certain date and time. The current usage of mine, which is through either Claude Code, CLI, or through the chat apps, doesn't work as of now, or simply put, doesn't have this ability 

So in future, I think I would certainly want to try out one of these setups

## Code generation use cases. 

I seriously started using code generation via GitHub Co-Pilot sometime last year. I was trying it out on Visual Studio Code at first, and it used to have this ask mode inside it. 

Sometime during last year, Agent Mode was added into what began to be called Agentic Programming. I did use it inside GitHub Co-Pilot in Visual Studio Code, but my experiences were a bit varied in terms of usage, and I would usually just default to Ask Mode. 

I started with just the same file being the context file, and eventually, with each prompt, I would give the whole directory I was trying to give it context for. This used to be a cumbersome process, for example 

My first real experience of making use of AI to do a real task was when I had to upgrade a lot of dependencies in a Go project which I was using. The challenge was mostly that the Go project had a lot of dependencies which depended on each other and had a common dependency where upgrading one of these dependencies would cause the other to break. This was the challenge during that upgrade. Nevertheless, I threw the LLM on it the first time sometime around the middle of last year and did a lot of prompting at some points, I did feel I think the LLM was going around in circles and circles, so I had to manually intervene quite a lot at that point, which I think is termed as hallucinating. 

## Claude code and how it changed my coding workflow 

At some point last year, during the last few months of the year, I started using Claude Code, and that I feel has changed the way I use AI for coding. 

But I also feel that having the context windows increase was also an effect in general. I think what Claude Code naturally got, or agents in general got, was the terminal being the interface. Since the CLI tools are the interface for developing something and working around the developer tooling, once you are in a tight feedback loop with what you have added, you run your tests and other linters and other feedback loops being integrated inside your project setup. The agent can work very easily with verifying whether something it has added is working or not. 

My experience has been that scaffolding has never been easier, and at the same time, test generation has become quite easier. Of course, you need to tune down and remove a lot of these tests which AI adds, where it might be testing something or paths which might never be needed to be tested, because the tests are also code and need to be read again and again. Sometimes it can get quite verbose, and that's where I think the decision on the developer itself comes to what to accept and what not to. 

But in general, I think the whole plan mode and execute mode, as being two separate terminal windows for something that you are developing, has certainly improved my workflow over time. 

Another thing which I do is have multiple code sessions open in my terminal. For example, if I am working on a couple of features inside the same codebase or in different codebases, what I have sometimes ended up doing is:
- on one window, there is a plan mode for one feature and execute mode on the other
- and then similarly a couple of more similar paired sessions of age planning and executing
 

Although context switching is difficult sometimes, but I think the idea in itself is quite powerful if you are, for example, doing a language version upgrade for a programming language in one context window and your test harness is pretty strong. All the agent has to do is keep running tests and see if the things function, and the feedback loop is quite tight. You can come back at some point on the other window and focus on the first window, planning a feature and brainstorming

Which brings me to the plan mode. What I usually end up doing now is dumping a lot of information on exact product requirements for the feature and giving it context in the directory for the project I'm working on. Having done that, what I end up having is now the whole context of what the project currently has. What needs to be done? Sometimes it will deviate into different directions, at which point I would just jump in and course correct. 

As recently as yesterday night (for me, local time), I ended up seeing Claude Code releasing their new model, Claude Opus 4.7, so we'll probably write more about that soon. 

I still remember Claude Opus 4.6 getting released in Feb, and that is when I stopped using Perplexity and started paying for a paid subscription for Claude code and installed it on my personal machine. To test it out, I want to do a migration of my blog, which was written in Jekyll to Hugo, which I ended up doing in a record short time. It could have taken much more than if I would have done it myself. I have written about it in a blog post, which I will link here if you want to read about it. 

## Future

Which brings me to my final summary of the blog as of now, is that the frontier models are moving quite fast. Something which I have observed and also independently confirmed with a couple of my friends is that the frontier models have gotten really well, and it surpassed a threshold sometime around the end of last year. For me personally, that was when I started realizing through personal use how good it has become at generating code and understanding code as compared to when I first started using it actively sometime in the middle of last year before that, it was mostly just generating one-off scripts in 2024. And mostly doing smaller things, but I think the models have had a giant leap, and the cat is really out of the box, I feel

What is there to be seen for this year? I would be really curious to see how things evolve, but one thing is for sure: I feel models will start getting better and better and also cheaper to run. Recently noticed the Gemma models released by Google, and most of them seem to be pretty reasonably sized to be able to be run on a personal machine, which I think was not really possible for the last few models. 

I think more and more of these models would be becoming more efficient and be able to run on machines which are off the shelf, which I think would be the real utility out of the improvements. I guess I'll look back on this blog in a few months to see how things have progressed and my own usage has changed, or what I have re-learned over time. Until next time then. 
