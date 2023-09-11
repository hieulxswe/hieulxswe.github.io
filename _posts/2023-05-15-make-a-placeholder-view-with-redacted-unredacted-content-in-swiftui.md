---
title: "Make a placeholder content in SwiftUI with Redacted & Unredacted"
date: 2023-05-15T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: //make-a-placeholder-view-with-redacted-unredacted-content-in-swiftui/
categories: SwiftUI
tags: [swiftui, ios]
---

![alt](http://hieulxswe.com/wp-content/uploads/2023/05/redacted.png)

{% highlight swift %}

import SwiftUI

struct PlaceholderView: View {
    var body: some View {
        VStack(spacing: 20) {
            RoundedRectangle(cornerRadius: 10)
                .foregroundColor(.gray)
                .frame(height: 200)
                .redacted(reason: .placeholder)
            Text("Title")
                .font(.title)
                .redacted(reason: .placeholder)
            Text("Description")
                .font(.body)
                .multilineTextAlignment(.center)
                .redacted(reason: .placeholder)
        }
        .padding()
    }
}

struct ContentView: View {
    @State var isLoading = true
    
    var body: some View {
        if isLoading {
            PlaceholderView()
                .redacted(reason: .placeholder)
        } else {
            VStack(spacing: 20) {
                Image(systemName: "person.crop.circle.fill")
                    .resizable()
                    .frame(width: 100, height: 100)
                Text("Hieu X. Leu")
                    .font(.title)
                Text("Software Developer")
                    .font(.body)
                    .multilineTextAlignment(.center)
            }
            .padding()
        }
        
        Button(action: {
            isLoading.toggle()
        }, label: {
            Text(isLoading ? "Show content" : "Hide content")
        })
    }
}

{% endhighlight %}

In this example, I creating a `PlaceholderView` that contains a gray rectangle as a placeholder for an image, a title, and a description. I using the `.redacted(reason:)` modifier on each of these views to redact the content and display a placeholder instead.

I also using a `@State` property called `isLoading` to toggle between the redacted and unredacted views. When `isLoading` is true, I show the `PlaceholderView` with the `.redacted(reason:)` modifier. When `isLoading` is false, I show the unredacted content.

To switch between the redacted and unredacted views, I using a `Button` with an action that toggles the `isLoading` state.

This is useful if you want to display a placeholder while data is being loaded, and then switch to the actual content once it's available.