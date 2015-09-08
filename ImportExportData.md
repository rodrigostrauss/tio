# Introduction #
You can use tia.py to import and export data to and from Tio.

```
Usage: tia.py [options]

Options:
 -h, --help            show this help message and exit
 -i INPUT, --input=INPUT
                       Input. If not specified, will read from stdin. Use
                       tio://server/container to specify a tio source
 -o OUTPUT, --output=OUTPUT
                       Output. If not specified, will write to stdout. Use
                       tio://server/container to specify a tio output
 -f INPUT_FORMAT, --input-format=INPUT_FORMAT
                       Specify a input format. [csv], [len_prefixed]
 -u OUTPUT_FORMAT, --output-format=OUTPUT_FORMAT
                       Specify a output format. [csv], [len_prefixed]
 -e LOG_EACH, --log-each=LOG_EACH
                       log to stderr each N records
 -k KEYS, --keys=KEYS  dump keys instead of values
 -r, --resume          resume load. Skip N input records, where N =
                       destination record count
 -g, --ignore-not-empty
                       write to tio container even if it's not empty. The
                       default action is abort
 -c CREATE_CONTAINER_TYPE, --create-container=CREATE_CONTAINER_TYPE
                       create the Tio containers if they don't exist, using
                       the informed type

```

# Examples #

<pre>
# copy the records from my_container on server1 to my_container_copy on server2<br>
tia.py -i tio://server1:6666/my_container -o tio://server2:6666/my_container_copy<br>
<br>
# copy the records from my_container on server1 to a file<br>
tia.py -i tio://server1:6666/my_container -o /tmp/container_dump<br>
<br>
# load records from file to a container. if the container doesn't exist, create it as a volatile_list<br>
tia.py -i /tmp/container_dump -o tio://server1:6666/my_container -c volatile_list<br>
<br>
# load records from file to a container and log to stderr each 1000 records<br>
tia.py -i /tmp/container_dump -o tio://server1:6666/my_container -e 1000<br>
<br>
# Resume load operation. If the container has N records, it will skip the N first records on file<br>
tia.py -i /tmp/container_dump -o tio://server1:6666/my_container -r<br>
<br>
# Update a Tio list with the Apache access log in real time<br>
tail -f access.log | tia.py -o tio://server1:6666/apache_log_real_time -c volatile_list<br>
</pre>