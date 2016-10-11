---
layout: post
title:  "Oracle Cascade Delete - Always better than explicit child table deletes"
date:   2015-05-16 15:34:11
categories: oracle
comments: True
---
Throughout all applications, there are many areas where we enforce basic referential integrity, but various applications handle differently the scenarios when it comes to deleting the child records in the event of the parent record delete.  Oracle, like many other databases, offers referential integrity with on delete cascade option. When child tables are created with foreign keys that have the 'on delete cascade' option, then deletion of parent record will automatically delete the records in all child tables.
The other option to delete the child table records is to, explicitly delete the records in all child tables. This article discusses the POC of both approaches. Oracle run time stats have been gathered for both approaches. If you want to try please use sqlplus.
 
##### POC Sample (Cascade Delete vs manual deletion of child table records)

*	parent = parent
*	children = child1, child2, child2.
*	Tests conducted on exadata, version = Oracle enterprise version, 11.2.0.4.0 - 64bit

##### No On Cascade Delete Option #
Without using 'on delete cascade' option, the application needs to perform a explicit delete of all child tables.

{% highlight sql %}
SET AUTOTRACE ON
SET TRANSACTION NAME 'POC_NO_CASCADE_DELETE'
DELETE FROM CHILD1 WHERE ID = 100;
DELETE FROM CHILD2 WHERE ID = 100;
DELETE FROM CHILD3 WHERE ID = 100;
SELECT USED_UBLK FROM V$TRANSACTION WHERE NAME='POC_NO_CASCADE_DELETE';
ROLLBACK;
{% endhighlight %}


##### On Cascade Delete Option

Using this deletion option, need to specify 'on delete cascade' when creating the constraint for each child table.

{% highlight sql %}
ALTER TABLE CHILD1 ADD CONSTRAINT FK01_CHILD1 FOREIGN KEY (ID) REFERENCES PARENT (ID) ON DELETE CASCADE ENABLE;
ALTER TABLE CHILD2 ADD CONSTRAINT FK01_CHILD2 FOREIGN KEY (ID) REFERENCES PARENT (ID) ON DELETE CASCADE ENABLE;
ALTER TABLE CHILD3 ADD CONSTRAINT FK01_CHILD3 FOREIGN KEY (ID) REFERENCES PARENT (ID) ON DELETE CASCADE ENABLE;
SET TRANSACTION NAME 'POC_CASCADE_DELETE'
SET AUTOTRACE ON;
DELETE FROM PARENT WHERE ID = 100;
SELECT USED_UBLK FROM V$TRANSACTION WHERE NAME='POC_CASCADE_DELETE';
{% endhighlight %}

Once the constraints with 'on delete cascade' option is in place, deleting the parent record while automatically remove the child record.

Oracle Stat  			| No Cascade Delete | Cascade Delete
------------------------|-------------------|---------------
------------------------|-------------------|---------------
Total Recursive Calls  	|426 				|363
Total db block gets 	|19					|19
Total redo				|3224				|2956
Total undo blocks		|3 					|1


 
##### Results Summary
1. There is no performance penalty using the 'On delete cascade'. Oracle stats perform well on all areas with 'Cascade Delete' option.
2. There is a significant improvement of undo tablespace usage when using the 'Cascade Deletion' option. Oracle is using at least 1 block for undo 	associated with each DML statement. However, when deletes are implicit using cascade deletes, the undo usage for all deletes is just 1 undo block. This is an important observation because when benchmarking the results of deletes, undo and redo are perhaps the most important stats.
3. Using 'On Delete Cascade' , the application code (data models) is very easy to maintain. The purge scripts are also easier to maintain. We don't need to track the order of the child table deletes. Importantly, when table model change or new child tables are created, the application code and purge scripts do not need to updated.