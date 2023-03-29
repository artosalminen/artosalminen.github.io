---
title: "How to collect all npm licenses from multiple subdirectories with Powershell"
date: 2023-03-29
series: ["Coding", "Powershell", "Git"]
weight: 1
tags: ["Coding", "Powershell", "Git"]
author: "Arto Salminen"
draft: false
---

1. Install `license-report` (https://www.npmjs.com/package/license-report)

   `npm install -g license-report`

1. run the script
   ```PowerShell
   Get-ChildItem -Directory | foreach { $_ >> ./licenses.csv ; license-report --output=csv --only=prod --package=./$_/package.json >> ./licenses.csv }
   ```

You can use the same idea to run other stuff in subdirectories. Just replace the command. Here is an example of `git pull`.

```PowerShell
Get-ChildItem -Directory -Force -Recurse *.git | ForEach-Object { cd $_.Parent.FullName; Write-Host $_.Parent.FullName; git pull }
```
