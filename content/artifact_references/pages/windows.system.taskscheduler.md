---
title: Windows.System.TaskScheduler
hidden: true
tags: [Client Artifact]
---

The Windows task scheduler is a common mechanism that malware uses
for persistence. It can be used to run arbitrary programs at a later
time. Commonly malware installs a scheduled task to run itself
periodically to achieve persistence.

This artifact enumerates all the task jobs (which are XML
files). The artifact uploads the original XML files and then
analyses them to provide an overview of the commands executed and
the user under which they will be run.


<pre><code class="language-yaml">
name: Windows.System.TaskScheduler
description: |
  The Windows task scheduler is a common mechanism that malware uses
  for persistence. It can be used to run arbitrary programs at a later
  time. Commonly malware installs a scheduled task to run itself
  periodically to achieve persistence.

  This artifact enumerates all the task jobs (which are XML
  files). The artifact uploads the original XML files and then
  analyses them to provide an overview of the commands executed and
  the user under which they will be run.

parameters:
  - name: TasksPath
    default: C:/Windows/System32/Tasks/**
  - name: AlsoUpload
    type: bool
    description: |
      If set we also upload the task XML files.
  - name: UploadCommands
    type: bool
    description: |
      If set we attempt to upload the commands that are
      mentioned in the scheduled tasks
  - name: Username
    type: regex
    default: .*

sources:
  - name: Analysis
    query: |
      LET Uploads = SELECT Name, OSPath, if(
           condition=AlsoUpload='Y',
           then=upload(file=OSPath)) AS Upload, Mtime
        FROM glob(globs=TasksPath)
        WHERE NOT IsDir

      // Job files contain invalid XML which confuses the parser - we
      // use regex to remove the invalid tags.
      LET parse_task = select OSPath, Mtime, parse_xml(
               accessor='data',
               file=regex_replace(
                    source=utf16(string=Data),
                    re='&lt;[?].+?&gt;',
                    replace='')) AS XML
        FROM read_file(filenames=OSPath)

      LET Results = SELECT XML.Task.RegistrationInfo.URI AS TaskName,
            Mtime,
            expand(path=XML.Task.Actions.Exec.Command)  AS Command,
            XML.Task.Actions.Exec.Arguments AS Arguments,
            XML.Task.Principals.Principal.UserId AS UserId,
            XML.Task.Principals.Principal.RunLevel AS RunLevel,
            XML.Task.Principals.Principal.LogonType AS LogonType,
            XML.Task.Triggers.SessionStateChangeTrigger.StateChange AS StateChange,
            XML.Task.Actions.ComHandler.ClassId AS ComHandler,
            timestamp(epoch=XML.Task.RegistrationInfo.Date) AS RegistrationTime,
            timestamp(epoch=XML.Task.Triggers.CalendarTrigger.StartBoundary) AS StartBoundary,
            XML as _XML
        FROM foreach(row=Uploads, query=parse_task)

      SELECT *,
         authenticode(filename=Command) AS Authenticode,
         if(condition=UploadCommands and ExpandedCommand,
            then=upload(file=Command)) AS Upload
      FROM Results
      WHERE UserId =~ Username
      
column_types:
- name: Upload
  type: upload_preview
- name: Authenticode
  type: collapsed

</code></pre>

