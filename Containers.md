# Introduction #
Tio containers can be volatile or persistent. Persistent container will survive a Tio process restart, volatile ones won't.

## Volatile conatiners ##

### volatile\_vector ###
Implemented as an auto resizable array. Indexed access is fast, but insertion in the middle of container will move all item one offset to the right.

  * Item access: By numeric index, O(1)
  * Available Operations:
    * push\_back, amortized O(1)
    * push\_front, O(TODO)
    * pop\_back, O(1)
    * pop\_front, TODO: will items be moved?
    * get, O(1)
    * get\_count, O(1)
    * insert, O(1) to find the insertion point, O(n/2) average items will be moved 1 slot forward
    * delete, O(n/2) average items will be moved 1 slot backward
    * subscribe: subscribing without informing start item will **not** send current items. If informed, start item is interpreted as the first item to send (items will be sent as push\_back event)

Example

```
import tioclient
tio = tioclient.Connect('tio://127.0.0.1:12345')
v = tio.CreateContainer('vector_1', 'volatile_vector')

v.append('abcdefghij')
v.append(100)

assert v[-1] == 100 # Tio containers support negative indexes the same way Python containers do

v.append(3.1415)
v.append('value', 'metadata') # inserts a value and associate a metadata to the item
v.push_back('one more value') # push_back and append are equivalent. append is more pythonic

del v[0] # delete item at index 0
v.insert(0, 'first item')
v.push_front('now I am the first one')

v.clear()
assert len(v) == 0

v.append(123)
v.append(456)
v.append(789)
assert v.pop_front() == 123
assert v.pop_back() == 789

```

### volatile\_list ###
Implemented as an Double Linked List with item count.

  * Item access: By numeric index, O(n/2) average
  * Available Operations:
    * push\_back, O(1)
    * push\_front, O(1)
    * pop\_back, O(1)
    * pop\_front, O(1)
    * get, O(n/4) average. If index > (n /2), Tio will walk the list backward from the last item
    * get\_count, O(1)
    * insert, O(n/2) average to find insertion point.
    * delete, O(n/2) average to find item to delete.
    * subscribe: subscribing without informing start item will **not** send current items. If informed, start item is interpreted as the first item to send (items will be sent as push\_back event)

Example:

```
import tioclient
tio = tioclient.Connect('tio://127.0.0.1:12345')
l = tio.CreateContainer('list_1', 'volatile_list')

l.append('abcdefghij')
l.append(100)

assert l[-1] == 100 # Tio containers support negative indexes the same way Python containers do

l.append(3.1415)
l.append('value', 'metadata') # inserts a value and associate a metadata to the item
l.push_back('one more value') # push_back and append are equivalent. append is more pythonic

del l[0] # delete item at index 0
l.insert(0, 'first item')
l.push_front('now I am the first one')

l.clear()
assert len(v) == 0 # checking list length is fast

l.append(123)
l.append(456)
l.append(789)
assert l.pop_front() == 123
assert l.pop_back() == 789

```


### volatile\_map ###
A key/value container. Keys must be strings, values don't. On Windows and Linux map is implemented as a Red-Black Tree (TODO: check the Linux info)

  * Item access: by string key, O(logn/2). Items can be accessed using numeric index (TODO: O(??))
  * Available Operations:
    * get\_count, O(1)
    * insert, O(logn/2)
    * delete, O(logn/2)
    * subscribe: send all records on subscription, as "set" events. To receive only updates, you must send `__none__` as initial item.

Example:

```
import tioclient
tio = tioclient.Connect('tio://127.0.0.1:12345')
m = tio.CreateContainer('map_1', 'volatile_map')

m['a'] = 'b'
m['c'] = 12

def show(container, event, key, value, metadata):
  print container, event, key, value, metadata

m.subscribe(show) # items 'a' and 'b' will be received as two 'set' events

assert m[-1] == 12

m.insert('value', 'metadata') # inserts a value and associate a metadata to the item. If the item already exists, will raise an exception

del m['a']
del m['c']

assert len(m) == 0

#using the synchronous api
tio.DispatchAllEvent()

```

## Persistent conatiners ##

## persistent\_list ##

TODO

## persistent\_map ##
All keys and values will be added to the transaction log, but keys will be kept on memory, on a Red-Black Tree. There's a cache for data written to transaction log.

  * Item access: by string key, O(logn/2). Items can be accessed using numeric index (TODO: O(??)). Disk will not be touched if the required item doesn't exist.
  * Available Operations:
    * get\_count, O(1), memory operation only
    * insert, O(logn/2) plus the disk write
    * delete, O(logn/2) plus the disk write
    * subscribe: send all records on subscription, as "set" events. To receive only updates, you must send "none" as initial item.