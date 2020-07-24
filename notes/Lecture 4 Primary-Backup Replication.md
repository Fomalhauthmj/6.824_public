**[课程首页](https://pdos.csail.mit.edu/6.824/index.html)**

**[课程安排](https://pdos.csail.mit.edu/6.824/schedule.html)**

##  **Lecture  4:** [Primary/Backup Replication](https://pdos.csail.mit.edu/6.824/notes/l-vm-ft.txt)

### What kinds of failures can replication deal with?

```
  "fail-stop" failure of a single replica
    fan stops working, CPU overheats and shuts itself down
    someone trips over replica's power cord or network cable
    software notices it is out of disk space and stops
  Maybe not defects in h/w or bugs in s/w or human configuration errors
    Often not fail-stop
    May be correlated (i.e. cause all replicas to crash at the same time)
    But, sometimes can be detected (e.g. checksums)
  How about earthquake or city-wide power failure?
    Only if replicas are physically separated
```

### Two main replication approaches:

```
  State transfer
    Primary replica executes the service
    Primary sends [new] state to backups
  Replicated state machine
    Clients send operations to primary,
      primary sequences and sends to backups
    All replicas execute all operations
    If same start state,
      same operations,
      same order,
      deterministic,
      then same end state.
```

### Overview

```
  [diagram: app, O/S, VM-FT underneath, disk server, network, clients]
  words:
    hypervisor == monitor == VMM (virtual machine monitor)
    O/S+app is the "guest" running inside a virtual machine
  two machines, primary and backup
  primary sends all external events (client packets &c) to backup over network
    "logging channel", carrying log entries
  ordinarily, backup's output is suppressed by FT
  if either stops being able to talk to the other over the network
    "goes live" and provides sole service
    if primary goes live, it stops sending log entries to the backup
```

### Output Rule

```
The Output Rule is a big deal
  Occurs in some form in all replication systems
  A serious constraint on performance
  An area for application-specific cleverness
    Eg. maybe no need for primary to wait before replying to read-only operation
  FT has no application-level knowledge, must be conservative
```

```
Q: But what if the primary crashed *after* emitting the output?
   Will the backup emit the output a *second* time?

A: Yes.
   OK for TCP, since receivers ignore duplicate sequence numbers.
   OK for writes to disk, since backup will write same data to same block #.

Duplicate output at cut-over is pretty common in replication systems
  Clients need to keep enough state to ignore duplicates
  Or be designed so that duplicates are harmless

Q: Does FT cope with network partition -- could it suffer from split brain?
   E.g. if primary and backup both think the other is down.
   Will they both go live?

A: The disk server breaks the tie.
   Disk server supports atomic test-and-set.
   If primary or backup thinks other is dead, attempts test-and-set.
   If only one is alive, it will win test-and-set and go live.
   If both try, one will lose, and halt.
   
The disk server may be a single point of failure
  If disk server is down, service is down
  They probably have in mind a replicated disk server
```

