# Query scheduling

### Overview

The split queries are enqueued to a request queue by query-frontend.

Some queriers, which are connected to the query-frontend via bidirectional gRPC, get and process them.&#x20;

In addition, a querier creates a goroutine for each query so that it can handle some queries in parallel.&#x20;

When the query is completed, it returned the result to the query-frontend via gRPC.&#x20;

The queued query request has an original query-frontend address so that it can return it to the correct instance.

Here is the overall figure for processing a query.

![](<../.gitbook/assets/query\_schedule\_parameter.drawio (1).png>)

In conclusion, this architecture has some advantages as listed.

* Splitting a query by query-frontend allows queriers to process it in parallel
* The query-frontend enqueues queries and queriers pull them from the queue so that queriers can determine when to process queries by themselves.

That's how we can use queriers efficiently.

### What is query-scheduler

In 2.4.0 newer version, query-scheduler is released.&#x20;

This is the component that cuts out the query request queue from query-frontend.

Here is how it works.

![How to schedule queries with query-scheduler](../.gitbook/assets/query\_scheduler\_v2.drawio.png)

Query-schedulers have each request queue.&#x20;

At first, a query-frontend receives a query request and enqueues it to a request queue.

And then, the query is sent to a query-scheduler and enqueued in that.

All of queriers observe the queues in query-schedulers and they get queries from them and process.

That's how a dependency between the query-frontend and querier has gone away.

### Why do we need the query-scheduler?

Older architecture has an issue with scaling.&#x20;

Each querier has the configured number of connections to query-frontends.&#x20;

It is configured by "querier.worker-parallelism" or "querier.max-concurrent".&#x20;

It means that query-frontend can't scale more than the parameter.&#x20;

However, the new architecture with query-scheduler allows us to scale query-frontends regardless of the queriers.&#x20;

The query-frontend doesn't depend on them anymore.