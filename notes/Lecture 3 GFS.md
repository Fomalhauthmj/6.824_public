**[课程首页](https://pdos.csail.mit.edu/6.824/index.html)**

**[课程安排](https://pdos.csail.mit.edu/6.824/schedule.html)**

##  **Lecture  3:** [GFS](https://pdos.csail.mit.edu/6.824/notes/l-gfs.txt)

### Why is distributed storage hard?

```
  high performance -> shard data over many servers
  many servers -> constant faults
  fault tolerance -> replication
  replication -> potential inconsistencies
  better consistency -> low performance
```

### GFS

#### Overall structure

```
  clients (library, RPC -- but not visible as a UNIX FS)
  each file split into independent 64 MB chunks
  chunk servers, each chunk replicated on 3
  every file's chunks are spread over the chunk servers
    for parallel read/write (e.g. MapReduce), and to allow huge files
  single master (!), and master replicas
  division of work: master deals w/ naming, chunkservers w/ data
```

#### Master state

```
  in RAM (for speed, must be smallish):
    file name -> array of chunk handles (nv)
    chunk handle -> version # (nv)
                    list of chunkservers (v)
                    primary (v)
                    lease time (v)
  on disk:
    log
    checkpoint
```

#### What are the steps when client C wants to read a file?

```
  1. C sends filename and offset to master M (if not cached)
  2. M finds chunk handle for that offset
  3. M replies with list of chunkservers
     only those with latest version
  4. C caches handle + chunkserver list
  5. C sends request to nearest chunkserver
     chunk handle, offset
  6. chunk server reads from chunk file on disk, returns
```

#### What are the steps when C wants to do a "record append"?

```
  paper's Figure 2
  1. C asks M about file's last chunk
  2. if M sees chunk has no primary (or lease expired):
     2a. if no chunkservers w/ latest version #, error
     2b. pick primary P and secondaries from those w/ latest version #
     2c. increment version #, write to log on disk
     2d. tell P and secondaries who they are, and new version #
     2e. replicas write new version # to disk
  3. M tells C the primary and secondaries
  4. C sends data to all (just temporary...), waits
  5. C tells P to append
  6. P checks that lease hasn't expired, and chunk has space
  7. P picks an offset (at end of chunk)
  8. P writes chunk file (a Linux file)
  9. P tells each secondary the offset, tells to append to chunk file
  10. P waits for all secondaries to reply, or timeout
      secondary can reply "error" e.g. out of disk space
  11. P tells C "ok" or "error"
  12. C retries from start if error
```

#### Summary

```
  case study of performance, fault-tolerance, consistency
    specialized for MapReduce applications
  good ideas:
    global cluster file system as universal infrastructure
    separation of naming (master) from storage (chunkserver)
    sharding for parallel throughput
    huge files/chunks to reduce overheads
    primary to sequence writes
    leases to prevent split-brain chunkserver primaries
  not so great:
    single master performance
      ran out of RAM and CPU
    chunkservers not very efficient for small files
    lack of automatic fail-over to master replica
    maybe consistency was too relaxed
```