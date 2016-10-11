---
layout: post
title:  "High Availability for MongoDB"
date:   2015-08-01 15:34:11
categories: mongodb
comments: True
---
MongoDB is a no-sql database that is very easy to setup and administer.

High availability is achieved using replica sets that provide automatic fail-over. It is important to note that fail-over is automatic, in that it does not require manual interaction. 

In this post we examine how replica sets are configured and how automatic failover is achieved. This assumes that mongodb is already installed.

In order to understand replica sets it is important to understand the mongod process. mongod is the process that listens for incoming requests on a certain port and process those requests.  If there is only one mongod process involved, then it is a single point of failure, however to incorporate failover using replica sets we create multiple mongod process and designate them as primary, secondary and arbiter.

*	Primary -> primary member handles all write operations. It is the current master instance.
*	Secondary -> it is a replica set member that replicates the contents of the master database. 
*	Arbiter -> the purpose of arbiter is to solely vote in elections. Arbiter does not replicate data.

Create 3 mongod instances, once each for primary, secondary and arbiter
{% highlight bash %}
cd /data
mkdir db1
mkdir db2
mkdir db3
mongod --dbpath ./db1 --port 30000 --replSet "demo" --fork --logpath /var/log/mongodb.log
mongod --dbpath ./db2 --port 40000 --replSet "demo" --fork --logpath /var/log/mongodb.log
mongod --dbpath ./db3 --port 50000 --replSet "demo" --fork --logpath /var/log/mongodb.log
{% endhighlight %}

At this points, 3 instances are created. It could be verified.
{% highlight bash %}
ps -ef|grep -i 'mongod'

root      4760     1  0 04:13 ?    00:00:00 mongod --dbpath ./db1 --port 30000 --replSet demo --fork --logpath /var/log/mongodb.log
root      4892     1  0 04:14 ?    00:00:00 mongod --dbpath ./db3 --port 50000 --replSet demo --fork --logpath /var/log/mongodb.log
root     20490     1  0 01:56 ?    00:01:01 mongod --dbpath ./db2 --port 40000 --replSet demo --fork --logpath /var/log/mongodb.log
{% endhighlight %}

Now, lets define which instance is primary, secondary and arbiter.

Connect to one instance.
{% highlight bash %}
mongo --port 30000
db.getMongo()
{% endhighlight %}

At this point, we are in mongo shell, now we create a javascript that a json object that defines the primary, secondary and arbiter.Assigning a priority of 10 means it is primary. After it is defined then rs.initiate is run

{% highlight javascript %}
var demoConfig = {
	"_id" : "demo",
	"members" : [
			{
				"_id" : 0,
				"host" : "192.200.15.72: 30000",
				"priority" : 10
			},
			{
				"_id" : 1,
				"host" : "192.200.15.72: 40000"
			},
			{
				"_id" : 2,
				"host" : "192.200.15.72: 50000",
				"arbiterOnly" : true
			}
		]
}
rs.initiate(demoConfig)
{% endhighlight %}

At this point, replica set is initiated. Now will create a collection in primary and verify it was replicated in secondary.
{% highlight javascript %}
use test
db.phonecountrycodes.save({_id:1, country:'USA', code:001})
db.phonecountrycodes.find()
{% endhighlight %}

Now exit and connect to secondary. 
{% highlight javascript %}
mongo --port 40000
{% endhighlight %}
Check the data in created collections, it will give an error initially because secondary needs to be set as slave.
{% highlight javascript %}
db.phonecountrycodes.find()
db.setSlaveOk()
db.phonecountrycodes.find()
{% endhighlight %}

To test the automatic failover, from another window kill the primary and watch the prompt changes to primary of the open mongo shell. Through the use of polling algorithm arbiter detects primary is down and makes secondary as primary.
{% highlight bash %}
ps -ef|grep -i 'mongod'
kill xxxxx
{% endhighlight %}

If you can run a python script, I suggest to perform this test using python. It will help you understand, how application reacts to automatic fail over.

Install pymongo and run this program.
{% highlight python %}
from pymongo import *
from time import *

while (True):
    sleep(3)
    client = MongoClient(['192.200.15.72:30000','192.200.15.72:40000', '192.200.15.72:50000'], replicaSet='demo')
    db = client['test']
    code = db.phonecountrycodes.find_one()
    print client.primary
    client.close()

{% endhighlight %}

While this program is running, in another window, kill the mongod primary process and restart it after a few seconds.

{% highlight python %}
kill xxxxx
mongod --dbpath ./db1 --port 30000 --replSet "demo" --fork --logpath /var/log/mongodb.log
{% endhighlight %}

Clearly, you can see the primary output of the python program alternate between 192.200.15.72:30000 and 192.200.15.72:40000 very seamlessly.
{%highlight python %}
/System/Library/Frameworks/Python.framework/Versions/2.7/bin/python /Users/rakesh.varma/PycharmProjects/dtv_test/pymongotest.py
('192.200.15.72', 30000)
('192.200.15.72', 30000)
('192.200.15.72', 30000)
('192.200.15.72', 30000)
('192.200.15.72', 30000)
('192.200.15.72', 30000)
('192.200.15.72', 40000)
('192.200.15.72', 40000)
('192.200.15.72', 40000)
('192.200.15.72', 40000)
('192.200.15.72', 40000)
('192.200.15.72', 40000)
('192.200.15.72', 30000)
('192.200.15.72', 30000)
('192.200.15.72', 30000)
{% endhighlight %}

No disruption is observed by the application. 





