---
layout: post
title: Dealing With A Big New Scary Codebase
---

There's nothing more disappointing than cracking open an old project of my own, clicking through the files, scanning the code, and wondering "Oh shit, how does this work again?". When working on something for weeks, I feel like the master of that little universe by the time I wrap it up, making that pang of confusion when revisiting an unempowering feeling. 

It's a natural occurance, and thankfully, it can be mitigated by thoughtful and well-positioned comments, as well as having a clean and intuitive project structure to scaffold the whole thing. But what about code that you've never seen before? What if it's devoid of comments, and the structure seems at first glance like an endless sprawl of alien folders that seem to just go deeper and deeper? Worse yet, what if it's half-way through development, doesn't work, and you have little or no context for why?

The feeling of bewilderment that ensues can be a bit like tumbling down a well. You look up, see a pinprick of light at the top, and have a moment of panic. Is it even possible to get out? Should you just give up and resign to living life as a ghoulish well-monster, whose only appearance on the surface will be to haunt people in a blockbuster horror franchise? That's no way to live.

Myself and a pair-programming pal were recently invited by a friend to take part in a project he's contributing to. I've spent the past two months or so living in a blissful world of smaller react projects that are flat and simple enough to where even Redux seems like an unnecessary complication. Looking at this felt like a clusterfuck:

(insert pic of liquid vote project structure)

Wew, that's a lot of components to digest. There are hundreds of them, and many of them are spawned out of iteration. The rest are deeply nested.

It helps tremendously to just take a moment to calm down and embrace the confusion. It's good to recognize that any issues won't (or can't) be solved right away. You might spend hours, or even days, understanding how all the plumbing in a project fits together, how data flows, and where the edges of the current build drop off. It's like a surgery case. The doctors are going to pore over an MRI to understand what's going on under the hood before they start scooping stuff out willy-nilly.

So let's do that initial legwork in familiarity. Let's take a look at the repo up on github to see what kind of branches we're working with:

(pic of branches and update dates)

It looks like gh-pages is a whole bunch of commits ahead of master
v0.1.0 is a few ahead..

In this case, it looks like v.0.1.0 is the production branch. It's not intuitive at first glance, but the total amount of commits, as well as commits ahead, can be pretty misleading. One of the commits in the production branch had likely taken in a lot of changes and consolidations at once, and has been updated more recently. After we fork and clone down the repo, we'll work from that.

Now that we have the project, let's start it up and see if we can get some sort of feedback loop going.

So, unsurprisingly, it doesn't work.

(screenshot of console log with 1776 and 2018 not found)

So the immediate concern has something to do with the localhost ports the dev server is trying to connect to. Let's do a projectwide search in atom to see where references to 1776 and 2018 are lurking:

(screenshot of webpack)

Bingo. Looks like it's a node environment variable thang. Since the API_URL_V1 and API_URL_V2 wasn't defined when I started the dev server, it's defaulting to these local ports. And we're not running anything on them.

Can we run the API locally? Unfortunately not. The API is closed, and we can't really mess with it. But we do have links to the API from elsewhere in the repo, so let's go ahead and make sure the webpack script has access to API_URL_V1 and API_URL_V2 and doesn't default to something we can't run

(screenshot of node command, or hard coding)

(screenshot of front end)
![it works!](/images/frontendworks.jpg)

F yeah, we have a front end up!

It seems to work just fine for US congress, but what about the /sf endpoint, for local san francisco bills?

(screenshot of empty object in react, and sync error)

![](/images/Screen%20Shot%202017-10-03%20at%202.04.44%20PM.png)

Looks like the bills object is empty. We also have this sync error. We can deduce that the data isn't getting hooked in. Where does sync occur in the project?

(screenshot or snippet of sync in the bill file).
<pre><code>
componentDidMount() {
    const { dispatch, isVerified, match, sessionId } = this.props
    const { date } = match.params
    if (!this.props.bills[date]) {
      if (date) {
        fetch(`${API_URL_V1}/bills/${date}`)
          .then(response => console.log('response:', response.json()))
          .then(bills => dispatch({ bills, date, type: 'SYNC_BILLS' }))
      } else {
        fetch(`${API_URL_V2}/legislation/?json=${JSON.stringify({ legislature: 'us' })}`)
          .then(response => response.json())
          .then(bills => bills.map(oldBill))
          .then(bills => dispatch({ bills, legislature: 'us', type: 'SYNC_BILLS' }))
      }
    }
 </code></pre>

Since it seems like we have the dates, it's trying to fetch data from the V1 API URL. Before we get ahead of ourselves, let's drop a console log in there to make sure the data is working out. If the API is goofed up, our options for recourse here are looking pretty bad.

(screenshot of data) Cool. That works just fine. We're getting the data. 



So let's check out what's going on in the reducer:

(screenshot or snippet or reducer)
<pre><code>
case 'SYNC_BILLS': // eslint-disable-line no-case-declarations
    console.log('sync bills: ', action)
    const oldBills = (action.replace ? [] : state.bills.us) || []
    const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})

    if (action.legislature === 'us') {
      action.bills.forEach((bill) => {
        bills[bill.bill_uid] = bill
      })
    }
 </code></pre>

I think this is the crux of our problem. One thing to take note of is the 'legislature: us' key. The US congress version of the site works just fine.. But this endpoint should presumably be legislature: sf. Have the maintainers of this repo gotten that far?

Taking a quick glance at the repo to see if there's any insight, it looks like they've updated this issue just hours ago, right around when we started working for the day. Our detective work and hunch was correct, and we zeroed in on the problem in a pretty big, unfamiliar project. Although we got beat to the punch on the fix, while sifting through the folders, components, and data flow, we began building out a mental model of how this project works. There are still lots of nooks and crannies unexplored, but working on it is a whole lot less scary.



