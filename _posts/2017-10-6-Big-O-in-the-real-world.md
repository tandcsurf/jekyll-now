---
layout: post
title: Big O in the ~real world~
---

For most people learning software engineering, big O notation is initially cast aside as obscure CS-nerd witchcraft: "Algorithm efficiency? My arrays contain dozens of things, dozens! And everything runs smooth as a dolphin"

If you're developing a simple UI or small one-page app, your low hanging fruit for perfomance might be as simple as debouncing a nifty scroll event function. But what happens when you build an app that plays with a lot of data? And what if that data needs to be sorted out client-side before you can hook it into your UI? Your app might now be running as smooth as a dolphin trapped in a brillo-pad factory. And that's sad. What can we do to free the dolphin?

But first: Having slow performance right out of the gate isn't bad - it means you built something. Perfectionism has a way of making doable things seem like moving mountains, so it's good not to sweat refinements. While it's good to take the time to think carefully about the shape of your data and how to build a UI around it, data speed can take a back seat. But sooner or later, performance will catch your eye.

![](/images/guygirlmeme.jpg)


Right now, I'm working on liquid.vote(link to website), a cool project I talked about rolling my sleeves up on in my last post(link to post). An integral part of the UI is displaying a past legislative agenda, which is organized by date components.

(screen of /sf agenda)

When you click through the dates on the SF agenda, there are very few bills, in contrast to /nyc (what I'm working on), where each date has a hundred or more bills - the sample I got from our /nyc scraping API turned out a dousey:

(screen of bills for one date)

Until we refine the legislative agenda-scraping API for the /nyc endpoint, we'll already be doing some lifting on the client side to get an array of dates out of the bills array. Eventually, the dates array could have thousands of entries.

Since we're ultimately going to map over the dates array with the goal of only laying down one date component per date, we'll need to remove all the duplicates. There are a bunch of ways to remove duplicates from an array, so let's take a look at them, and see if we can pluck something sexy off the vine.

Example one: n2

example two: n2

example three: n logn

let's roll with example 3
