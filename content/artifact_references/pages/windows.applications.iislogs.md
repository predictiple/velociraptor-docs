---
title: Windows.Applications.IISLogs
hidden: true
tags: [Client Artifact]
---

This artifact enables grep of IISLogs.
Parameters include SearchRegex and WhitelistRegex as regex terms.


<pre><code class="language-yaml">
name: Windows.Applications.IISLogs
description: |
  This artifact enables grep of IISLogs.
  Parameters include SearchRegex and WhitelistRegex as regex terms.

author: "Matt Green - @mgreen27"

parameters:
  - name: IISLogFiles
    default: '*:/inetpub/logs/**3/*.log'
  - name: SearchRegex
    description: "Regex of strings to search in line."
    default: ' POST '
    type: regex
  - name: WhitelistRegex
    description: "Regex of strings to leave out of output."
    default:
    type: regex

sources:
  - precondition: SELECT OS From info() where OS = 'windows'

    query: |
      LET files = SELECT OSPath FROM glob(globs=IISLogFiles)

      SELECT * FROM foreach(row=files,
          query={
              SELECT Line, OSPath FROM parse_lines(filename=OSPath)
              WHERE
                Line =~ SearchRegex
                AND NOT if(condition= WhitelistRegex,
                    then= Line =~ WhitelistRegex,
                    else= FALSE)
          })

    notebook:
      - type: vql_suggestion
        name: IIS Groks
        template: |
            /*
            ### IIS grok

            Note:  IIS doesn't have a standard logging format so we have added some
            suggestions. Comment in preferred or add / modify your own.
            */

            LET target_grok = "%{TIMESTAMP_ISO8601:LogTimeStamp} %{IPORHOST:Site} %{WORD:Method} %{URIPATH:UriPath} %{NOTSPACE:QueryString} %{NUMBER:Port} %{NOTSPACE:Username} %{IPORHOST:Clienthost} %{NOTSPACE:Useragent} %{NOTSPACE:Referrer} %{NUMBER:Response} %{NUMBER:Subresponse} %{NUMBER:Win32status} %{NUMBER:Timetaken:int}"
            --LET target_grok = "%{TIMESTAMP_ISO8601:log_timestamp} %{IPORHOST:site} %{WORD:method} %{URIPATH:page} %{NOTSPACE:querystring} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:clienthost} %{NOTSPACE:useragent} %{NOTSPACE:referer} %{NUMBER:response} %{NUMBER:subresponse} %{NUMBER:scstatus} %{NUMBER:timetaken:int}"
            --LET target_grok = "%{TIMESTAMP_ISO8601:log_timestamp} %{WORD:iisSite} %{NOTSPACE:computername} %{IPORHOST:site} %{WORD:method} %{URIPATH:page} %{NOTSPACE:querystring} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:clienthost} %{NOTSPACE:protocol} %{NOTSPACE:useragent} %{NOTSPACE:referer} %{IPORHOST:cshost} %{NUMBER:response} %{NUMBER:subresponse} %{NUMBER:scstatus} %{NUMBER:bytessent:int} %{NUMBER:bytesrecvd:int} %{NUMBER:timetaken:int}"


            LET parsed = SELECT Fqdn, ClientId as _ClientId, Line as _Raw,
                  grok(data=Line,grok=target_grok) as GrokParsed
              FROM source()

            SELECT * FROM foreach(row=parsed,
                  query={ SELECT *, Fqdn, _Raw FROM GrokParsed })

</code></pre>

