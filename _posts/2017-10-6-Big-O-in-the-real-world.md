---
layout: post
title: Big O in the ~real world~
---

For most people learning software engineering, big O notation is initially cast aside as obscure CS-nerd witchcraft: "Algorithm efficiency? My arrays contain dozens of things, dozens! And everything runs smooth as a dolphin"

If you're developing a simple UI or small one-page app, your low hanging fruit for perfomance might be as simple as debouncing a nifty scroll event function. But what happens when you build an app that plays with a lot of data? And what if that data needs to be sorted out client-side before you can hook it into your UI? Your app might now be running as smooth as a dolphin trapped in a brillo-pad factory. And that's sad. What can we do to free the dolphin?

But first: Having slow performance right out of the gate isn't bad - it's good not to sweat refinements. While it's good to take the time to think carefully about the shape of your data and how to build a UI around it, data speed can take a back seat til later.

But sooner or later:

![](/images/guygirlmeme.jpg)


Right now, I'm working on <a href="https://liquid.vote">liquid.vote</a>, a cool project I talked about rolling my sleeves up on in my <a href="https://tandcsurf.github.io/Dealing-With-A-Big-New-Scary-Codebase/">last post</a>. An integral part of the UI is displaying a past legislative agenda, which is organized by date components:

![](/images/liquidvotesf.png)

When you click through the dates on the SF agenda, there are very few bills, in contrast to /nyc (what I'm working on), where each date has a hundred or more bills - the sample I got from our /nyc scraping API turned out a doozy:

![](/images/liquidvotenycdata.png)

Until we refine the legislative agenda-scraping API for the /nyc endpoint, we'll already be doing some lifting on the client side to get an array of dates out of the bills array. Eventually, the dates array could have thousands of entries.

Since we're ultimately going to map over the dates array with the goal of only laying down one date component per date, we'll need to remove all the duplicates. There are a bunch of ways to remove duplicates from an array, so let's take a look at them, and see if we can pluck something fast and efficient off the vine, using a little big of big O to forge comparisons.

One concise way to do this is with filter:

<pre><code>
uniqueDates = dates.filter(function(date, pos) {
    return dates.indexOf(date) === pos;
})
</code></pre>

This is pretty straightforward, and totally works. Filter iterates over the dates one by one, and compares the current date's position to the index of its first occurrance in the array. However, when a function takes an array item and compares it to all other array items, you end up with quadratic time: O(N^2). The time difference will be negligable for small arrays, but will become significant with anything big.

What about underscore or Lo_Dash?

These work similarly to the function above, and also result in quadratic time (O(N^2).

Thankfully, we can use a hash table to remove duplicates much more efficiently:

<pre><code>
function unique(array) {
    var seen = {};
    return a.filter(function(item) {
        return seen.hasOwnProperty(item) ? false : (seen[item] = true);
    });
}
</code></pre>

This works by placing each item in a hash table, and doing an instant lookup of each item along the way to see if it's already in the table. It's worth noting that hash keys don't distinguish between numbers and numeric strings, but thankfully, our data is purely numeric strings. This method gives us linear time: O(n), which will be way faster than the first code snippet.

Given the general "expense" of function calls, we can speed up the above even faster by replacing filter with a loop:

<pre><code>
function uniqueFaster(a) {
    var seen = {};
    var uniqueArray = [];
    var len = a.length;
    var j = 0;
    for(var i = 0; i < len; i++) {
         var item = a[i];
         if(seen[item] !== 1) {
               seen[item] = 1;
               output[j++] = item;
         }
    }
    return output;
}
</code></pre>

However, this is a refinement. If we want something that's a good middle ground, there's another solution, using ES6 Sets. It's relatively quick, and avoids the imperative soup above.

<pre><code>
const uniqueArray = array.from(new Set(a));
</code></pre>

The creation of the Set has a time complexity similar to our solution above. I'm assuming that array.From has an indexing cost for each element, giving it a time complexity of O(n). If the combined operation is O(2n), this still beats the pants off of quadratic time, and is a very clean one-line solution. Since this is quick and concise, we went ahead and stuck with this, making note that it could always be swapped out later.


