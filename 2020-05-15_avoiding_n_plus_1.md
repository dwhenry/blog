# Avoiding N+1

Rails apps are slow, this is a known fact, ask anyone. But do people reaslly know why there app is so slow, it rarely the framework,
though that can contribute. Most endpoints are slow because the are doing too much, or are doing it in a way that is inefficient. The
classic examplke with rails is N+1 queries, where you access and assoiacted object in the view for each record in an array and it does
an extra query for object, or you got mutliple objects deep in the view and suddenly you are doing 100's if not more DB lookups.

Rails offer a simple way to avoid this, you can just include the assocaition(s) and that will load them all into memory in a single DB
query. What about when you have multiple associations nested multiple levels deep? Rails will handle that but it quickly becomes difficult
to follow and requires each developer to mapp out the associations on each endpoint.

As a contract developer being asked to help speed up/simplify this type of legacy code - that was fine with 10 clients, but is now falling
over with 100 - I am always looking for better ways to approach this sort of problem.

I was recently introduced to a clear approach that allows the assocaitiosn to be loaded in the models, instaed of needing to declare
them in the controllers, and while this approach only work well where you have multiple endpoints which share similar loading requirements,
it is definitely a trick
