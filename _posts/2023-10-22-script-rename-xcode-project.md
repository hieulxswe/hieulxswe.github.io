---
title: "Script rename Xcode Project"
date: 2023-10-22T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: /script-rename-xcode-project/
categories: ios
tags: [rename xcode project, rename xcode]
---

Swift script for renaming Xcode project

It should be executed from inside root of Xcode project directory and called with two string parameters: `$OLD_PROJECT_NAME & $NEW_PROJECT_NAME` 

Script goes through all the files and directories recursively, including Xcode project or workspace file and replaces all occurrences of **$OLD_PROJECT_NAME** string with **$NEW_PROJECT_NAME** string (both in each file's name and content).

[image](../assets/images/rename-xcode-prj.png)
