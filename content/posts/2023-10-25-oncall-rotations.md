---
title: "Oncall in product teams"
description: "Oncall in product teams"
tags: [softwaredevelopment]
comments: true
share: true
cover_image: ''
---

I have been oncall for as long as I can remember being in the industry, so far for every organisation I have been part of. Different things have worked at different phases of the organisation and the teams priorities. I thought I would put down some notes over things which I have seen have worked well.

# Why do we need to have oncall rotations?

Because simply put, it would help people not burn out of only being oncall unofficially over time, as they get pulled into specifics of systems which they are more aware of.

Having a proper cycle which cycles through all engineers in the team uniformly, will also allow the teams knowledge and toil being distributed on what does the team get paged more often.

While I was at Gojek, I would recommend graduate engineers joining the team(and it was the same that I recommended to the new graduate engineers in my team), to always shadow while doing production support, as it would be a very good avenue to get onboarded to the services which the team would own and have touchpoints for.

Further more, it reduces the bus factor, as knowledge is then forced in a way to be distributed via docs, debugging problems, runbooks over time in the team.

# Communication Channels

To organise comms during an incident, ideally, have channels, where people can update the status of incidents and talk/discuss on what are they doing.

Also is helpful, to have channels, where you run urgent cares for changes like deployment/config changes to help keep track of what changed for folks to have a look in the production systems, in case they have an incident and wanted to see what were the changes that went inside the system.

# Oncall is for whom

Apart from the usual pattern of engineers of the product teams who are oncall, potentially it's also very useful in situations for having senior engineers who have a lot of cross team information to be oncall along with leaders from the teams, as it helps again in distributing the load of oncall.

# Getting paged too often

This is another post in itself, but reliability and product development pace need not be exclusive from each other. Change in essense might be the simplest way to introduce instability in the system, but that's the nature of product development and preventing software rot.

Change is inevitable in the system, so the idea is to always make change, easy to make and easy to revert back as much as possible.

If the team is getting paged more often, the ideal path is to focus on preventing that fire first to prevent people from burning out, but balance is key.

More Reliability comes with a cost.

There are also some parts of the system, which a specific person might have more knowledge due to tribal knowledge possibly or simply because they have built it or spent huge amounts of time with it, and it could end up being that the person then gets paged more in-evitably because of that, this is a symptom to fix to distribute the work of oncall load on the person.

# Primary and Secondary

Layering your oncall is helpful in case, there is a miss by the primary to respond to a page, which gets auto escalated to the layer above. If your team can follow the sun pattern for oncall, that's potentially the best of options to have, to provide coverage for the team.

# Oncall as part of SRE/Operations teams

There's not a fixed set of checklists which will work for every team, but on a general note, reducing toil for oncall and having a rotation is a good place to start with and the iterate as we go.

I have also [talked about how first responders for teams](https://www.tasdikrahman.com/2022/02/21/evolution-of-support-for-infrastructure-teams/) requests also is ideally good to be spread across the team over a fixed rotation cycle.

# Ending notes

Oncall is a challenging problem but inevitable for teams, as long as you are providing something to the users. Heroism in oncall, along with toil management will be something which you will have to manage in any team nevertheless, as long as you are not completely ignoring these aspects, you are progressing forward with having a better oncall.

There's a lot of good literature around the internet on how teams should run oncall, and I would highly recommend reading them. As always, I end up learning something new when I read how teams are operating and learn from them.
