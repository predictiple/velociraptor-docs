---
title: Server.Information.Clients
hidden: true
tags: [Server Artifact]
---

This artifact returns the total list of clients, their hostnames and
the last times they were seen.


<pre><code class="language-yaml">
name: Server.Information.Clients
description: |
  This artifact returns the total list of clients, their hostnames and
  the last times they were seen.

type: SERVER

sources:
  - query: |
        SELECT client_id,
               os_info.fqdn as HostName,
               os_info.system as OS,
               os_info.release as Release,
               timestamp(epoch=last_seen_at/ 1000000).String as LastSeenAt,
               last_ip AS LastIP,
               last_seen_at AS _LastSeenAt
        FROM clients()
        ORDER BY _LastSeenAt DESC

</code></pre>

