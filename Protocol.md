# Introduction #

Tio protocol is simple and inspired by the memcached protocol, as we can see in the following telnet/nc session (just note ">>" prefix sent data, it's not part of the protocol):

```
>> ping abc
answer ok PONG abc
>> create test volatile_list
answer ok handle 1
>> get_count 1
answer ok count 0
>> push_back 1 value string 3
>> abc
answer ok
>> push_back 1 value string 9
>> hello tio
answer ok
>> get_count 1
answer ok count 2
>> get 1 key int 1
>> 0
answer ok data key int 1 value string 3
0
abc
>> get 1 key int 1
>> 1
answer ok data key int 1 value string 9
1
hello tio

** reconnect **

>> open teste
answer error key not found
>> open test
answer ok handle 1
>> subscribe 1 0
answer ok
event 1 push_back value string 3
abc
event 1 push_back value string 9
hello tio
>> push_bac
answer error invalid command
>> push_back 1 value
answer error Invalid parameter count
>> push_back 1 value string 4
>> xpto
event 1 push_back value string 4
xpto
answer ok
>> get 1 key int 2
>> -1
answer ok data key int 2 value string 4
2
xpto
>> subscribe 1
answer error already subscribed
>> unsubscribe 1
answer ok
>> subscribe 1
answer ok
>> delete 1 key int 1
>> 0
event 1 delete key int 1
0
answer ok
```

### Basics ###
  * All commands follow the pattern `<command>` `<parameters>` and are terminated by a "\r\n" or "\n"
  * Protocol also expects "\r\n" after data sent on data commands, like push\_back. Actually, it expects two more chars, you can add everything. If you're using netcat instead telnet like I do, you will need to press enter twice to send "\n\n".
  * Volatile containers survive reconnections, but don't survive a Tio restart.
  * Subscribe command has an optional second parameter used to inform the index to start the transmission. If you don't send this parameters, the behavior depends on the container you are using. When using maps, you will receive a "set" event for each existent item. If it's a list or vector, you will not receive current items if subscribing with no args.
  * You can access index based containers (like lists) using negative indexes, just like in Python language. -1 points to the last member, -2 to the item before the last, and so on. You can use this when subscribing to lists too.

### Data Model ###
Every item on a container has three fields: key, value and metadata. Value and metadata are set by the user. The key field interpretation depends on the container:

  * Lists and vectors: key is the item index, and can't be changed
  * Maps: the key is (surprise!), the item key. Maps only support string keys, but you can _access_ them using a integer index


## Simple Commands ##

_syntax: command\_name `<required parameters>` `[optional parameter]`_

>> create `<name>` `<type>`<br />
_answer ok `<handle>`_<br />
Create a container, or open if it already exists. Note you can inform a real type (like volatile\_map) or an type alias (create using the --alias command line parameter)

>> open `<name>` `[type]`<br />
_answer ok `<handle>`_<br />
Open an existing container. The "type" parameters is optional, you can open a container without knowing its type

## Data Commands ##
The data commands syntax follow the following pattern:

>> command\_name `<handle>` [`<key | value | metadata>` `<data_type>` `<data_size>`]... ?
>> \r\n<br />
>> `[data for field 1]`<br />
>> `[data for field 2]`<br />
>> `[data for field 3]`<br />

Data type can be "string", "int", "double" or "raw". Data size is the byte count of data sent via the socket. The size of a int field with value 10 will be 2, since the protocol is ascii only.

_**example\_command (`<list of supported fields>`)**: explanation_<br />
**get (key)**: get the item having the informed key. The key can be returned too if it's different from the informed key (if you access a list using a negative index, you'll receive the positive index)<br />
**push\_back (value, metadata)**: add an item to the end of the container<br />
**push\_front (value, metadata)**: add an item to the beginning of the container<br />
**set (key, value, metadata)**: change an item value<br />
**insert (key, value, metadata)**: if the container is a list, insert an item at the index _key_. If it's a map, insert the item or return an error if the item already exists<br />
**delete (key)**: delete the item having the informed key<br />
**pop\_back**: return the last item in the container and deletes it<br />
**pop\_front**: return the first item in the container and deletes it<br />

### Examples ###
>> push\_back 1 value string 3<br />
>> abc<br />
_answer ok_

>> set 1 key string 3 value int 2 metadata double 6<br />
>> abc<br />
>> 10<br />
>> 3.1415<br />
_answer ok_

>> get 1 key string 3<br />
>> abc<br />
_answer ok data key string 3 value int 2 metadata double 6_<br />
abc<br />
10<br />
3.1415<br />

>> get 1 key string 3<br />
>> ABC<br />
_answer error key not found_<br />