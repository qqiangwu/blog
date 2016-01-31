# How to accelarating a web application?
Reverse proxy: use proxies to redirect requests to real web servers. The proxy itself can do a lot of extra stuff such as caching, balancing, securing and compressing.

+ Cache: pages are cached in RAM thus can be served fast than reading directly raw files.
+ Balancer: balancing requests to multiply servers

# Famous reverse proxy tools
## Varnish
+ What: Varnish is a web application accelerator. You install it in front of your web application and it will speed it up significantly.
+ Why: contents are cached in RAM thus they can be access faster than reading from a web server.

## HAProxy
+ What: a load balancer
    + backend max connections and frontend queue to throtlling connections
    + splice system call: let kernel to read the file
    + failover, cli management, clustering
    + also support tcp load balancing
+ Why: load balancing, connection throtlling and is generally better than nginx

## Nginx
+ What: a web server and a reverse proxy
    + gzip
    + serve static contents
    + load balancer

## Comparision
+ nginx:
    + ssl: yes
    + compress: yes
    + cache: yes
    + load balancing: yes
+ varnish:
    + ssl: no
    + compress: ?
    + cache: yes(primary feature)
    + load balancing: yes
+ haproxy:
    + ssl: no
    + compress: ?
    + cache: no
    + load balancing: yes(primary feature)

# Monitoring
To gain a better understanding of your reverse-proxied system, you'd better use a monitor to grasp system metrics.