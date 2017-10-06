---
layout: post
title: Big O in the ~real world~
---

For most people learning software engineering, big O notation is initially cast aside as obscure CS-nerd witchcraft: "Algorithm efficiency? My arrays contain dozens of things, dozens! And everything runs smooth as a dolphin"

If you're developing a simple UI or small one-page app, your low hanging fruit for perfomance might be as simple as debouncing a nifty scroll event function. But what happens when you build an app that plays with a lot of data? And what if that data needs to be sorted out client-side before you can hook it into your UI? Your app might now be running as smooth as a dolphin trapped in a brillo-pad factory. And that's sad. What can we do to free the dolphin?

But first: Having slow performance right out of the gate isn't bad - it means you built something. Perfectionism has a way of making doable things seem like moving mountains, so it's good not to sweat refinements. While it's good to take the time to think carefully about the shape of your data and how to build a UI around it, data speed can take a back seat. But sooner or later, performance will catch your eye.

![](/images/guygirlmeme.jpg)


In the app we're making (link here), we get data objects that get filtered down to their date. These data objects could potentially be gigantic. We need to remove duplicate dates, since the component we're concerned with only renders one date on a date menu, revealing the associated bills after.

There are a few ways to remove duplicates

Example one: n2

example two: n2

example three: n logn

let's roll with example 3
