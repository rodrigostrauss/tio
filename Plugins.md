# Introduction #
Tio plugins can access/modify Tio containers and subscribe to containers events, the same way a client can do. But a plugin does this without all the network penalty.

Tio plugin support was made in a way that a program can run as a plugin (inside Tio) and as a standalone executable (outside Tio) with few modifications. The plugin interface mimics the client API.

# Python Plugins #

As you may know, to connect to a Tio server a python script should call `tioclient.Connect` to create a Connection Manager. The difference between a plugin and a regular client is that a plugin will receive the Connection Manager through a call to the plugin function `TioPluginMain`.

```

def TioPluginMain(container_manager, parameters):

    print 'testplugin. parameters: %s' % parameters
		
    def Log(container, event_name, key, value, metadata):
        print 'testplugin.log: ', container, event_name, key, value, metadata
    
    container_manager.open('meta/containers').subscribe(Log)

```

The above example, when loaded as plugin, with log to stdout everytime new container is created or deleted.

# C Plugin #
TODO

# C++ Plugin #
TODO