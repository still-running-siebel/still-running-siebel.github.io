---
title: "Why won't my Siebel Server service start? Orphaned siebmtsh and siebproc processes"
date: 2026-08-03
---

You go to start the Siebel Server service. It doesn't come up. The Windows
service sits at *Stopped*, or flicks to *Start Pending* and falls back.
Nothing in the obvious places explains it. You restart the host, and now
it works - which is deeply unsatisfying, because you still don't know why.

Nine times out of ten, the cause is the same: **leftover worker processes
from a shutdown that didn't finish cleanly.**

## What's actually happening

A running Siebel Server isn't one process. The service itself
(`siebsvc.exe`) supervises a family of multithreaded server processes that
do the real work - the ones you see in Task Manager as:

- `siebmtsh.exe`
- `siebmtshmw.exe`
- `siebproc.exe`
- `siebprocmw.exe`

When the Siebel Server stops normally, those child processes are torn down
with it. But when a shutdown is abrupt - a service timeout, a kill, a host
that went down hard, a hung component that never released - some of those
worker processes can survive. The service reports Stopped; the workers are
still running underneath.

When you then try to start the service, it collides with the survivors -
ports still held, shared memory segments still mapped, resources still
locked - and the start fails. From the outside it looks inexplicable,
because the service you're starting looks fully stopped. The problem is the
processes the service *doesn't* show you.

## How to confirm it

Before doing anything, check whether the orphans are there. On the affected
host:

```powershell
Get-Process siebmtsh, siebmtshmw, siebproc, siebprocmw -ErrorAction SilentlyContinue
```

If the Siebel Server service is stopped but that command returns
processes, you've found your answer. Those are orphans - there should be no
Siebel worker processes running while the service is down.

## The fix

Terminate the orphaned workers, then start the service:

```powershell
Get-Process siebmtsh, siebmtshmw, siebproc, siebprocmw -ErrorAction SilentlyContinue | Stop-Process -Force
```

Give it a few seconds for the OS to release what they were holding, then
start the Siebel Server service as normal. It should come up cleanly. This
is exactly what a host restart accomplishes - it just clears the processes
as a side effect - but killing the four workers directly is faster and
doesn't take the whole box down.

## Doing it across an estate

The manual version means remoting into the affected host, opening Task
Manager or PowerShell, and clearing the processes by hand. Fine once. Less
fine when it's a multi-node environment, or when it happens at 2am as part
of a scheduled restart that's now stalled.

The same check works remotely over CIM/WMI, so you don't have to RDP in:

```powershell
Get-CimInstance Win32_Process -ComputerName SBLAPP01 `
  -Filter "Name='siebmtsh.exe' OR Name='siebmtshmw.exe' OR Name='siebproc.exe' OR Name='siebprocmw.exe'"
```

and terminate what comes back. Wrap that with a check that the Siebel
Server service is genuinely stopped first - you never want to kill workers
belonging to a *running* server - and you have a safe, repeatable cleanup.

## Prevention

You can't always prevent the ungraceful shutdown that leaves orphans
behind, but you can stop it from blocking the next start: make the orphan
check a standard step in your startup routine. Before starting a Siebel
Server service that's currently stopped, look for surviving workers and
clear them first. It turns a mystifying failed start into a non-event.

---

*This is one of the checks built into [Siebel Service Manager for
Windows](/tools/siebel-service-manager/) - it detects orphaned worker
processes on any host whose Siebel Server service isn't running and clears
them automatically before a start, across every node, with no Remote
Desktop. If you manage Siebel restarts regularly, it's worth a look.*
