---
layout: post
title: Zen and the Art of Reducer Maintenance
---

Since React is essentially a group of UI components painted and propped up (pun intended) by its data, it's important to understand the data flow. A wise man once told me "The soul of a react app is in the reducers." With that in mind, I wanted to get a hang of the most-used reducer in the app I'm working on, in order to have a better intuition while contributing fixes and new features.

The app is <a href="http://united.vote">united.vote</a>, and I began to unpack it in <a href="https://tandcsurf.github.io/Dealing-With-A-Big-New-Scary-Codebase/">this post</a>. One of the things that stands out about the code, is the ubiquity of the "SYNC_BILLS" reducer being called. It seems to be the primary means of hydrating the app with data, as well as updating it, in several components throughout the app.

Here it is as I found it:

<pre><code>
case 'SYNC_BILLS': // eslint-disable-line no-case-declarations
    const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])
    const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})

    action.bills.forEach((bill) => {
      bills[bill.bill_uid] = bill
    })

    const newBills = _.orderBy(Object.values(bills), ['last_action_date', 'bill_uid'], ['desc', 'desc'])
    newBills.synced = action.synced

    return { ...state,
      bills: { ...state.bills,
        [action.legislature || action.date]: newBills,
      },
    }
</code></pre>
  
  Production code isn't always an easy thing to scan. In our case variables are declared, and objects and arrays are worked on, but the order is a little scattered. On top of that, this code is very concise.
  
 And though "very concise" code has a brevity that's eloquent to behold, easy readability usually takes a bit of a hit. So let's break this down:
 
<pre><code>
const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])
const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})
</code></pre>

We declare oldBills, and set it equal to the result of a ternary operator. Does the action object have a 'replace' property? If so, set oldBills to an empty array, giving us a blank slate. If not, oldBills will be set to the current bills in state, specific to the [action.legislature] or [action.date] specified in the action that was dispatched.

The next line, where we declare ``const bills``, is a little trickier. We're taking our oldBills and reducing it down. If oldBills was set to be an empty array, we're done. But if we do have some bills:

We're starting our reduce with the accumulator referencing an object, the initial value being just a blank object literal, ``{}``.

On each iteration of reduce, it's going to run Object.assign. The first parameter (the target) is ``obj``, which is referencing our accumulator, ``{}``; the reference being passed in by our enclosing reduce call.

Object.assign is going to, on each iteration, create an object, take the ``bill_uid`` from each bill, and assign that key that entire bill object. That key and its associated object are going to be accumulated in our accumulator, the empty object: ``{}``. Repeat until we've passed through all of oldBills.

So now, assuming we had a couple of dusty oldBills chuckling around the woodwork, we should have an object that looks like this:

``{us-115-s-res-327: {…}, us-115-s-res-1866: {…}, us-115-hr-res-3889: {…}, us-115-hr-res-3823: {…}}``

Next up, we're gonna start bringing in some fresh data:

<pre><code>
action.bills.forEach((bill) => {
  bills[bill.bill_uid] = bill
})
</code></pre>

action.bills is our action payload, and the app gets it from a data API. It's a big chunk of JSON, an object full of objects.

So now we're going to iterate over it, and for each bill that we've pulled in via fetch and dispatched to our reducer, we're going to see if see if the bills object above has a match for the ``bill_uid``. If so, we're going to update it with that bill. However, if bills is empty, or the ``bill_uid`` doesn't exist, we'll add it to the object.

We end up with the same structure we have above, an object of ``bill_uid`` keys, and the associated values, like above:

``{us-115-s-res-327: {…}, us-115-s-res-1866: {…}, us-115-hr-res-3889: {…}, us-115-hr-res-3823: {…}}``

So now that we've either updated or filled in our empty bills object, the reducer looks to want to reformat it:

<pre><code>
const newBills = _.orderBy(Object.values(bills), ['last_action_date', 'bill_uid'], ['desc', 'desc'])
newBills.synced = action.synced
</code></pre>

The end result is an array of objects. Working our way from the inside-out:

Object.values is taking our bills object full of bills, and turns it into an array of objects. In essence, the bill_uid keys get lopped off, and we're left with an array:

``[{…},{…},{…},{…}]``

And now, lodash provides us an orderBy function, which is supposed to reach into the objects, and sort them according to the ``last_action_date`` and ``bill_uid``.

And after that, we return an updated copy of state:

<pre><code>
return { ...state,
      bills: { ...state.bills,
        [action.legislature || action.date]: newBills,
      },
    }
</code></pre>

Now that we've traced through the code and understood it, what stands out?

First off, orderBy isn't actually reordering anything. Could it be because the values it's looking for are numerical strings instead of numbers? For now, it's not a breaking change.

Looking back at our ternary in oldBills:

<pre><code>
const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])``
</code></pre>

It looks like it always returns an empty array. The only time I found a relevant reference to "replace" in the code was in a search function that isn't wired up yet. We can probably chalk this up to lots of the app's plumbing being unfinished.

Those are just two things that immediately presented themselves. Now, with a better understanding, whenever I see SYNC_BILLS popping off throughout the components, I'll have a good intuition of what exactly is happening to the data that's being dispatched. And since the legislation data drives a vast majority of the components, I'd say this was time well spent.






 
 
 
 



