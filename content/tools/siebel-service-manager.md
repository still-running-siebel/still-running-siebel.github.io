---
title: "Siebel Service Manager for Windows"
price: "$99"
gumroad: "https://stillrunner.gumroad.com/l/siebel-service-manager"
weight: 1
---

Stop, start, and restart a Siebel environment the way an experienced admin
would - in order, across every host, with one command. A PowerShell module
for administrators, DBAs, and consultants running Siebel CRM on Windows.

**What it does**

- Ordered, tiered start/stop/restart across one or many hosts - gateway,
  Tomcat containers, Siebel Server - with configurable settle delays
  between tiers.
- Clears orphaned `siebmtsh` / `siebproc` worker processes that block a
  Siebel Server start after a failed shutdown - remotely, no RDP.
- Hang detection and recovery for services stuck in a pending state.
- Discovery that scans your hosts and writes the environment config for
  you.
- Health checks, `-WhatIf` safety, and a timestamped log per run.
- No agent. Plain PowerShell 5.1 or 7+, CIM over DCOM/WSMan. Full source
  included.

Single-organization license - use it across every environment you
administer. Requires Windows PowerShell 5.1+ and rights to control
services on your Siebel hosts.

Questions before buying? [hello@stillrunningsiebel.com](mailto:hello@stillrunningsiebel.com)
