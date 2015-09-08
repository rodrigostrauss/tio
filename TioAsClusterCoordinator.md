# Introduction #

Tio can be used as a cluster coordinator using its "wait and pop" commands.

## wnp\_next (Wait And Pop Next) ##

The `wnp_next` command will pop the _first_ record in a list. If no record is available, it will pop it as soon as it becomes available (item is popped and sent via wnp\_next event) . If more than one session send this command when no record is available, the records will be distributed to them in first-ask-first-served fashion. This will, in practice, make Tio distribute the messages between all clients. To create a computing cluster, you just need to follow this pattern:

  * A producer sends (push\_back) all work items to a list. Of course, it doesn't need to add all items at once.
  * The consumers enter a loop where they send a wnp\_next command, process the item when the event comes, and send the command again after processing the item. Every time there's a new workitem, the first consumer that asked will receive it. The load will be balanced between consumers, since they will not just receive new items, they will ask for them when idle.

All sessions subscribing to this list will receive a "delete" event when the record is popped. Note that consumers won't lock until the next workitem arrives, since they will receive the notification as an event, asynchronously.

### Python Example ###
Producer:
```
import random
import tioclient
tio = tioclient.Connect('tio://127.0.0.1:12345')
source = tio.CreateContainer('factorial_cluster/source', 'volatile_list')
destination = tio.CreateContainer('factorial_cluster/destination', 'volatile_list')

source.set_property('destination', 'factorial_cluster/destination')

def OnItemDone(container, event_name, key, value, metadata):
    print value
    
destination.subscribe(OnItemDone)

# add 10k integers from 1 to 500
item_count = 10 * 1000
for x in range(item_count):
    source.append(random.randint(1,500))
    
print 'generated %d items, now waiting for results' % item_count

tio.RunLoop()
```

Processor:
```
import tioclient
tio = tioclient.Connect('tio://127.0.0.1:12345')
source = tio.CreateContainer('factorial_cluster/source', 'volatile_list')

destination_name = source.get_property('destination')
destination = tio.OpenContainer(destination_name)

def factorial(n):
	if n == 0: return 1
	return n * factorial(n -1)

def GottaWorkToDo(container, event_name, key, value, metadata):
	print 'calculating factorial(%d)' % value
	# like "'20!=2432902008176640000'"
	destination.push_back('%d!=%s' % (value, factorial(value)))
	source.wait_and_pop_next(GottaWorkToDo)

source.wait_and_pop_next(GottaWorkToDo)
tio.RunLoop()
```

### Protocol example ###
```
>> open test
answer ok handle 1
>> get_count 1
answer ok count 2
>> wnp_next 1
answer ok
event 1 wnp_next value string 9
hello tio
>> get_count 1
answer ok count 1
>> wnp_next 1
answer ok
event 1 wnp_next value string 4
xpto
>> wnp_next 1

** note that the wnp_next event wasn't generated now, since the container is empty

>> push_back 1 value string 3
abc
event 1 wnp_next value string 3
abc
answer ok

>> get_count 1
answer ok count 0

```


## wnp\_key (Wait And Pop Key) ##

The map container equivalent to wnp\_next. But you must inform the key of the item that will be popped. When an item with the informed key is inserted to the map, it will be popped to this session client:

```
>> create_container dict volatile_map
answer ok handle 1
>> set 1 key string 7 value string 7
>> rodrigo
>> strauss
answer ok
>> wnp_key 1 key string 7
>> rodrigo
answer ok
event 1 wnp_key key string 7 value string 7
rodrigo
strauss
>> wnp_key 1 key string 7
>> rodrigo
answer ok
>> set 1 key string 5 value string 7
>> linus
>> tovards
answer ok
>> set 1 key string 7 value string 7
>> rodrigo
>> Surname
event 1 wnp_key key string 7 value string 7
rodrigo
Surname
answer ok
```