Tio is a data structure server. Instead of serving tables like common databases servers do, it serves lists, vectors, maps, etc. But, different from a common data server, it's build upon a publisher/subscriber foundation. In Tio, you can subscribe to changes in any container, even metacontainers used to control Tio internal state. Want to know when someone creates a new list? Just subscribe to the list "meta/containers" (note this slash-based nomenclature is just a convention).

Every server software (web server, chat server, mail server, etc) keep variables holding its state and when some state changes (user online on a chat server, new e-mail, etc) some clients must be notified. Tio maintains the state in containers and notify interested clients about state changes. Extrapolating the concept to its extreme, you can move all your global variables to Tio, and your software can keep running from another machine if something happens to the current one. And you don't need to write code to notify clients about state change.

Tio enables software to keep state in the same format used in code. Every server code maintains lists and maps in memory holding all server information. It must somehow serialize these containers contents to some permanent store, like an SQL database. Tio allows you to keep the data in a server using the exact same format and using it in the exact same way you do with your variables.

Some simples servers can be created using just Tio, **without a single line of server code**. Examples:

Chat

  * A map to hold users information ("chat/users"). Each new user just create a new item in this map. Like `users_map["rodrigostrauss"] = "it's me"` (it's Python, add a semicolon to make it valid C++ :-)). Anyone who wants to be notified about new users just need to subscribe to this map.
  * Lists to hold each user messages ("chat/messages/rodrigostrauss", for example). User subscribe to this list and other users "push\_back" to this list to send him a message. If he's offline, the message will wait for him in the list.
  * A map to hold online users ("chat/online\_users"). The key is the user id and the value is the message status. When online, a user will set a message status message (`online_users["rodrigostrauss"] = "I'm here!"`), when offline user deletes himself from this dictionary (`online_users.delete("rodrigostrauss")`). You can create a simple program to subscribe to this map and delete users on timeout, you just need to add a "last heard" info to the value. All containers can have properties associated to them, you can use this to hold the schema you'll use in the value field. Or create an schema per record using the "metadata" field.
  * Chatrooms can be simple lists. Subscribe to receive messages and push\_back to send

VERY Simple Multiplayer game
  * A map to hold players ID and its properties
  * Each, say, 64x64 blocks region is controlled with some lists: users, npcs and items on it. A map can hold users positions
  * If a player want to move, if just update the map holding users positions
  * All users subscribe all containers controlling the 64x64 region where it is. If someone moves, he will receive a "set" event from the position map, and the client can update the screen to reflect the change

All use cases above are naive and don't take security into account, but Tio has a security system where you can specify the commands each user can run against each container (each container has its own ACL). Elaborating the chat example, users on your contact list would be able to push\_back messages to your message list, but you would be the only one able to get the messages. If Tio security system is not enough for you, you can create a list where client will ask server to do something (send a message, for example). A little program would check user permissions permissions (probably querying a map) and change the other container (your message list in this example).

Tio is made in C++ using Boost Libraries, and its protocol is simple, inspired by the memcached protocol. It runs on Windows, Linux and MacOSX.

