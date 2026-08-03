---
title: "Egress Filtering Is the Control We Never Implemented"
date: 2026-08-01T12:24:15-04:00
draft: true
description: "We've known how to authenticate and authorize outbound access since the 2000s. Autonomous agents are what turn a hardening project we kept deferring into a containment requirement."
categories:
  - Technology
tags:
  - ai
  - security
  - zero trust
  - networking
  - egress
# For a header image, drop it in this folder and uncomment:
cover:
  image: "keep-out.jpg"
  alt: "Keep Out | Annapolis, MD, USA | 2025 | Damien DeVille"
  relative: true
---

On July 21, 2026, we learned that a model reached the open internet from an environment that was supposed to be sealed and launched a cyberattack against another company. On July 30, we learned that it happened again at a different lab. That lab's disclosure was careful to list the ways the two cases differed: it had caught the problem itself, its models had left through an open path rather than a novel exploit, and its newest model stopped once it worked out the environment was real.

I [wrote about the first incident]({{< relref "/technology/it-wasnt-air-gapped" >}}) on July 24 and won't revisit it here. The short version: the model didn't defeat containment; the containment wasn't there.

Every one of those distinctions is real, but the list dismisses a subtle reality. In both cases, a process could send packets destined for the real internet and no enforcement point applied policy to the traffic in a meaningful way. In Anthropic's case we know it left from inside a third party's evaluation network and carried out real attacks against companies that were never part of the test. Frankly in 2026, we know better and yet open egress access persists. Outbound internet access should be governed in the same way that we govern inbound access.

That is not an AI problem. It's the access problem, and it is considerably older than any model involved. Many of my colleagues and I were arguing for outbound filtering in the late nineties. The papers written about egress filtering then were largely about spoofing; the argument that a host should have to say where it was going, and be told no, took another decade to be written down. The egress controls we've deferred since then are the ones that now determine whether an agent's mistake becomes an incident or not.

I spent a good part of my career managing firewalls, trying (and mostly failing) to make egress filtering work in a way the business would tolerate. I can tell you exactly why the business refuses to accept filtering access from inside the network to the internet: it is hard, it breaks things people need, and it addresses a risk nobody feels.

Before the advent of autonomous agents, we believed we had time to respond to and contain an incident. That was perhaps debatable, but we believed that we could continue treating egress as a hardening project — something worth doing, eventually, maybe once the higher-priority work was finished. Autonomous systems change that equation. They aren't inherently malicious, and they aren't smarter than the people who ran the attacks by hand. But they are fast and they close the gap between initial alert and consequence. Frankly, it's impossible for humans to keep up against autonomous agents.

## How "inside the network" became a permission

Firewalls were built with the assumption that we were facing a threat arriving from outside the network perimeter and this assumption drove behavior. Inbound access got the default deny, the change control, the review board, the quarterly audit. Outbound got a permit-any rule, and a nagging sense for some of us that we should tighten it, but later.

Every time I tried to narrow egress rules in production and apply them to human-generated traffic, something broke. Sometimes what broke was genuinely business-critical, and the policy deserved to be modified, but more often the person affected simply had enough standing to get the security decision reversed, and that in large part is why permit-any outbound continues to exist. No one wants to fight that political battle, and no amount of engineering fixes it.

Network location became an authorization decision. A process that could route to the edge could talk to anything, not because it had proven what it was or established what it needed, but because of where it happened to be running. We all knew that we should be allowing access based on the user's identity but an IP address was the closest thing we had to an identity on the network. Network addressing, and location stood in for identity, and identity stood in for permission.

The design model had a name, castle and moat, and it was a coherent model, representative of what we understood about threats at the time. It was codified in the platforms we bought to make configuration convenient because nearly every network was already following this design principle. NetScreen named its zones _Trust and Untrust_, and the label did the work of a policy: traffic leaving the trusted side didn't need to justify itself. Cisco's PIX made it arithmetic, assigning every interface a _security level_ from 0 to 100, with inside at 100 and outside at 0, and _permitting traffic from a higher level to a lower one implicitly._ You wrote access lists to let things in. The number attached to the interface decided outbound access.

Internet Protocol and the networks built on it were not designed for a hostile environment. The organizing principles were interoperability and, honestly, altruism: get networks that had never met to exchange packets, and assume that both ends wanted the exchange to succeed. Nothing in the core protocols asks a sender to establish who it is. Nobody had built a fully distributed interconnected network at that scale before, which meant there was no operational experience to draw on and no accumulated record of failure to learn from. We had guidance from plenty of places. What we didn't have was precedent.

