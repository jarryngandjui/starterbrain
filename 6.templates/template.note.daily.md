---
id: ""
cover: ""
kind:
  - sprint
status:
  - 🔲 backlog
project: "[[daily]]"
story: ""
complete: false
category:
  - planning
tags:
  - template
url: ""
links: ""
due date: ""
completed date: ""
start date: <% tp.date.now('YYYY-MM-DDTHH:mm') %>
cancelled date: ""
notetoolbar: sprint
productivity: ""
---
<% await tp.file.include("[[script.daily]]") %>
# Tasks
```tasks
preset tasks_daily
```

# Retro
 - What went well?
 - What I want to improve? 