---
title: Windows.System.AuditPolicy
hidden: true
tags: [Client Artifact]
---

Artifact using auditpol to retrieve the logging settings
defined in the Windows Audit Policy.

Use this artifact to determine what Windows event logs are audited
and if there are any discrepancies across the environment.


<pre><code class="language-yaml">
name: Windows.System.AuditPolicy

description: |
   Artifact using auditpol to retrieve the logging settings
   defined in the Windows Audit Policy.

   Use this artifact to determine what Windows event logs are audited
   and if there are any discrepancies across the environment.

type: CLIENT

author: Zach Stanford - @svch0st

implied_permissions:
  - EXECVE

sources:
  - precondition:
      SELECT OS From info() where OS = 'windows'

    query: |
      LET output = SELECT * FROM execve(
        argv=["auditpol.exe","/get","/category:*","/r"])

      SELECT * FROM foreach(
        row=output,
        query={
            SELECT * FROM parse_csv(filename=Stdout,accessor="data")
        }
      )

</code></pre>

