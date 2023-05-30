---
title: "{{ replace .Name "-" " " | title }}"
date: {{ now.Format "2006-01-02" }}
mathjax: false
draft: true
tags: [craft, {{ index (split (replace .Name "-" " ") " ") 0 }}]
---
