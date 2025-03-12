# Options to generate unique IDS in distributed systems:

- Multi-master replication
- Universally unique identifier (UUID)
- Ticket server
- Twitter snowflake approach

## 1. Multi-master replication

Uses databases auto_increment feature. This approach uses the databases auto_increment feature. Instead of increasing the next ID by 1, we increase it by K, where K is the number of database servers in use. As illustrated in the figure above, the next ID to be generated is equal to the previous ID in the same server plus 2. This solves some scalability issues because IDs can scale with the number of database servers.

![Multi-master replication](2020250125145601.png)

However, this strategy has some major drawbacks:
- Hard to scale with multiple data centers
- IDs do not go up with time across multiple servers
- It does not scale well when a server is added or removed

## 2. UUID

A UUID is another easy way to obtain unique IDs. UUID is a 128-bit number used to identify information in computer systems.

![UUID](2020250125150435.png)

In this design, each web server contains an ID generator, and a web server is responsible for generating IDs independently.

**Pros:**
- Generating UUID is simple. No coordination between servers is needed so there will not be any synchronization issues.
- The system is easy to scale because each web server is responsible for generating IDs they consume. ID generator can easily scale with web servers.

## 3. Ticket Server

The idea is to use a centralized auto_increment feature in a single database server (Ticket Server).

![Ticket Server](2020250125150840.png)

**Pros:**
- Numeric IDs
- It is easy to implement, and it works for small to medium-scale applications.

**Cons:**
- Single point of failure. Single ticket server means if the ticket server goes down, all systems that depend on it will face issues. To avoid a single point of failure, we can set up multiple ticket servers. However, this will introduce new challenges such as data synchronization.

## 4. Twitter snowflake ID

![Twitter snowflake ID](2020250125151613.png)

**ID Sections:**
- Sign bit: 1 bit. It will always be 0. This is reserved for future uses. It can potentially be used to distinguish between signed and unsigned numbers.
- Timestamp: 41 bits. Milliseconds since the epoch or custom epoch. We use Twitter snowflake default epoch 1288834974657, equivalent to Nov 04, 2010, 01:42:54 UTC.
- Datacenter ID: 5 bits, which gives us 2^5 = 32 datacenters.
- Machine ID: 5 bits, which gives us 2^5 = 32 machines per datacenter.
- Sequence number: 12 bits. For every ID generated on that machine/process, the sequence number is incremented by 1. The number is reset to 0 every millisecond.

**Pros:**
- Chronological ordering
- Smaller than UUIDs (128 bits vs 64 bits)
- Systems that need partitions (shards) can use the Snowflake ID to identify which node or partition the data was generated from.

**Cons:**
- If the server clock is inaccurate (e.g., skewed or adjusted backward), it may cause issues with ID ordering.
- If a node generates more IDs than the 12-bit counter can support (4,096 IDs per millisecond), collisions may occur.
- The 10-bit node ID must be carefully managed to avoid accidental duplication across nodes.
