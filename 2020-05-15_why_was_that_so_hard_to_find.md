# Why was that so hard to find

In the interest of learning something new and filling in my time during the lockdown, I have been building my first react application.
I recently been moving to use hooks and adding test coverage. For the most part I have found it easy to write the code, maintain sensible
object boundries, and find help online when needed. However the documentation for to update data from external processes was
unclear and for the most part non-existant.

I'm not sure if this is due to me being new to React and everyone just knows how to do this, or (and this seems unlikley) the only
people who write react blogs are actually those who are building a tasks list application and so don't need this functionality.
What I will say is that once you understand how to use the hooks to pull data in it is actually extremely easy, which again makes
me ask why this isn't covered more widely.

So how did I do it:

### Component hold the global state and poll for updates:

```
let cardsByStack = {};
let watchers = {};

// allow registering to receive updates
export const watch = (stackId, watchMethod) => {
  watchers[stackId] = watchers[stackId] || [];
  watchers[stackId].push(watchMethod);

  watchMethod(cardsByStack[stackId]);
}

// allow deregistering to receive updates
export const unWatch = (stackId, watchMethod) => {
  watchers[stackId] = watchers[stackId].filter(watcher => watcher !== watchMethod);
}

// do an update
export const pollForAddedCards = () => {
  let stack_id, cards = ...

  cardsByStack[stack_id].push(card);

  watchers[stackId].forEach(watchMethod => watchMethod(cards[stackId]));
}

setInterval(pollForAddedCards, 1000)
```

### Registor interest in update:

```
import React, { useState, useEffect } from "react"
import {addEvent, updateCard, watch, unWatch} from '../state/CardState';

const CardStack = (props) => {
  const [cards, setCards] = useState();

  ....

  useEffect(() => {
    let watchCallback = (cards) => { setCards(cards) }

    watch(props.stackId, watchCallback)
    return () => { unWatch(props.stackId, watchCallback) }
  });
}
```

So how does this all work....

`useEffect` is the important thing here. Calling it without a second property means the code inside is run when
the component loads/unloads, this allows me to pass the state modifier funtion `setCard` into the non React
view hierarchy part of my application (called CardState here), where it can be called (and trigger view updates)
asyncronously. Once you have understoof this, it is quite easy to receive updates from external sources.

**WARNING:** This is demonstrating how you trigger render updates from external parts of your code -
i.e. using setTimeout to poll the server for updates from other players - with just react hooks. This is done by using
module variable to store what is essentially global state, and while this works I would recommend against it
for anything that have more than 1 global state. You are instead better off using one of the existing libraries like:
* redux
* reactn
* recoil
