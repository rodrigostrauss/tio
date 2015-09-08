  * Subscription filter, by key, value and by event type
    * Containers adapter, like STL ones. Queue, stack, etc
    * New containers: priority queue, set, file
      * file/blob
        * set(key=file offset, value=data)
        * push\_back(value=data to append to file)
        * get(value="50:200") --> read from offset 50 to offset 200
  * Bidiretional communication using Tio queue, like MQSeries
  * Container inside container, making container another valid data types. It would make trees and relations easier.
  * Binary Protocol
  * Replication
  * Failover
  * Formal schema support in libs
  * Value Indexing. This can be done outside tio, subscribing to the container and updating the index
  * Console for queries, better than usign telnet everytime
  * Making it multithreaded, not very hard using Boost.Threads
  * Support for manipulating containers without opening, using a syntax like "blogs/users/`[user-id]`" instead of the handle. It will open the map "blogs/users" and get the value for key `[user-id]`
  * Limited size container, like http://www.mongodb.org/display/DOCS/Capped+Collections
  * Integrate Tio publish/subscribe feature to a SQL database, using triggers
  * Formal support for more fields than value and metadata
  * State Machine Framework running over Tio