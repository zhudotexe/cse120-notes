Memory
======

- Static RAM (SRAM)
    - 20MB/chip, 0.2-2.5ns, >$100/GB
- Dynamic RAM (DRAM)
    - 500MB/chip, 50-70ns, ~$5/GB
- Flash (non-volatile)
    - 16GB/chip, 20us-5ms, ~$0.50/GB
- Magnetic Disk (nonv, mechanical)
    - >1TB/disk, 5-20ms, $0.05/GB

Big mems are cheap but slow, and small mems are fast but expensive

so we need a memory hierarchy!

- Store everything on a slow mem
- Copy recently accessed & nearby data to smaller DRAM memory
    - DRAM is called main memory
- Copy more recently accessed & nearby data to smaller SRAM memory
    - Called the cache

.. image:: _static/memory/hierarchy.png

Terminology
-----------
- Hit – accessed data found at “upper” level
    - hit rate: fraction of accesses to upper level that finds the data
    - hit time: time to access data on a hit

- Miss: accessed data found at “lower” level of the hierarchy
    - processor typ. waits until block fetched into upper level
    - miss rate: 1 – hit rate
    - miss penalty: time to get block from lower level and satisfy request

- local hit rate vs global hit rate

- hit time << miss penalty

Locality
--------

- principle of locality
    - programs work on a relatively small portion of data at any time
    - can predict data accessed in the near future by looking at recent accesses

- temporal locality
    - if an item has been referenced, it will probably be referenced again soon
    - probability of accessing A again at time t+x highest as x approaches 0

- spatial locality
    - if an item has been accessed, nearby items will tend to be referenced soon
    - probability of accessing A+x at time t highest as x approaches 0

Caches
------

- holds recently referenced data
    - Functions as a buffer for larger, slower storage components
- Exploits principle of locality
    - Provide as much inexpensive storage space as possible
    - Offer access speed equivalent to the fastest memory
        - For data in the cache
        - Key is to have the right data cached
- Computer systems often use multiple caches
- Cache ideas are not limited to hardware designers
    - Example: Web caches widely used on the Internet

Questions
^^^^^^^^^
- Where can I store a particular piece of data? (mapping)
    - Direct mapped (single location)
    - Fully associative (anywhere)
    - N-way set associative (anywhere in a set of size N)
- What do I throw out to make room? (replacement policy)
- How much data to I move at a time? (block size or cache line size)
- How do we handle writes?
    - Bypass cache
    - Write thru the cache
    - Write into the cache – and then write back

Direct Mapped
^^^^^^^^^^^^^

- location in cache determined by main memory address
- cache address = (memory address) mod (cache size)
- but we need to remember what memory address is in a given cache slot
    - store the higher order bits "tag"
    - and also a valid bit in case the cache was never written to

