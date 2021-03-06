## How this system looks like?

Twitter is one of the biggest systems that users can share and update status. Additionally, most of the users can easily update their 
status via Tweets. Moreover, users can easily search Tweets whatever they want. Notice that this system is so huge and because of that 
system should design very carefully.

## Requirements

There are three types of requirements and these are functional, non-functional and extended requirements

### Functional Requirements

Users can search the tweets whether they are registered or not. (Public or private accounts are important). When users search the keys, 
these keys can consist of one or two words. Because of that system should bring all of the possible suggestion occurring "AND" and "OR". 
Users can delete their tweets when these tweets deleted, the system should not suggest these tweets.

### Nonfunctional Requirements

The system should be highly available. This means that any user must be able to reach the system whenever they want. This is provided by
using the replication process. The system should have at least three replicas for each server including application servers. Additionally,
replication helps system to decrease the response time and traffic-heavy( we can use a load balancer that rotated requests.)

The system should answer the request with minimum latency. This means that system should be real-time systems. This is provided via the 
sharding. Sharding means that data are kept different database servers. Notice that this system exists if system saves the data using the 
MongoDB. These parts will be explained below.

The system should store the data efficiently. Because of that, we can select the NoSQL rather than the RDBMS. Notice that NoSQL can have
horizontal partition easily(sharding.). however if RDMBS is selected to use, the horizontal partition is so hard. (vertical sharding can 
use.)
One more thing that, if the system should be so reliable and tables have more relations then you use RDBMS such as MySql, PostgreSQL or 
Mssql because RDMS support the Transactional operations (ACID). On the other hand, NoSQL works in RAM to provide increasing time speed 
and NoSQL work with CAP (BASE) theorem.

### Extended Requirements

The system should be monitored. This means that, if server count is not enough and any server fails then the system should be alerted. 
Notice that we can use ManageEngine "Opmanager" to monitoring. You can monitor the CPU Utilization, disk utilization and memory 
utilization via "opManager."

## Capacity and Estimations

We should determine the number of the users. Let's assume that, Twitter has approximately 2 billion users and 500 million daily active 
users. Also assume that, Twitter gets 500 million tweets in each day. We can easily call that every tweets is approximately 300 bytes. 
Additionally, we should determine the number of searches in every day and let's say that search number is 500 million. With above 
informations, 500 million each tweets * 300 bytes = 15 * 10^10 = 150 GB each day. Because of that if we calculate the system capacity 
after the 10 years, this amount is so huge, because of that we should prefer to NoSQL. Additioanlly, if we decide to use the caching 
system. Caching mechanism rule is 80-20. So 150 GB each day / 5 = 30 GB for caching. Notice that moderns servers can store more than 
30 GB. We can easily store the cache datas in one server.

## System API's

We can use the SOAP or REST APIs to service the datas in our system. But if we think about the modern technologies and mobile applications
, usage of REST API's is better than the SOAP. Because of that, we can select the REST API's. Notice that we have one API which is 
basically called the search.

* Search(api_keys, search_terms, suggested_results_count, sort).

Notice that, api_dev_keys is used for the registered account. This will be used for determining the quotas. Additionally, search_terms
includes the "AND" and "OR" results.

The "search API's" return the suggested results via JSON formats. These results are returned with UserID, userName, text, creation time,
number of likes, number of dislikes etc..

## Beginning High Level System Design

Clients redirect to the application server to update or search status. Application server redirects data to the storage server to save 
or application server goes to the storage server and index server to find query results. Also, application server also goes to index 
server to create an index.

## Detailed Component Design

Notice that we can easily calculate the over the 5 years data to store. As we explained above, every 150 GB data is coming to store
to this system.

So 150 GB * 365 days * 5 = approximately 250 TB.