Default-allow egress descends directly from all of this. It's the legacy of the network's founding assumption, still running in production, and every security control we have bolted on since has been a retrofit. That is the actual problem: a reasonable assumption outlived the conditions that made it reasonable, but as defenders, we didn't adapt.

In 2010, Eric Hutchins, Michael Cloppert, and Rohan Amin published Lockheed Martin's intrusion kill chain: seven phases running from reconnaissance through weaponization, delivery, exploitation, installation, command and control, and actions on objectives. On the sixth phase the paper is blunt about what an adversary requires. _"The adversary must establish a command channel to the compromised host to be able to direct the actions, also known as 'beaconing.'"_ The defensive claim attached to the model is equally blunt. _"The defender needs to stop the chain at just one phase to stop the intrusion."_

Read those two sentences next to each other. The adversary's control channel is an _outbound connection_, and interrupting a single phase is sufficient to end the intrusion. Egress control has been a documented way to break the chain for sixteen years. And yet, open access to the internet from "inside" the network persists to this day.

The same year (September of 2010), John Kindervag published Forrester's Zero Trust model, starting from the position that network location is not authorization. Most organizations have spent the sixteen years since applying that principle in precisely one direction (users accessing applications). Application access got rebuilt around identity, device posture, and an explicit decision per request. Users stopped being trusted because they were on the corporate LAN. Meanwhile the permit-any rule pointing outward survived the entire program intact, in most cases untouched by it. We accepted that location proves nothing about traffic accessing our apps and continued treating it as sufficient for traffic destined to the internet.

## What autonomy actually changed

Anthropic is explicit that the models in its evaluations "compromised the impacted organizations' infrastructure using basic techniques," including weak passwords, unauthenticated endpoints, and SQL injection. There is nothing in that list a competent attacker couldn't have done in 2005, and nothing that argues the models were unusually clever. What was different is that a machine orchestrated the attack lifecycle, iterating through the phases and adapting when one didn't work, with no human making the decisions.

Automated tooling is old news. Scanners have always been fast, exploit kits have always been packaged, and beaconing was automated long before any of this. But the sequence still had a person in it. Someone chose the target list, looked at what came back, and decided whether to proceed. Reconnaissance ran at machine speed and then waited for a human to read the output. In Anthropic's third incident, a research model scanned roughly nine thousand targets, selected one, compromised it through SQL injection, and exposed credentials. The scan wasn't the novel part. What was novel was that there was no human in the loop between the scan and the exploitation.

In one incident, Claude Mythos 5 built a malicious Python package and published it to PyPI, where it stayed available for about an hour before PyPI's own security systems removed it. In that hour, fifteen real systems downloaded and ran it, including a scanner belonging to a security company, where the hidden code exfiltrated that company's credentials. A published package is not a session you can kill. The takedown was fast, automated, and still too late, because the package had already been downloaded and executed by third parties who had no relationship with the evaluation. Publishing a malicious package to a registry could have been prevented with appropriate egress inspection (as well as ingress inspection at the package registry and at the victims).

None of the three affected organizations detected the activity. Anthropic found the evidence in its own evaluation transcripts and reached out to all three, which means every one of them learned it had been compromised from the company whose model did it. The blast radius wasn't contained by detection and response, because no analyst saw anything. It was contained by what the models could reach which was effectively everything exposed to the internet. Whether detection and response can be automated fast enough to close the gap is a real question, and a different post.

In the first incident, Opus 4.7 "recognized that it had reached production systems but continued its attack." Mythos worked out that the environment was real but convinced itself that its reasoning was unsound and kept going anyway. A newer, unnamed, model was the only one where internal guardrails caused it to cease the attack. Whatever you conclude about model judgment from a sample that small, and Anthropic is careful to say you should conclude very little, we should be clear that relying on guardrails built into a system is a strategy designed to fail. If the only barrier between an autonomous process and somebody else's production database is that process's own situational awareness, there is no barrier. The control must be independent of the agent and must be able to inspect either traffic or api calls, or both to be effective.

## Three things have to be true on the way out

