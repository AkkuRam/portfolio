---
title: "Creating a disk analyzer in Python"
date: 2024-10-20T10:00:00+01:00
---

<a href="https://github.com/AkkuRam/disk-analyzer">
  <i class="fab fa-fw fa-github"></i> Disk Analyser
</a>

![Disk]({{ "images/disk_analyzer" | relURL }})

## Overview

This is a Python project, which is designed to replicate the "Performance" page of the Task Manager app. The library "Rich" was used for the terminal UI. The displayed statistics mainly utilizes the libraries (psutil, os & platform) displaying:

- CPU Usage & Specs
- Disk Space
- Network Speed in KB/S
- System & Other Specs