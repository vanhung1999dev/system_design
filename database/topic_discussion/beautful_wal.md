# Beautiful WAL

In any database system there are often two major storage units, data and indexes. Data represents tables, collections or graphs. Indexes represents fast-path lookups to data often created with B+Tree data structure. <br>

## Pages

Both data and indexes live in fixed-size pages in data files on disk. Databases may do it differently but essentially every table/collection is stored as an array of fixed-size pages in a file. The database can tell which offset to seek to based on the page number and the page size is used to know how many bytes to read. This allows the database to issue an operating system <br>

```
 #include <unistd.h>

       ssize_t pread(int fd, void buf[.count], size_t count,
                     off_t offset);
```

That single I/O to read a page will get you not one row but many rows. That is because a page will have many rows/documents as part of its internal structure. And similarly, to update a field in one row in the page you may have to flush the entire page to disk even though only few bytes might have changed. <br>

This is key to understanding database performance is this: You want pages to remain “dirty” (ie pages that have been written to) as long as possible so hopefully receive a lot of writes so we can flush it once to disk to minimize I/O. You don't want to flush the page to disk (with a write) with every update you make. <br>

## WAL

I can start a transaction that updates a row in page 0, inserts a new row in page 10 and deletes a row from page 1 and to commit my transaction I can write the three pages to disk. Two problems with this approach. Small writes lead to massive I/Os which slows down commits. The second problem and the worse is if the database crashed midway of writing the pages, we have some pages written and some are not and we lose atomicity. <br>

In addition we have no reference on startup after crash to remove pages belonging to transactions that crashed and must be rollbacked as we don’t even know what the original page looked like. <br>

Meet WAL or write-ahead log. How about we do this instead? In addition to updating pages in memory we also record the changes transactions make in an append log and we flush the log to disk when the transaction commits. We know flushing the log is going to be fast because the changes are much smaller than writing the full pages of data files and indexes. And yes we can safely leave the updated data pages in memory and not flush them to disk saving massive I/Os, I’ll explain why later. <br>

By “writing-ahead” of the data files in this log file, the data files on disk are technically left behind and older than what is in the log file, this is where the name write-ahead in WAL comes from. <br>

## Checkpointing

Wait what? data files are out of sync? what if we crashed? Before we crash and go ballistic let me say most databases don’t just crash, 99% cases are when things go well, so let us explain when things are rosy first. <br>

Technically speaking we are consistent in memory because we do write to pages as we write the WAL and because we always read and write from and to memory first we get the latest and greatest. So data files and indexes pages are up to date in memory but they are out of date on disk. <br>

So in a happy path, I issue a commit and my WAL is flushed successfully to disk and the database returns OK to the client (me) who issued the commit. So the WAL is updated. Then moments later the database start flushing my full pages to disk and now both data files and WAL are in sync on disk. We call this a checkpoint and the process that creates this checkpoint is checkpointing. The database will actually create a checkpoint record that says, I hereby declare at this moment in time, everything written to WAL on this moment is completely in SYNC with what we have on data files on disk. <br>

```
Checkpointing kicks in often by the database to make sure crash recovery is faster (we will explain that next) but it is I/O intensive can you guess why? Well because of all the full pages writes that we must do.
```

## When databases crash

Ok I issue a commit and my WAL is flushed successfully to disk and I return OK to the client who issued the commit. Then moments later the database crashed losing all my fat and juicy in-memory pages which had my changes. The database restarts and now we have the old pages in files which are old! We CANNOT start the database in this state otherwise transactions will be reading old data. Here is what happens. <br>

When the database restarts it finds the last checkpoint where the WAL and data files were completely in sync, we can trust that the pages are in sync up to that point. After that point we know the WAL and the data files are out of sync. So what the database does is take the WAL records after the checkpoint and start “redoing” the changes to the data files again. Why is it called redo? because remember we already DID apply those changes to the pages in memory but we lost them. So we have to Redo them. That is why the WAL is also known as redo log, much better name than write-ahead log if you ask me. <br>

That is why if we don’t run checkpoints often, in case of a crash, the database has to do so much more work in recovery to redo all the changes to the files. You probably also have a large WAL which isn’t infinite, there is a fixed size WAL. And Oh I forgot to mention, when we do a checkpoint, we can technically safely purge the WAL because we know all changes are applied to the data files. But of course we can archive the WAL somewhere else for history purposes. <br>

```
I know what you’re thinking, what if we crashed midway through writing the WAL? That is also fine because we know exactly what transactions wrote those WAL changes and upon startup we will clean up the WAL records that belong to transactions that didn’t commit, we effectively roll it back.
```

## Undo logs

Ok. Here is another case for you, a transaction starts and writes new rows, updates some existing rows the data pages in memory and the WAL has those new changes. However the transaction fails and rollbacks, the data files in memory now has writes from a transaction that has rolledback!, we need to restore the original state of the row to a state that was good. <br>

The WAL only has changes, it has no clue what the original rows looked like. So for that we need another log to record the original state of the row before we made the change and that is the undo log. <br>

Undo log will be used to "undo" the changes made by the rolledback transaction, and restore the rows to original state. <br>

Undo logs are also useful for crashes, suppose the database crashed after flushing the data pages which has changes of a rolledback transaction or an in progress transactions. Upon restart, those changes should be undone. This is where the database first replays the undo logs, then follow it up with the WAL (redo log) Until we get to a state where we are completely consistent. <br>

The undo logs are also used by transactions that need MVCC visibility to the old version of the row. This means having a long running transaction that does massive changes may slow down some isolation level read transactions as those have to go to the undo log and restore the original row before use. Not to mention the constant need to construct the state of the original rows. <br>

Luckily Postgres architecture doesn't have undo logs because of how versioning is done <br>

## The WAL tells a story

The WAL has all the changes since epoch (the start). Technically speaking if your data files are empty and you have the WAL archive for your entire database you can replay the changes and construct the final state of the database. That is why the WAL is powerful concept for replication too as it is enough to stream the WAL changes to replicas so they can replay it back on their version of files. Assuming they all started from a well known state. <br>

## Destroy the WAL

Explaining what something is rarely works for me and I imagine this might be true for others. It is like putting the cart before the horse. Something inside you rejects the idea of what when stated first. That is why I prefer to explain the problem first (the why), then add the solution and only then explain what that solution is. <br>

In doing that, I simply outline the problem and I give you the reader a freedom of choice, a freedom to reject this idea, a freedom to come up with your own solution. <br>

Nothing is perfect. <br>

One of you might look at the problem and come up with a different solution that no one of the clever computer scientists and engineers have thought about. I want you to challenge this. This is what we have today, you don’t have to accept it. <br>

Destroy the WAL if you wish. Or climb it. Your choice. It is useful but it doesn’t mean you have to follow what has been done. <br>
