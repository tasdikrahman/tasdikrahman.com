---
title: "KubeCon EU 2025, My experiences attending and speaking at it"
description: "Reflections on attending and speaking at KubeCon EU 2025"
tags: [conferences, kubecon]
comments: true
share: true
cover_image: ''
---

Back in 2017, I attended my first KubeCon in Austin, which had around 1,500 attendees. Fast forward to 2025, KubeCon EU in London saw over 12,000 participants—a testament to how much the Kubernetes community and ecosystem have grown and evolved over the years.

This was my 3rd kubecon, and the first time speaking at KubeCon, and it was an incredible experience!

# Takeaways

## Conference Highlights

- Attended a lightning talk on [Kanister](https://github.com/kanisterio/kanister), a framework for application-level data management on Kubernetes.
- I really enjoed the talk on Kubernetes CRD design for long-term evolution. Key takeaways:
  - Design CRDs to evolve gracefully and avoid unnecessary version upgrades.
  - Prefer copying and adapting external API types rather than embedding them, to prevent breaking changes.
  - Use reusable types like `metav1.TypeMeta`, `ObjectMeta`, and `Conditions` where appropriate.
  - Pay attention to OpenAPI specs, deprecation policies, and CRD markers (use tools like Kubernetes API Linter).
  - Design GitOps-friendly APIs by using list types with ownership keys.
  - Choose clear, descriptive field names and avoid generic terms or camelcase concatenation.
  - Take inspiration from upstream APIs and gather feedback from users.
  - Be mindful of challenges like API removals ([example issue](https://github.com/kubernetes-sigs/cluster-api/issues/11920)).
- Nick Young has some amazing suggestions on Avoiding common Kubernetes controller mistakes. 
  - suggestions on using the Patch instead of Update to send only the changes, instead of the whole object.
  - using a framework like controller-runtime, krt, StateDB instead of hand rolling something.
  - versioning your CRD. 
  - Thinking how users will use your CRD API, making sure you spend good amounts of time in thinking about the API design.
  - [slides](https://static.sched.com/hosted_files/kccnceu2025/53/Don%27t%20write%20controllers%20like%20Charlie%20Don%27t%20does_%20avoiding%20common%20Kubernetes%20controller%20mistakes.pptx.pdf)
  - [YouTube recording for 2024 talk of Nicks on Avoiding common CRD Design errors](https://www.youtube.com/watch?v=pMnuso9KGtU)

## My Presentation

I had the opportunity to present my talk at KubeCon EU 2025: "Resilient Multi-Cloud Strategies: Harnessing Kubernetes, Cluster API, and Cell-Based architecture" along with my colleague Javier Mosquera

<iframe class="speakerdeck-iframe" style="border: 0px; background: rgba(0, 0, 0, 0.1) padding-box; margin: 0px; padding: 0px; border-radius: 6px; box-shadow: rgba(0, 0, 0, 0.2) 0px 5px 40px; width: 100%; height: auto; aspect-ratio: 560 / 315;" frameborder="0" src="https://speakerdeck.com/player/353687d1b5e84e6494945dcfa74816e3?slide=2" title="Resilient Multi-Cloud Strategies: Harnessing Kubernetes, Cluster API and Cell-Based Architecture" allowfullscreen="true"></iframe>


<iframe width="560" height="315" src="https://www.youtube.com/embed/4DjydLH21nM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Here are the slides and the recording links.

- [SpeakerDeck slides](https://speakerdeck.com/tasdikrahman/resilient-multi-cloud-strategies-harnessing-kubernetes-cluster-api-and-cell-based-architecture)
- [YouTube recording](https://www.youtube.com/watch?v=4DjydLH21nM)


Here are a couple of pictures from the conference:

<center><img src="/content/images/2025/07/kubecon-image1.jpeg"></center>
<center><img src="/content/images/2025/07/kubecon-image2.jpeg"></center>
<center><img src="/content/images/2025/07/kubecon-image3.jpeg"></center>
<center><img src="/content/images/2025/07/kubecon-image4.jpeg"></center>
<center><img src="/content/images/2025/07/kubecon-image5.jpeg"></center>

## Ending notes

Giving this talk was a truly memorable experience. I thoroughly enjoyed sharing our journey, answering questions, and connecting with so many passionate members of the Kubernetes community.

It's exciting to see the community grow each year, if I compare the kubecon which I attended last year in Paris. Looking forward to attending the next one!

Until next time!
