## MySQL Background

When a data manipulation query is sent to the RMDB engine, it goes through syntax parsing, writing to binlog, and persisting to disk. For example, 

```sql
SELECT first_name FROM students where last_name="Azure";
```

```java
students.getAll().stream()
    .filter(student -> student.getLastName()=="Azure")
    .map(Student::getFirstName)
```

```sql
SELECT COUNT(*) FROM students GROUP BY last_name;
```

```java
students.getAll().stream()
    .collect(Collectors.groupingBy(student -> student.getLastName(), Collectors.counting()));
```

The binlog is a record log of all transactions commited. 

![innodb-architecture.png (700×537)](https://dev.mysql.com/doc/refman/5.7/en/images/innodb-architecture.png)

## DURABILITY AT SCALE

There are numbers of benefits to replicate data across nodes. There are two principles of writing to/ reading from the node cluster. 

- We should write to more than half of nodes
- We should read from enough nodes to gurantee the data is up to date.

## THE LOG IS THE DATABASE

The key is to make the persistence asynchronous. The synchronous call was an old design for standalone MySQL databases. But if we synchronize the binlog, not the database, the workflow will be simpler. 

MySQL mirrors the binlog changes to the replicas. It waits the data to be persisted in disks before acknoledging the queries. 

The Aurora design brings us a benefit that, the process of crash recovery is spread across all normal foreground processing. Nothing is required at database startup. 

Aside from it, the storage design reduces the latency, because the queries do not wait the acknoledges from the disks. 

## THE LOG MARCHES FORWARD

### Asynchronous Processing

If the DML queries do not wait the disk acknowledges, how do we achieve consensus among the replicas? 

Aurora uses the Log Sequence Number (LSN) to keep the transactions serialized. The LSN is also used for recovery. 

Now let's exam the normal operations on Aurora. 

- Write: read from the disk, apply the transaction, flush the dirty page to disk
- Commit: mark the transaction, not the table entry as accepted. Return 200 to the client. 
- Read: if the entry is not in the cache, read from the disk. Apply the transaction records. Return to the client
- Replicate: (a) the only log records that will be applied are those whose LSN is less than or equal to the VDL, and (b) the log records that are part of a single mini-transaction are applied atomically in the replica's cache 

### Recovery

When we recover a node, we do not replicate the snapshot of the master node. Actually, we reconstruct the database using the binlog. 