Putting the control in the data or API path means answering three questions about every outbound connection an agent opens: what is asking, where is it permitted to go, and who can see the answer. None of this is new theory. Zero Trust is about removing implicit trust, and outbound is where the implicit trust was left in place. Most organizations already own the technology required for Zero Trust: identity solutions, firewalls, and advanced inspection capabilities. The problem lies in policy design, staffing, and enforcement on egress internet access.

### The agent needs an identity of its own

Most agents today inherit their credentials. They run as a service account, hold a long-lived API key, borrow the identity of the human who launched them, or simply occupy a subnet that grants them access. Every one of those substitutes something around the agent for the identity of the agent itself. That is what an IP address was doing in the old model, and it fails for the same reason: anything that lands in the same place, or gets hold of the same key, inherits the same authority. An identity the application layer verifies but the enforcement point cannot see contributes nothing to an egress decision.

### Egress is an explicit allow list, or it isn't a policy

The policy is default deny outbound, with an explicit allow list bound to the agent's identity and expressed in destinations and methods rather than addresses. Anyone who has attempted egress filtering for a human population has the objection ready: you cannot enumerate the web. That is true, and it settles far less than it appears. Browsing is one subset of what an environment grants when inside-to-outside is open. What most of us actually permitted was any destination, any protocol, and any port. We have better ways to inspect traffic than we did when firewalls were first designed. You can permit any address and still inspect what crosses: DNS filtering, URL categorization, anti-malware scanning in the stream, and session-level threat prevention are all mature. This maturity means that "allow the web" and "allow anything" stopped being the same sentence a long time ago.

With this in mind, writing egress policy for agents is arguably easier than writing policy for human users. An agent has a declared job, and its legitimate destinations are usually limited to a few stable endpoints, known at design time and declared during deployment. What requires categorical policy for a person is frequently a literal list for a workload. That difference makes egress policy more manageable for agents than for people. An agent shouldn't ship to production with its destination set undefined. Codify egress requirements in the deployment and review them alongside the rest of the agent's authority as part of your CI/CD pipeline. And when policy blocks a connection, log and alert on it. A silent denial becomes a mystery outage; a visible one tells you whether the policy or the agent needs to change.

### Visibility has to exist wherever the agent executes

An enforcement point that only sees the TCP/HTTP state of a connection cannot distinguish an agent retrieving documentation from one exfiltrating credentials. TLS becomes the limiting factor in most of our modern traffic. When traffic is encrypted with TLS, the payload is not visible for inspection and makes our enforcement points effectively blind. TLS decryption restores that visibility and egress control points have the required inspection capabilities. Without it, the policy generally enforces on the classic source, destination, and port/protocol decision capabilities of early technologies.

Anthropic's incidents were caught by their own monitoring rather than by any of the affected organizations, which is a fair demonstration of what egress logging is worth. At Irregular, the third-party evaluation partner, an apparently lax egress policy allowed evaluation containers direct internet access, and neither party knew until monitoring surfaced it. When you delegate execution to someone else, you inherit their egress posture. That is a thing to verify explicitly and test, not to assume from a contract.

## The parts that stay hard

In my experience TLS decryption adoption tends to be low in the enterprise, precisely because implementation presents so many challenges. Additionally, network based TLS decryption is necessary but not sufficient. Certificate pinning breaks decryption by design, and the applications that pin tend to be the ones you cannot simply refuse to run. Mutually authenticated sessions won't survive a device decrypting and re-encrypting the flow. Additionally, for reasons that are legal in some jurisdictions and simply decent in others there are some types of traffic that shouldn't be decrypted, which means an exception list, and every exception is a documented gap somebody has to justify. For these reasons, defense in depth still applies and controls and enforcement at the host level are critical.

Policy maintenance is the cost people consistently underestimate, because it never ends. Allow lists rot: a vendor moves to a different CDN, an API adds an endpoint, a library starts fetching telemetry from somewhere new, and a control that was accurate in March quietly becomes an outage in June. What determines whether the policy survives is not its initial quality but our ability to keep it current. Egress policy fails at the speed of its change process.

Then there is the case my argument handles least well. Some agents genuinely need the open web. A research agent, a crawler, an assistant asked to go find out about a company: their destination set is unbounded by design, and the property that made agent allow lists tractable is exactly the property they lack. An allow list will not help here. The answer is categorical inspection of the kind human traffic should be getting, and, more importantly, a hard separation between the identity that browses and the identity that holds anything worth stealing. An agent that reads the open internet should not also have the credentials or network path to your production database. If a workflow needs both, separate the browsing task from the production step and pass only the necessary data between them.

