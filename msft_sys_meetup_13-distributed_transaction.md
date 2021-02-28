## Recap of Database Transaction
ACID (atomicity, consistency, isolation, durability). Take the money transfer between two bank accounts for example.

Initially, there are \$100 in each account. 
| id   | balance |
| ---- | ------- |
| 1    | 100     |
| 2    | 100     |

```sql
begin;
update accounts set balance=balance+200 where id=2;
update accounts set balance=balance-200 where id=1; -- this statement will fail
commit;
```
Atomicity means the above two statements in one transaction will both rollback given the failure of the second statement. 

## Distributed Transaction Solutions
X/Open XA standard (short for "eXtended Architecture")

![xa](https://user-images.githubusercontent.com/11760687/109410635-29bb4380-79d7-11eb-8170-630b152098c8.png)

Two-phase commit, as a generalization of the XA. 

![two_phase_commit_service](https://user-images.githubusercontent.com/11760687/109410638-2f188e00-79d7-11eb-8a3f-c3ce6b9bbc5f.png)

