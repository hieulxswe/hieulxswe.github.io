---
title: "[en] Delete Xcode Previews' cached data"
date: 2024-05-06T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: /delete-xcode-previews-cached-data/
categories: ios
tags: [ios, xcode, xcode previews, xcode tips]
image: ../assets/images/xcode-previews-cache.png
---

Xcode Tip💡

Delete Xcode Previews' cached data to save disk space and resolve any unusual behaviors with Previews.

## Step 1: Open Terminal
## Step 2: Use command
```php
xcrun simctl --set previews delete all
```
