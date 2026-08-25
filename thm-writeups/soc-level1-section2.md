# SOC L1: Alert Triage and Alert Reporting

TryHackMe rooms: [SOC L1 Alert Triage](https://tryhackme.com/room/socl1alerttriage) and [SOC L1 Alert Reporting](https://tryhackme.com/room/socl1alertreporting)

These two rooms sit under the SOC Level 1 path and work as a pair. Alert Triage covers how to actually process an alert once it lands in the queue, and Alert Reporting picks up from there, covering how to write up the alert once a verdict has been reached. Both use TryHackMe's own SIEM interface rather than a real product, but the workflow maps onto how a SOC queue works in practice.

## The alert queue

The SIEM shows a running list of alerts with time, name, severity, status, verdict, and assignee. At the start of the room the queue looks like this, with most alerts still awaiting action:

<img width="1348" height="598" alt="soc19" src="https://github.com/user-attachments/assets/e9165ed8-ab89-4631-bd13-df707689fb35" />


Five alerts, ranging from Low to Critical severity. Two are already closed with a verdict, three are unassigned and waiting.

## Triage workflow

The room lays out a reference flow for how an L1 analyst is expected to work an alert from start to finish:

<img width="1321" height="588" alt="soc21" src="https://github.com/user-attachments/assets/187ed502-1aae-4450-8da1-123ded258aa4" />


The steps in order:

- Prioritise the alerts and pick the first one to work
- Assign the alert to yourself
- Move it to "In Progress"
- Read the alert's name and description
- Note the relevant fields: host, IP, user, and so on
- Check if a workbook exists for this alert type. If it does, follow the documented steps. If not, investigate directly in the SIEM
- Reach a verdict: true positive or false positive
- Decide whether the alert needs escalating to L2
- Add an analyst comment explaining the reasoning
- Close the alert (or escalate it, then loop back to the next alert in the queue)

This is the core of the room: it is less about technical detection and more about following a repeatable process so nothing gets missed and every decision is documented.

## Working the queue

After going through each alert in the queue and assigning myself, investigating, and setting a verdict, the queue looked like this:
<img width="1311" height="639" alt="soc23" src="https://github.com/user-attachments/assets/a6f8e1bb-ce49-4ed0-88f6-e47ae3e8c064" />

Every alert now has a status of Closed and a verdict attached. Two came back true positive (the double-extension file creation and the bruteforce attempt from an external source), and the rest were closed as false positives after investigation. Assignee is filled in for each one, which matters for accountability in a real queue.

## Phishing alert example

One of the alerts worked in the room was an email flagged as phishing after delivery:

<img width="1329" height="464" alt="soc22" src="https://github.com/user-attachments/assets/58e66dfa-1d1d-4d66-b120-89738fe10e92" />


The email impersonated Microsoft support and used a spoofed sender address, with SPF and DKIM both failing. The subject line played on urgency ("600% price increase") to get the recipient to open an attached RAR file, which is a fairly standard social engineering setup: fake authority, fake urgency, and a payload disguised as something the recipient would feel obligated to check. It was marked in progress with a true positive verdict pending final review.

## Escalation example

Not every alert stops at L1. One alert in the queue involved a spike of domain discovery commands, which turned out to be evidence of a reverse shell and an attempted privilege escalation:

<img width="1247" height="596" alt="soc20" src="https://github.com/user-attachments/assets/dc247b7e-24c1-4899-9241-7010ab86af82" />


This one got reassigned to an L2 analyst, kept as a true positive, and closed out with a comment explaining what was found and why it needed to go up a level: the activity was tied to the NT AUTHORITY\SYSTEM account on a Windows Server 2012 host, which is well outside what a normal process should be doing and is a strong indicator of a compromised system. Escalating with clear reasoning attached is what the earlier flow chart calls for when the analyst doesn't have enough context or authority to close something out on their own.

## Takeaways

The two rooms together cover the full lifecycle of an alert: pick it up, investigate it, decide if it's real, document the decision, and either close it or hand it off. What stood out most was how much of the job is process and documentation rather than pure technical analysis. A correct verdict with no comment or a badly assigned alert is still a problem in a real SOC, since other analysts and auditors rely on that trail. Writing a clear analyst comment that explains the "why" behind a verdict, not just the "what," is the habit this room is really trying to build.

# SOC Metrics and Objectives — Notes

Part of the SOC Level 1 path on TryHackMe. This room is about how a SOC actually measures whether it's doing its job, not just theory for theory's sake. Writing this up mostly so I remember it later and so I have something to point to when I'm prepping for interviews.

## Why metrics even matter here

A SOC's whole job is to protect confidentiality, integrity, and availability. But "we protect the company" isn't something you can put on a dashboard. You need numbers that tell you if the team is actually catching things fast enough, or if alerts are just piling up while real threats slip through.

The room makes a point early on that zero alerts for a month is not a good sign. It usually means detection is broken, not that nothing happened. That reframed how I think about alert volume — low numbers aren't automatically a win.

## The core detection and response metrics

These are the four that come up constantly in SOC work:

- **MTTD (Mean Time to Detect)** — how long between something bad happening and the SOC actually noticing it.
- **MTTA (Mean Time to Acknowledge)** — how long between the alert firing and an analyst picking it up.
- **MTTR (Mean Time to Respond/Resolve)** — how long from acknowledgment to the incident actually being contained or closed out.
- **False Positive Rate** — the percentage of alerts that turn out to be noise instead of real threats.

The order matters. Detection has to happen before acknowledgment, which has to happen before response. If any one of these is slow, it drags the whole timeline out, and that timeline is usually what's in the SLA with the client or the business.

## False Positive Rate is the one that quietly kills teams

This was the part that stuck with me the most. A high false positive rate doesn't just waste time — it trains analysts to stop trusting alerts. Once that happens, real threats start getting triaged with the same lazy attention as the noise, and that's how things get missed.

The room frames 80% false positives as a rough ceiling you don't want to cross. Past that point the alert pipeline is basically broken and needs tuning, not just more analysts.

## SLA ties it all together

An SLA (Service Level Agreement) is usually where these metrics get formalized — especially in managed SOC setups where you're reporting numbers back to a client. If your SLA says critical alerts get acknowledged within an hour, MTTA is the number that proves whether you're holding up your end.

Working hours matter here too. If the team works 8/5 and a critical alert lands over the weekend, the clock on acknowledgment effectively doesn't start until the next business day. That's an important detail for on-call planning — coverage gaps show up directly in your metrics whether you like it or not.

## In-house vs managed SOC

Quick distinction worth remembering:

- **In-house SOC** — the security team is part of the organization itself. Metrics stay internal, used for continuous improvement.
- **Managed SOC (MSSP)** — a third party runs security operations for the client. Metrics become contractual — they're literally what the client is paying for and checking against.

This changes the stakes on getting metrics right. In an MSSP, a bad MTTR isn't just an internal problem, it can be a contract problem.

## What an L1 analyst can actually control

As an L1, you're not setting SOC-wide policy, but you do influence these numbers directly:

- Acknowledging alerts promptly instead of letting the queue sit
- Documenting clearly so escalation to L2 doesn't add unnecessary delay
- Flagging patterns of false positives instead of just dismissing them one by one — that feedback is what gets rules tuned
- Not treating alert triage as a checkbox exercise, since sloppy triage inflates both MTTR and the false positive rate over time

## Takeaway

Metrics aren't just something management looks at in a quarterly review. They're a feedback loop. Slow detection, slow acknowledgment, or a noisy alert pipeline all show up as numbers, and those numbers are what get used to justify more staff, better tooling, or process changes. Understanding them as an L1 means understanding what you're actually being measured against, not just what tickets you're closing.
