## Redis ##
  * Redis is C and POSIX, Tio is C++ and Multiplatform
  * Redis focus on being a data structure server. Tio focus on the same thing, but with emphasis on publish/subscribe
  * In Redis, if you subscribe to any container, you can't send commands until unsubscribe
  * Tio has WaitAndPop commands, Redis has BLPOP for "dequeue when available". But BLPOP blocks until a message is available (with a timeout)
  * The publish/subscribe model in Redis follow the channel model. In Tio, it's intrisic to every container. A list can be a channel.
  * Tio has better authentication support. Containers have thiers own ACLs.
  * Both use append only log for persistence, but Tio uses an IO\_DIRECT file instead of fsync'ing every time. Windows' fsync equivalent, FlushFileBuffers, is very slow.
  * Redis lacks POP support, user must create a MULTI GET/DELETE command
  * Tio data model supports strings, doubles and int32
  * Redis has item expiration support, Tio don't
  * Redis has key rename support, Tio don't
  * Redis is faster than Tio
  * Tio management is done using meta containers, user can subscribe to them for monitoring. On Redis you have commands for this.
  * Tio has a metadata field for every value
  * Tio containers have properties