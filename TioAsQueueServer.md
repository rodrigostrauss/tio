  * Create a list
  * Producers and Consumers open the list. You can have more than one producer and more than one consumer. If you don't want this, block read and write accordingly using ACL
  * Producers put items in the list with push\_back command
  * Consumers get item from the list using the _pop\_front_ command or the [wnp\_next](TioAsClusterCoordinator.md) command

## Python Example ##
Producer
```
import tioclient
server = tioclient.Connect('tio://127.0.0.1:6666')
container = server.CreateContainer('xpto', 'volatile_list')

for x in range(10):
    container.append('value %d' % x, 'metadata')
```

Consumer
```
import tioclient
server = tioclient.Connect('tio://127.0.0.1:6666')
container = man.OpenContainer('xpto')

def OnNewItem(container, event_name, key, value, metadata):
    print value

# if you want to receive events for all messages in the list, use start=0
container.subscribe(sink=OnNewItem, event_filter='push_back', start=None)

man.RunLoop()
```

Note that the messages will not be removed from the list, the client must use the start parameter to continue from where it stopped.

If you want to remove messages you can use the pop\_front method on a push\_back event or use [wnp\_next](TioAsClusterCoordinator.md)