---
layout: post
title: Zen and the Art of Reducer Maintenance
---

Since React is essentially a facade of UI components painted and propped up (pun intended) by its data, it's important to understand the data flow. A wise man once told me "The soul of a react app is in the reducers." With that in mind, I wanted to get a hang of the most-used reducer in the app I'm working on, in order to have a better intuition for its issues.

The app is <a href="http://united.vote">united.vote</a>, and I began to unpack it in <a href="https://tandcsurf.github.io/Dealing-With-A-Big-New-Scary-Codebase/">this post</a>. One of the things that stands out about the code, is the ubiquity of the "SYNC_BILLS" reducer being called. It seems to be the primary means of hydrating the app with data, as well as updating it, in several components throughout the app.

Here it is as I found it:

<pre><code>
case 'SYNC_BILLS': // eslint-disable-line no-case-declarations
    const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])
    const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})

    action.bills.forEach((bill) => {
      console.log(bills[bill.bill_uid], "bills[bill.bill_uid]");
      bills[bill.bill_uid] = bill
      // console.log(bills, "bills inside of the forEach");
    })
    console.log(bills, "bills")

    const newBills = _.orderBy(Object.values(bills), ['last_action_date', 'bill_uid'], ['desc', 'desc'])
    console.log(newBills, "newBills")
    newBills.synced = action.synced

    return { ...state,
      bills: { ...state.bills,
        [action.legislature || action.date]: newBills,
      },
    }
  </code></pre>
  
  Production code isn't always an easy thing to scan. In our case variables are declared, and objects and arrays are worked on, but the order is a little scattered. On top of that, this code is very concise.
  
 And though "very concise" code has a brevity that's eloquent to behold, easy readability usually takes a bit of a hit. So let's break this down:
 
```
const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])
const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})
```

We declare oldBills, and set it equal to the result of a ternary operator. Does the action object have a 'replace' property? If so, set oldBills to an empty array, giving us a blank slate. If not, oldBills will be set to the current bills in state, specific to the [action.legislature] or [action.date] specified in the action that was dispatched.

The next line, where we declare ``const bills``, is a little trickier. We're taking our oldBills and reducing it down. If oldBills was set to be an empty array, we're done. But if we do have some bills:

We're starting our reduce with the accumulator referencing an object, the initial value being just a blank object literal, ``{}``.

On each iteration of reduce, it's going to run Object.assign. The first parameter (the target) is ``obj``, which is referencing our accumulator, ``{}``; the reference being passed in by our enclosing reduce call.

Object.assign is going to, on each iteration, create an object, take the ``bill_uid`` from each bill, and assign that key that entire bill object. That key and its associated object are going to be accumulated in our accumulator, the empty object: ``{}``. Repeat until we've passed through all of oldBills.

So now, assuming we had a couple of dusty oldBills chuckling around the woodwork, we should have an object that looks like this:

``{us-115-s-res-327: {…}, us-115-s-res-1866: {…}, us-115-hr-res-3889: {…}, us-115-hr-res-3823: {…}}``

Next up, we're gonna start bringing in some fresh data:

```
action.bills.forEach((bill) => {
  bills[bill.bill_uid] = bill
})
```

action.bills is our action payload, and the app gets it from a data API. It's a big chunk of JSON, an object full of objects.

So now we're going to iterate over it, and for each bill that we've pulled in via fetch and dispatched to our reducer, we're going to see if see if the bills object above has a match for the ``bill_uid``. If so, we're going to update it with that bill. However, if bills is empty, or the ``bill_uid`` doesn't exist, we'll add it to the object.

We end up with the same structure we have above, an object of ``bill_uid`` keys, and the associated values, like above:

``{us-115-s-res-327: {…}, us-115-s-res-1866: {…}, us-115-hr-res-3889: {…}, us-115-hr-res-3823: {…}}``

So now that we've either updated or created our bills object, the reducer reformats the data before returning an updated copy of state:

```
const newBills = _.orderBy(Object.values(bills), ['last_action_date', 'bill_uid'], ['desc', 'desc'])
newBills.synced = action.synced
```



 
 
 
 



