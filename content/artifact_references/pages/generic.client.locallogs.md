---
title: Generic.Client.LocalLogs
hidden: true
tags: [Client Event Artifact]
---

Write client logs locally in an encrypted container. This helps when
we need to access what the client was doing in the past.


<pre><code class="language-yaml">
name: Generic.Client.LocalLogs
description: |
  Write client logs locally in an encrypted container. This helps when
  we need to access what the client was doing in the past.

type: CLIENT_EVENT

parameters:
- name: LocalFilename
  default: "%TEMP%/locallogs.log"
  description: The local filename that will be written (Env variables will be expanded).
- name: MaxRows
  type: int
  default: "100"
  description: Flush the file when we cache this many rows.
- name: MaxWait
  default: "60"
  type: int
  description: Flush the file at least every this many seconds.
- name: MaxSize
  default: "100000000"
  type: int
  description: Truncate the file once it reaches this length.
- name: AlsoForward
  type: bool
  description: |
    By default we do not forward any of the logs to the server but
    this allows logs to be forwarded as well as written locally.
- name: Component
  default: generic
  description: The log component to forward (default "generic")
  type: choices
  choices:
    - generic
    - client
    - frontend
    - gui
    - api

sources:
- query: |
     LET _ &lt;= log(message="Writing local log to " + expand(path=LocalFilename))

     SELECT * FROM write_crypto_file(
       max_rows=MaxRows, max_wait=MaxWait, max_size=MaxSize,
       filename=expand(path=LocalFilename),
       query={
         SELECT timestamp(epoch=now()) AS Timestamp, *
         FROM logging(component=Component)
       })
     WHERE AlsoForward

</code></pre>

