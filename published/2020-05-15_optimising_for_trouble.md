# Optimising for trouble

Building an application to run on the heroku free tier is an exercise in optimisations. You need to consider your process counts,
do you need a background worker, what about database rows or even redis memory. These are all things that is you aren't cardefully
will bring down you application. Now don't get me wrong I'm happy to pay to a heroku worker if it is required, but I mostly definitely
don't want to pay for 3, plus a database if I don't have to.

I recently been building a card game, and my first thjought was that carding each card individually would quickly exceed the DB row limit
(currently 10,000) and bring down the application. To avoid this I stored the cards as a JSON blob on a single game object.
This approach is measn that I would need to deserailise the game state, update and the serialize it again for every update, this might(?)
be fine if I only have a single playing in the game, for a multiplayer game this would quickly become a bottleneck and
result in slow response times

## The initial solution

Storing card ownership in a redis DB, this would be super quick way to manage card locking adn avoid hitting the DB.

This approach has a number of issues:

* Redis locking is a pain
* ID selection is difficult, especially when you don't have a card ID fro the front-end, but instead just a location key.
* How do you release a locks? what happens if the user refreshes while holding a card?
* How do you avoid sync issues between redis and the postgres DB
* I would need a background worker to actually perform events later and sync the redis data back into postgres

Not only that, it was hard to reason about, difficult to tests for and would add a large amount of complexity to the
core of the backend, something that is never a good thing at the start of a project.

## The easy solution

While thinking over the problems I was having with the redis solution I satrted to think about why I wasn't just using the postgres DB to
do my locking. After all postgres is pretty good at locking, it has a number of feature to deal with different lockjing scenarios. In fact the
only reason I wasn't using it was the row limit, so I has a look at my current usage, 10 rows...

If I assumed each game would need around 100 cards, then I could have 100 games before I actually hit the row limit. It was at this
point I realised I has made a mistake. I was optimising against an issue that didn't currently exist, and if I was careful with my
implementation it would potentially be thousands of games before it was an issues.

With that in mind I update the application to store each card in the DB, this means that eadch player can update game cards simultaneously.
It:
* Solved the locking problems as I could just releasing any card I currently holding - with a simple update query - before I selected a new card.
* Was easy to reason about and test, which was a big win over the redis solution.
* allowed updated in request instaed of needing a back ground wokrer, as it only has to update simple card objects instead of a shared global state.

I still have the card state on the game object, this allows games which have been completed (or where no-one has made a move in a while)
to be archived. These chnages mean I would require around 100 simultaneous games in process before I reached the DB limit.

## Conclusions

I guess the key point here is that optimising is great, but if it isn't a requirement right now, then you are most likely just making your
code more complex without any gain, and in some cases you could even be dooming your project to fail. This is an easy pitfall to run into and
can be hard to recognise from the inside, but once you do it is often clear where you went wrong. The best way to guard against this to watch
out of smells in your code, as good design will (as a general rule) lead to simple code that is easy to understand.

