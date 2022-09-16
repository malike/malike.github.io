---
layout: post
comments: true
title: "ChatOps : Building A Custom Slackbot"
layout: post
author: malike
categories: [devops,sre]
fossname: slack-bot
fossurl: https://malike.github.io/slack-bot/
fossdescription: "[ChatOps] A slackbot that you can use to create actionable notifications,monitor AWS S3 and IAM policies."
fossimage: http://via.placeholder.com/220x220?text=Logo
tags: [aws,slack]
image:
  path: /posts/ai.png
  width: 800
  height: 500
  alt: kubecon.
---

### ChatOps

_"ChatOps is a new operational paradigm - work that is already happening in the background today is brought into a common chatroom. By doing this, you are unifying the communication about what work should get done with the actual history of the work being done. Things like deploying code from Chat, viewing graphs from a TSDB or logging tool, or creating new Jira tickets…all of these are examples of tasks that can be done via ChatOps"_

Although the use cases vary for different organizations some of the common things ChatOps helps with simple are automation tasks, actionable notifications and transparency in distributed teams. There are some advance use cases like using ChatOps for deployments to telling a joke in the company message board to lighten the mood.

### 1. Creating A ChatOps

To build one Chatbot for ChatOps for your organization there are certain checklists that need to be marked before you start and there are :

***a) A chat platform***

There is no Chatbot without a Chat platform. The common one is [Slack](https://slack.com) but there are other ones like  [Flowdock](https://www.flowdock.com/). There are open-source ones like [Mattermost](https://mattermost.com/) which can to be self-hosted. Whatever the choice it should be accessible across web, mobile, and desktops to respond in time to actionable alerts etc..

***b). Choose/Build a chatbot***

Secondly, we'll need a chatbot. This is the foundation of your ChatOps so there a lot will go in for making your choice. It doesn't need to be smart like Tony Stark's assistant J.A.R.V.I.S it should just be able to accept commands and respond by initiating actions. This doesn't need to be overly complicated so far as it can get the job done.

If you don't have time to build one from scratch there are tons of open-source chatbots with adaptors to work with different chat platforms. They offer different functionalities and mostly come with different extensions to upgrade or improve the capabilities of your chatbots.

Some of these can be found [here](https://github.com/exAspArk/awesome-chatops).

***c). Integrate and automate workflows***

Once the chatbot is ready, we'll need to define daily workflows, the mundane and repetitive ones, that we need to be automated and integrate them. If you're building your chatbot from scratch these are features you have to design and code yourself. But if you're using an already developed one you can search for extensions or search ways to write custom scripts to work with your chatbot. Since most of the chatbots have APIs to integrate custom functionalities.

An example is [Hubot](https://hubot.github.com/), a chatbot developed at Github, which has [ways](https://hubot.github.com/docs/scripting/) extending your bots functionalities by writing `.cofee` or `.js` scripts.

One important aspect to consider when integrating all these extensions to automate workflows is authentication, authorization and audit. It's very important to not expose every part of your system to every user via the integrations. Same way not every user can restart production services same way you need to incorporate authorization via the chatbot.

***d). Structure to get everyone on board***

Finally design a structure to keep the chatbot conversations as organized as possible. Imagine a use case where by chatbot spams the marketing team about CPU usage of a service. This may be considered as spam thereby affecting company-wide adoption. To create a culture that buys into ChatOps proper grouping on conversations from the bot needs to be created and gradually features and functionalities are integrated into daily workflows per team/department needs.

Onboarding should be made as simple as possible with actionable alerts. For example if there's an alert that a service is down, the true value of chatbot should also provide a means to start the service from the chat platform. As much as possible onboarding should require company /organization sso flow so it doesn't take another sign up form before people can start using it.

### 2. Building A Sample ChatBot : AWS S3 and IAM Policy Monitoring with Slack Slash Commands

By putting the cart before the horse, let's tackle packing first it has to be simple to install no matter the environment, so we'll use a container to help abstract this. You can find `Dockerfile` files. Using [travis-ci](https://malike.github.io/Build-Release-Dockerize-Run-With-Travis-CI.html) we can automate the building and release of the bot to DockerHub.

Now we just want a bot to be able to do simple things, like running a `bash` or `shell` script from a bastion host. I've decided to use Java Spring for this, you can use any language you're comfortable with.

I've added two utility scripts as well to help test, one that lists  all "public" S3 buckets on AWS and the other lists all users without MFA. Both scripts are specific to AWS and use the `aws-cli`

Now the main part, integrating the bot to work with Slack. There's an HTTP endpoint `/slackbot` which we'll have to configure for our slackbot to receive request or better known as **Slash Commands**. **Slash Commands** are basically shortcuts for specific actions in Slack. Type a slash command in the message field, select enter to send, and that's it. You've performed a task in one simple step. In our case we'll want a slash command as simple as
`/slackbot ping UAT` to ping a UAT server we've already mapped to the term _"UAT"_ with response that UAT server is healthy. We can also use it to execute bash scripts, which I've tested with two custom bash scripts one to get list of public S3 buckets on AWS and the other to list users with MFA turned off on AWS.

Once we get the results of the bash scripts are generated we can push it to slack using [slack-cli](https://github.com/rockymadden/slack-cli). This will then send a successful callback using the slack API to the initiator on slack.

But to have actionable notifications we should be able to either disable the public policy of the S3 bucket or notify users to turn on MFA, or restart a service after a ping returns _UNHEALTHY HOST_.

This is pretty much a simple slack bot(in progress) but hopefully it will give you ideas to build upon this or start  a fresh one for your organizations. It has a simple workflow and will have actionable notifications so users can easily respond to alerts anywhere they are just by using Slack.

> Source code available on [Github](https://github.com/malike/slack-bot), if you'll like to help finish it.

<br>
<br>
### References

[https://docs.stackstorm.com/chatops/chatops.html](https://docs.stackstorm.com/chatops/chatops.html)