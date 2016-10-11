---
layout: post
title:  "Oracle AWR generation using Python"
date:   2015-05-23 15:34:11
categories: python
comments: True
---
I had to implement a job that involved saving the Oracle performance stats to CSV files on a disk. It was an interesting problem to say the least because speed was a consideration and obviously any technology could be choosen. But I choose python, mainly because python (along with most common libraries) comes automatically shipped on most commercial linux servers.

The full souce code is available [at this link] (https://bitbucket.org/varmarakesh/fullstack/src/0c08ed6ea9e8a7c02ba702775725f60e37dddde4/python/awr/?at=master)

#Oracle AWR

AWR (automatic workload repository) collects and stores all runtime stats generated in Oracle database. It is an effective tool used by database community to troubleshoot and address performance problems. In short all the data related to application sql's, connections, resource usages, etc is stored in Oracle and is accessible thru a number of views (DBA_HIST_). Since this data tends to be historical, it is grouped by a certain interval and it is known as snapshot. 

This gives the list of snapshot Id's that would group all the stats generated today.  Usually the interval time would be every 30 min or 1 hour.
{% highlight sql %}
SELECT 
	distinct SNAP_ID, 
	to_char(END_INTERVAL_TIME, 'hh24:mi_dd_mon_yy') as "END_INTERVAL_TIME" 
FROM 
	DBA_HIST_SNAPSHOT 
WHERE 
	TRUNC(END_INTERVAL_TIME) = TRUNC(SYSDATE) 
ORDER BY 
	SNAP_ID
{% endhighlight %}

Once there is the list of snapshot Id's, we can get the runtime stats for all the sql's in the specific snapshot using this query like this.
{% highlight sql %}
SELECT
                     D.NAME,
                     SS.SNAP_ID,
                     SS.INSTANCE_NUMBER,
                     SS.SQL_ID,
                     ST.SQL_TEXT,
                     MODULE,
                     ACTION,
                     PARSING_SCHEMA_NAME,
                     FETCHES_TOTAL,
                     SORTS_TOTAL,
                     EXECUTIONS_TOTAL,
                     LOADS_TOTAL,
                     PARSE_CALLS_TOTAL,
                     DISK_READS_TOTAL,
                     BUFFER_GETS_TOTAL,
                     ROWS_PROCESSED_TOTAL,
                     CPU_TIME_TOTAL,
                     IOWAIT_TOTAL,
                     ELAPSED_TIME_TOTAL
                    FROM
                     DBA_HIST_SQLSTAT SS,
                     DBA_HIST_SNAPSHOT SP,
                     DBA_HIST_SQLTEXT ST,
                     V$DATABASE D
                    WHERE
                     SS.SNAP_ID = SP.SNAP_ID AND
                     SS.DBID = SP.DBID AND
                     SS.DBID = D.DBID AND
                     SS.INSTANCE_NUMBER = SP.INSTANCE_NUMBER AND
                     SS.DBID = ST.DBID AND
                     SS.SQL_ID = ST.SQL_ID AND
                     SS.SNAP_ID = :SNAP_ID
{% endhighlight %}

#Save Results to a CSV File

So, basically need to get the details snapshot details iteratively and save the results to a local CSV File. I used the csv module in python for it.
{% highlight python %}
import csv
conn = getOracleConnection()
        cursor = conn.cursor()
        cursor.execute(q_awr, [snap['snap_id']])
        with open(path, "wb") as csv_file:
            csv_writer = csv.writer(csv_file)
            csv_writer.writerow([i[0] for i in cursor.description]) # write headers
            csv_writer.writerows(cursor)
        cursor.close()
{% endhighlight %}

#Multi-threading

So, the need is issue Oracle queries multiple times (once every snapshot) and save the results in multiple CSV files (once file per snapshot). This is a classic IO bound problem, because, the program issues a database call, waits for the response, save the data to disk, then makes another call and so on. So, I implemented threading and you can in the end that the performance gain using threading is pretty amazing.

So, get a list of snapshots, then spawn a thread for each snapshot. Each thread spawn invokes a method called save_awr to get the snapshot details and saves the results to a file.

{% highlight python %}

def generate_awr_multithreads():
    snapshots = get_snapshots()
    threads = []
    for requestcount in range(snapshots.__len__()):
        t = threading.Thread(target = save_awr, args=[snapshots[requestcount]])
        threads.append(t)
        t.start()

    for t in threads:
        t.join()
{% endhighlight %}

This program saves the results to file system.

{% highlight bash %}
$ ls -lrt AWR*.csv
-rw-r--r--  1 rakesh.varma  staff    86K May 25 12:08 AWR_DLINKD_5074_10:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    85K May 25 12:08 AWR_DLINKD_5075_11:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    93K May 25 12:08 AWR_DLINKD_5076_12:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    81K May 25 12:09 AWR_DLINKD_5064_00:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    78K May 25 12:09 AWR_DLINKD_5065_01:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    85K May 25 12:10 AWR_DLINKD_5066_02:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    81K May 25 12:10 AWR_DLINKD_5067_03:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff   196K May 25 12:11 AWR_DLINKD_5068_04:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    87K May 25 12:11 AWR_DLINKD_5069_05:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff   111K May 25 12:12 AWR_DLINKD_5070_06:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    86K May 25 12:12 AWR_DLINKD_5071_07:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    95K May 25 12:13 AWR_DLINKD_5072_08:00_25_may_15.csv
-rw-r--r--  1 rakesh.varma  staff    56K May 25 12:13 AWR_DLINKD_5073_09:00_25_may_15.csv
{% endhighlight %}

If multi-threading could not be implemented, this is how a single threaded version would like.

#Single Thread
{% highlight python %}

def generate_awr_singlethread():
    snapshots = get_snapshots()
    for requestcount in range(snapshots.__len__()):
        save_awr(snap = snapshots[requestcount])
{% endhighlight %}

#Conclusion

Elapsed time differences between multi-threading and single threaded versions are very stark because this is a classic example of an IO bound problem.  Oracle connections and saving the AWR data to disk are two IO bound tasks. Saving the AWR data is very IO intensive given the volume of data. These are the results when I ran this script against my database.
<p> @timefn:generate_awr_multithreads took 58.2649428844 seconds </p>
<p> @timefn:generate_awr_singlethread took 361.62610507 seconds </p>

Please check the source at https://bitbucket.org/varmarakesh/fullstack/src and let me know the feedback. Anybody that has an Oracle client installed, the setup is quite minimal and it will work without issues.