---
title: "Evolution of support for infrastructure teams"
description: "Evolution of support for infrastructure teams"
tags: ["engineering-productivity"]
comments: true
share: true
cover_image: '/content/images/2022/02/Webervogelnst_Auoblodge.jpg'
---

## Context

As time has progressed, I have been part of teams of different sizes in terms of org size as well as the team sizes which I have been part of. This post is a conglomeration of the ideas I have picked up, things which have worked out/which haven't and mental models developed as being a part of such infrastructure teams and growing with them.

Another thing noticed over time, would be the amount of adhoc work being slightly higher than other engineering teams out there, hence the difference in structure on how to handle it as we will see over the course of this post.

This post condenses the ideas presented here.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A little late to the party but here are a few ways, infrastructure teams could function during the organization growth phases and different engineering team sizes over time (1/n) 🧵 <a href="https://t.co/wUC7ujyaU3">https://t.co/wUC7ujyaU3</a></p>&mdash; Tasdik Rahman (@tasdikrahman) <a href="https://twitter.com/tasdikrahman/status/1495881281745440772?ref_src=twsrc%5Etfw">February 21, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Before we start

I am deliberately not putting out number of engineers here to fit a case because you will start feeling the toil and support request influx eventually, leading you to adopt and try out different models of support over time as this also depends on things apart from team size.

## Initial days

For starters, for a new infrastructure team, you will notice the team getting adhoc requests on DM's/Slack channels/phone calls too maybe and other mediums of communication which you use. This is workable only up till the point where the respective infrastructure team folks don't get tired of responding to requests on the above mediums, which is bound to happen, as the distribution is random at best.

But if we look at it from the engineering teams perspective this is normal since there is no clear format/expectation set, which is what the infrastructure team needs to first do/would already be doing in some ways already. For the former part, the said person, receiving a request may then point it to the other team members on the standup that foo request was made and discuss it with others or if the task was too small maybe they already implemented it already before the next day.

The first thing which we can do in such a case would be, to ask folks requesting support to request for it on a respective slack channel and mark it as feature/bug fix/emergency request/query. If there's a ticketing system/kanban board for the infra team, ask them to create the ticket/card there and let someone from the infra team(possibly the manager/lead) route such requests to the infra team members depending on their prio and availability.

This lets you solve a couple of things, the prio problem, which then gets offloaded to a single person rather than the whole team, and the problem of discovery of what were the support requests which the team got/solved.

One thing to note here that, there is most likely no incentive for the infra team member to solve tickets, hence I have only seen this model work when there is a gentle bump each time from someone who is keeping track of the prio of the tickets and their status.

## As the team size grows

The team sizes grow, along with members in your team also growing, but the toil may also have increased over time, which could be a symptom of a couple of things. If you have been tracking the support requests which you have gotten, by this time you would have noticed what were the kind of requests which you most get frequently and how long do such requests take. The idea is to see what requests does your team get and how much time are you spending on such things for starters. After this, you would have a decent picture of requests/problems which your team is getting and you can focus on reducing the influx or requests by either.

Documentation to fix knowledge gaps for the team, helping them gain context and close off queries quickly. This also helps distribute knowledge inside the team. A general rule of thumb could be if you do it twice, write it down.

Automation being added to do repeated tasks, which have fairly predictable branch off's from the happy paths, which can be handled by the automation for those cases when it deviates from the happy path.

This can be the next step of documentation, which supplements the knowledge by putting the operational knowledge in a tool. But remember that prematurely writing the automation is not any better than not having any automation.

All code is liability, so be careful on what you write the automation in, it should be something which can be maintained by at least a couple of people in the team, to help de-risk the automation becoming a fragile codebase which the team is afraid to touch.

## Further growth and division into sub-teams

After a point of growth in your team, you will start noticing the larger infrastructure team to branch off and have separate sub teams for specific interest groups which the team works on. For example, observability, release engg, infrastructure, security, dev experience etc start getting formed as the team's requirement for more people to focus on specific problems arise from the growth in the engineering team.

At this point, the original problem of toil still remains and will remain for each sub-team to handle, the only difference here will be that, each ticket will then have a specific team to get routed to.

It can also become a point of contention on what parts are to be owned by which sub-team, hence preferably, the ownership of specific components should be pre-decided for each sub-team, apart from the obvious ownership of components. This helps resolve issues/confusion arising from who would own a certain ticket, the back and forth hence causing a possible fallthrough in SLA.

On SLA's, the teams would need to decide on the expectations on what is a good turn around time for a ticket requested, based on the priority on which it has been raised.

There will be cases when requests raised on priority P0/P1 might be P2/P3. A good rule of thumb is to have anything affecting affecting customers/bleeding money/security incident as P0 and needs immediate attention. The other prio groups can be discussed and decided upon. SLAs/tracking of requests/routing of tickets being at least have been addressed in this first pass, let's look at SLA's.

You might notice after a certain org engg team size and each sub-team having it's own sub section in the Quarters planning doc, that you might be falling short on SLA's for the support tickets turn around time.

It could be that there is automation missing, docs missing, or simply could be that the amount of support requests is simply big enough for the individual sub teams to be routed just via the TL/manager.

As they would also be distributing the request over to someone initially depending on their familiarity with who understands which stacks, but ideally this distribution in the sub team should be random at best as everyone should be distributed these requests equally.

If the request workload is still increasing, it might be time to introduce rotations for support inside the sub teams, having dedicated people from the sub teams to look and work on issues for the sprints duration and then rotating away afterwards

This helps in a couple of ways, for starters, helps prevent burnout for the team members as support becomes democratized within team. Since not everyone gets support tickets by default, the others can focus on strategic problems to be solved. The idea should be that, apart from toil work. There should be ample amount of time left for strategic work, which helps reduce toil work and improves dev experience, helping create good abstractions

## Further improving productivity inside the sub-teams

You could further go into having dedicated folks for support for each teams to having dedicated folks for handling support for infrastructure sub-team engineering support. This will not immediately start showing any positive impact as with every new member in the team, it takes some time for the impact to come across, but having an L1 support structure pays back in time for the overall team

This L1 support team can be the first line of defense for anything related to infrastructure queries for all the sub-teams inside

Over time with pairing and knowledge transfers, the idea would be for the L1 support team to handle some level of queries/providing solutions with some/no amount of support from the sub-teams. As time goes, these L1 folks would have enough context to even start working inside the specific sub-teams and absorbed there if the teams would like to, creating a funnel of engineers joining the team via L1 -> sub-team member and so on and so forth.

At this stage your infrastructure team could very well be said to be in a mature stage. And would have been able to keep support request SLAs to a favorable number along with working on strategic things.

## Final thoughts

As always, the main thing to note here would be reduction of toil as time goes by and keeping track of how much time is one spending on tickets/support requests, everyother solutioning follows this quest of reducing toil and a side outcome as part of it.

Would love to hear, how you folks handle support requests and adhoc work in your org

## References

Cover image by Harald Süpfle, CC BY-SA 2.5 <https://creativecommons.org/licenses/by-sa/2.5>, via Wikimedia Commons, https://commons.wikimedia.org/wiki/File:Webervogelnst_Auoblodge.JPG
