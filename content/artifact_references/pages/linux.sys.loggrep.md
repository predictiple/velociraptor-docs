---
title: Linux.Sys.LogGrep
hidden: true
tags: [Client Artifact]
---

This artifact enables grep and zgrep of linux logs and gzipped log files.


<pre><code class="language-yaml">
name: Linux.Sys.LogGrep
author: "Matt Green - @mgreen27"
description: |
  This artifact enables grep and zgrep of linux logs and gzipped log files.

parameters:
  - name: TargetGlob
    default: /var/log/**
  - name: GrepRegex
    type: regex
    description: "Regex of strings to search in line."
    default: 'malware\.php'
  - name: WhitelistRegex
    type: regex
    description: "Regex of strings to leave out of output."
    default:

sources:
  - query: |
      LET files = SELECT OSPath FROM glob(globs=TargetGlob)
        WHERE NOT IsDir

      SELECT * FROM foreach(row=files,
          query={
              SELECT Line, OSPath FROM parse_lines(filename=OSPath)
              WHERE
                Line =~ GrepRegex
                AND NOT if(condition= WhitelistRegex,
                    then= Line =~ WhitelistRegex,
                    else= FALSE)
          })

</code></pre>

