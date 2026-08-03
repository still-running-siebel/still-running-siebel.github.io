---
title: "Restarting a Siebel environment properly: order, waits, and the things that hang"
date: 2026-08-03
---

Restarting Siebel is one of those tasks that looks trivial until it isn't.
Stop the services, start them again - how hard can it be? Then a Siebel
Server sticks in *StopPending*, or comes up before the gateway is really
ready, or won't start at all because the last shutdown left worker
processes behind. What should take two minutes turns into twenty and an
RDP session.

Here's what "properly" actually involves.

## Order matters, both ways

Siebel services have a dependency order. The Gateway Name Server registers
the enterprise; the Siebel Server connects to it; the application
containers front the whole thing. Start them out of order and you get
services that come up, fail to connect, and sit there useless.

The reliable sequence on a typical estate is gateway first, then the
Tomcat application containers, then the Siebel Server. Shutdown is the
exact reverse: Siebel Server first, then containers, then gateway last.
On a multi-node environment the same tier across every host moves
together - all the containers, then all the Siebel Servers - rather than
node by node.

## "Running" is not "ready"

The trap that catches automation scripts: a Windows service reporting
*Running* only means the process launched, not that the software inside it
has finished initializing. Fire the next tier's start commands the instant
the previous tier reports Running and you race against a gateway that
hasn't finished coming up.

The fix is a deliberate settle pause between tiers - a few seconds after
the gateway, longer after the containers - before the next tier starts.
Not glamorous, but it's the difference between a restart that works every
time and one that works most of the time.

## The shutdown that doesn't finish

Sometimes a Siebel Server service won't stop cleanly, or stops but leaves
its worker processes - `siebmtsh.exe`, `siebmtshmw.exe`, `siebproc.exe`,
`siebprocmw.exe` - still running. The next start then fails, because those
orphaned processes are still holding on. The manual fix is to remote into
the host, open Task Manager, and kill them by hand before trying again.

This is worth detecting deliberately: if a Siebel Server service is
stopped but its worker processes are still present, they're orphans from a
failed shutdown and need clearing before the service will start.

## Settle times, hang thresholds, and disabled services

A few more things the careful admin accounts for:

- **Hang thresholds.** A service stuck in a pending state past a
  reasonable window isn't going to recover on its own - it needs the
  process terminated. But "reasonable" for a Siebel Server is longer than
  for a gateway, so the threshold has to be generous enough not to kill a
  service that's simply slow to start.
- **Disabled services.** Environments often have services installed but
  intentionally set to Disabled. A restart routine has to know to leave
  those alone rather than dutifully trying to start something that's off
  on purpose.
- **A record of what happened.** When a scheduled restart runs at 2am, you
  want a log the next morning showing exactly what stopped, what started,
  how long each took, and what - if anything - hung.

## Doing it repeatably

Once you've encoded all of this - the order, the settle waits, the hang
recovery, the orphan cleanup, the disabled-service handling - you don't
want to redo it by hand each time. That's exactly what we built [Siebel
Service Manager](/tools/siebel-service-manager/) to do: one command,
correct order across every host, with the waits and recovery built in.

Whether you use that or roll your own, the principles above are what
separate a Siebel restart that works every time from one that works most
of the time.