Third-party execution turns enforcement into verification. You cannot install your controls in a provider's environment, so its egress policy becomes something you have to verify: ask for evidence, test it where you can, and treat any gaps as part of your own exposure. The Anthropic–Irregular incident is a reminder that a contract saying an environment is isolated is not evidence that it is.

Expect the first policy to need tuning: it will allow some traffic it should block and block some traffic it should allow. Start with agents whose jobs are narrow and whose breakage is cheap. Use the denials and exceptions that follow to tune the policy before extending it to broader, more variable workloads. Stage your rollouts, learn from the challenges and policy exceptions, and iterate. The continual feedback loop will prevent you from delaying projects indefinitely.

## The same control catches everything else

An enforcement point that requires identity, consults a policy, and inspects what crosses does not care what is on the sending end. It is indifferent to whether the process holding the connection is a model, a dropper, a build agent, or a browser extension nobody remembers installing. The kill chain reasoning from 2010 applies unchanged: the outbound channel is a phase, and breaking a phase ends the intrusion, whatever occupies the other phases.

So the same work that constrains an agent constrains ransomware, which has to reach its operator for keys and instructions and, in the double-extortion pattern that has been standard for years now, has to move the data out before it encrypts anything. It constrains infostealers, whose entire purpose is a single successful outbound transfer.

Security teams have been asking for funding for this since the early 2000s and losing, because the ask was to fund a control against hypothetical harm at a concrete cost measured in broken applications and angry engineers. That argument failed on its merits, over and over. It is winning now, not because we found better words for it, but because there is finally an executive asking a pointed question about what the agents are allowed to do.

So take these examples into the budgeting conversations. Make the case for funding as an urgent need with real business risk, then after you have the money scope the work widely covering more than agents, because an enforcement architecture built to cover only agent traffic is the same architecture you will need for everything else. Building it twice would be the expensive mistake.

## The question worth asking

Both labs have now put on the record how their models reached the internet, and it was Anthropic that drew the contrast: OpenAI's models "exploited a novel vulnerability to escape isolation," while the Claude models "accessed the internet via an open path." Both descriptions are accurate, but neither address the fundamental problem of open egress access.

I would rather hear a security team ask the more important question: why is internet access at the edge unrestricted? Why could a process running a capture-the-flag exercise open a connection to an arbitrary host on the public internet without establishing what it was, without a policy deciding whether that destination was in scope, and without anything in the path recording the answer? That question doesn't depend on which model it was, which lab ran it, or how clever the exploit turned out to be.

Anthropic deserves credit for answering it directly. "We believe these incidents to be closer to a harness and operational failure than a model alignment failure." That is the finding. Most of the attention went to the adjacent question of what the models believed about their environment, which is genuinely more interesting intellectually but considerably less useful pragmatically.

The uncomfortable thing about an operational failure is that it implicates people who can do something about it. Model alignment is a research problem belonging to somebody else. Egress is ours, and it has been sitting in the backlog since before either of these companies existed.

The design hasn't changed. Identity, an explicit decision, and something in the path that can see and record what happened are fundamental requirements. What changed is that the interval between a mistake and its consequences has closed to virtually zero, and we no longer get to spend it deciding whether this is the year we finally do the work. We don't need a new control. We need to deploy the one we've been describing to each other since the early 2000s, before the next thing that reaches the internet from inside a network we run turns out not to be running an exercise.

## Sources and further reading

- [Anthropic: Investigating three real-world incidents in our cybersecurity evaluations (July 30, 2026)](https://www.anthropic.com/news/investigating-incidents-cybersecurity-evals)
- [Eric M. Hutchins, Michael J. Cloppert, and Rohan M. Amin: Intelligence-Driven Computer Network Defense Informed by Analysis of Adversary Campaigns and Intrusion Kill Chains (Lockheed Martin, 2010)](https://sustainability.lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf)
- [John Kindervag: No More Chewy Centers: Introducing the Zero Trust Model of Information Security (Forrester Research, September 14, 2010)](https://media.paloaltonetworks.com/documents/Forrester-No-More-Chewy-Centers.pdf)
- [OpenAI and Hugging Face partner to address security incident during model evaluation](https://openai.com/index/hugging-face-model-evaluation-security-incident/)
- [Hugging Face: Security incident disclosure — July 2026](https://huggingface.co/blog/security-incident-july-2026)
