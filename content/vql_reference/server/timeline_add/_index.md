---
title: timeline_add
index: true
noTitle: true
no_edit: true
---



<div class="vql_item"></div>


## timeline_add
<span class='vql_type pull-right page-header'>Function</span>



<div class="vqlargs"></div>

Arg | Description | Type
----|-------------|-----
timeline|Supertimeline to add to. If a super timeline does not exist, creates a new one.|string (required)
name|Name/Id of child timeline to add.|string (required)
query|Run this query to generate the timeline.|StoredQuery (required)
key|The column representing the time to key off.|string (required)
message_column|The column representing the message.|string
ts_desc_column|The column representing the timestamp description.|string
notebook_id|The notebook ID the timeline is stored in.|string

Required Permissions: 
<i class="linkcolour label pull-right label-success">READ_RESULTS</i>

### Description

Add a new query to a timeline.

