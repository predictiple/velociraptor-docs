---
menutitle: "pool_client"
title: 'The "pool_client" command'
date: 2026-09-09
last_reviewed: 2026-09-09
draft: false
weight: 67
summary: "Run a pool client for load testing"
description: |
  Run a pool client for load testing. The pool client simulates many
  clients on a single host, remembering query results so each query
  runs only once while every simulated client reports the results.
---

The `pool_client` command simulates many Velociraptor clients on a
single host. It is mainly used for load testing a Velociraptor
deployment: you can spin up hundreds or thousands of virtual clients
to see how the server copes with the extra load.

Each virtual client connects to the server with its own client ID,
so the server sees a fleet of independent clients. However, all the
virtual clients run on the same host, so the pool client avoids
running the same query over and over again on that host.

---

### [ pool_client ]

```text
pool_client [<flags>]
    Run a pool client for load testing.

    --number=NUMBER    Total number of clients to run.
    --writeback_dir=WRITEBACK_DIR
                       The directory to store all writebacks. If not
                       specified, in-memory writebacks are used.
    --concurrency=10   How many real queries to run.
    --start_rate=20    How many clients per second to start.
```

#### How the pool client handles collections

The pool client remembers the results of every collection it runs.
When a virtual client is asked to run a collection, the pool client
first checks whether it has already run the same collection before
(the same artifact with the same parameters). If it has, it reports
the saved results again instead of running the collection again. If
it has not, it runs the collection and saves the results.

This means each collection is only ever actually run once on the
host, no matter how many virtual clients receive it. Each client
then reports the results to the server as if it had run the
collection itself.

In practice this means:

- **A single collection** runs on the client you chose. A collection
  sent to one specific client is unique to that client, so it runs
  normally.
- **A hunt** runs once and the results are reused for every client
  in the hunt. Since all virtual clients share the same host, a
  hunt that targets all clients returns the same results for each
  one.
- **Running the same collection again** on another client returns
  the same results as the first time.
- **The initial interrogation** is shared by all clients. The first
  client to enroll runs `Generic.Client.Info` and the rest reuse its
  results. Each client then reports its own codename hostname, while
  the rest of the host information is the same for every client.

This behavior is intentional. The pool client is meant to put load on
the server (communications, datastore, GUI), not to run the same
query hundreds of times on the host. Because all virtual clients run
on the same host, an identical collection from any of them returns
the same data, so reusing the results is both correct and efficient.

The pool client also limits how many collections it runs at the same
time. By default it runs up to 10 collections in parallel (the
`--concurrency` flag). If more collections are requested, they wait
until a running collection finishes. This stops the pool client from
overwhelming the host with too many queries at once. If you run many
virtual clients, you may need to raise this value so collections and
hunts finish quickly. Be careful though: each parallel query uses
host resources, so raising it too far can slow the host down.

To distinguish the virtual clients in the GUI, each one gets a unique
codename hostname made from an adjective and a noun, for example
`brave-falcon` or `quantum-phoenix`. The codename is also used as the
fully qualified domain name, reusing the domain of the real host. So
on a host called `myhost.example.com`, a virtual client might report
its FQDN as `brave-falcon.example.com`. The rest of the reported
client information reflects the real host.

#### How the pool client stores client information

Each virtual client needs its own client ID and private key. By
default the pool client keeps these in memory, so nothing is written
to disk and every run starts with fresh clients. To keep the same
clients between runs, use the `--writeback_dir` flag to store the
client information in a directory on disk. The codename hostnames are
generated fresh each time the pool client starts.

###### Example

```sh
velociraptor pool_client --config client.config.yaml --number 100
```

This starts 100 virtual clients using the client configuration in
`client.config.yaml`. The `--start_rate` flag controls how quickly
the clients start, and the `--concurrency` flag controls how many
collections can run at the same time.