Using the monitoring tools, we can easily determine the usage of server and we don't want to use each server over than %80. This means 
that we should think the storage data like an approximately 320 TB to not using system server fully. Notice that,
because of the providing the available and being real-time system and fault tolerance, our system data capacity reaches the approximately 
900 TB. If we assume that modern servers can store data up to 4 TB, then we would need to 250 servers to store all data over the five 
years. Notice that we explained the usage of NoSQL and we can think about the usage of the MySQL in this part. MySQL is provided to 
sharding with tools. We can store the coming data in a table and this table contains two columns. First one is tweetIDs and second 
one is Tweettext. Let's assume we partition our data based on TweetIDs. We can create a hash function to store this data. Additionally, 
we should calculate the amount of statusID to use. If we calculate the 5 years status number,

500 M (number of coming tweets in each day) * 365 days * 5 years = approximately 750 billion tweets in 5 years. So we can identify these 
with 5 bytes (40 bits.). We can use a key generation service to create these ids. Notice that, if we create these ids before and save
these ids any server and each server keeps a piece of these ids, we can increase the response time speed. Notice that, the system should 
serve the ids is unique to each server. Additionally, key generation servers database has two table which is used and unused. If this 
service id, then immediately serviced ids transport to used table from the unused to table to provide the unique state. Moreover, we can 
use index to increase the response time speed. So how this index should be? So we can use the trie structure to store indexes and 
firstly, we can think about the index for every English word. If we assume that totally 500 K words in our index and we assume that 
each word length is 10, we can easily calculate the stored words 500K * 10 MB (each letter can be held by one byte) = 5 MB.

So 5MB for all the words to store.

We want to keep the index in memory for all tweets objects for only past one years. Since approximately 750 billion tweets in each day, 
750 B / 5 = 150 B each of tweetID is 5 bytes = 750 GB. (to store all the tweetIDs)

We keep the data with the key-value pair. Key would be word and value wold be list of tweetIDs. Assume that each tweet has approximately 
50 words if we subtract to sum unimportant words like "as", "and", "or", Total words to store in each tweet is approximately 20.

(750 GB * 15) + 5 MB = approximately 10 TB. Notice that if we use the high-end server to provide being real time, 
(assume that each high-end server has 150 MB) we need approximately 60 servers to built index.

## Sharding

Sharding will be done based on the tweetIDs. Notice that with these sharding process, servers will be balanced. Additionally, 
if any user searches to query, this query will be redirected to every server and each server will bring the related results. 
The final process is aggregator services. Aggregator services will collect these results and return the application server, 
application server gives these result to users.

* Users search queries
* Load balancer redirect request to suitable application servers.
* Application server goes to aggregator servers
* Aggregator servers go to database servers and index servers.
* Database servers can set the connection with the index-builder-server.
* İndex-builder-server can set the connection with the index servers.

## Fault Tolerance

If any server fails or dies, the secondary server can act a role like a primary server, but what if both primary and secondary servers 
fail?
If we meet this stage, we have to allocate new server and do the same process to create an index. But how can we do that? We don't know 
the words and tweets on these servers. If we created sharding via tweetID we have to use brute-force mechanism. This means that we 
have to create all indexes again from the database. With this process, when creating index again, any queries cannot take the 
responsibility for these servers. This situation conflict with high availability.

If we create any process that provides to reverse index, we can create index just only occur in these died servers. We can use a hash map
and this contains that key-value pair. Key is the tweetId and value is the tweetValue. This process will be much better than the first one.

## Cache

We can use 80-20 rule and LRU caching mechanism to increase the response time speed. We can use the Memcache to store the all hot tweet
in memory.

## Load Balancing

As we explained before, we can use the load balancer in two places. Fİrst one is between the client and application server, 
the second one is application server and aggregation servers. Notice that we can use intelligent load balancer algorithm rather than 
the Round Robin because round robin will continue to send request when the server dies or fails.

If we want to service the results with ranking, what should we do? Ranking can contain popularity, relevance, actuality, or locations. 
We can create any algorithm to create popularity number and store it with the index. In this process aggregator servers
will aggrate these result by considering the ranking. 
