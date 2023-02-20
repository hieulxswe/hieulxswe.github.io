---
title: Steps Upload Xcode Project to GitHub
tags: [ios, swift, github, steps upload project to github, upload source code to github]
style: 
color: 
description: Hey guys! I want to show how to upload Xcode project to GitHub (you must have a GitHub account. If you don’t have one, you can create it).
image: https://cdn-images-1.medium.com/max/873/1*9Cohf2sZkt6KqkrAypg-IQ.png
comments: true
---

## Step 1:
Open `Xcode` project and press `Xcode` / `Preferences`

![Image 1](../assets/images_post/xcode-github-1.png)

Click Accounts and find the `+` button, in the open window, scroll a bit and find `GitHub`, select it and click Continue.

![Image 2](../assets/images_post/xcode-github-2.png)

Now we can see Sign in to your GitHub account, add your GitHub name and Access Token. To find this information, visit your GitHub account.

![Image 3](../assets/images_post/xcode-github-3.png)

## Step 2:
Open `GitHub` account and press `Profile` / `Settings`

![Image 4](../assets/images_post/xcode-github-4.png)

Scroll a little and select `Developer settings`

![Image 5](../assets/images_post/xcode-github-5.png)

Click Personal access tokens and select Tokens (classic)

![Image 6](../assets/images_post/xcode-github-6.png)

In the Confirm access window that opens, enter your password. And now you need to configure your future token. In the Expiration field, I recommend choosing No expiration. Select scopes — I select all of these. And click Generate token.

![Image 7](../assets/images_post/xcode-github-7.png)

Now you can see your Personal access tokens. Just cope and save it.

![Image 8](../assets/images_post/xcode-github-8.png)

## Step 3:
We can now go back to Xcode and add all the necessary information.

![Image 9](../assets/images_post/xcode-github-9.png)

Open `Source Control` navigator, choose `Repositories`, click `Remotes` and preferred `New "Upload to GitHub" Remote…`

![Image 10](../assets/images_post/xcode-github-10.png)

Just make a little settings to your repository.

![Image 11](../assets/images_post/xcode-github-11.png)

## Step 4:
And finally go to `Source Control` and click `Push`

![Image 12](../assets/images_post/xcode-github-12.png)

Click `Push`

![Image 13](../assets/images_post/xcode-github-13.png)

And we’re done! To make sure everything is fine, we can visit our GitHub and check if the new project is available in the Repositories.

![Image 14](../assets/images_post/xcode-github-14.png)