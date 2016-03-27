# SaaS architecture
+ 12Factor app
+ Cloudfront CDN
+ Http cache
    + Cache-Control(public/private)
    + Expires
    + Conditional requests
        + Time-based
        + Content-based
    + Cache prevention: Cache-Control:no-cache, no-store
+ Storage services: Using AWS S3 to Store Static Assets and File Uploads
+ Worker Dynos, Background Jobs and Queueing
+ Scheduled Jobs and Custom Clock Processes
    + clock process
    + don't use cron!
    + Separating a job’s execution from its scheduling so that we can scale executors: Use a job scheduler only to queue background work and not to perform it.
    + What's the weakness of in-app schedulers
        + Multi-instance schedule the same job multiple times
        + Job/queueing can be better
+ Communication: http or queue(better)
    + http: for instant communiation
    + message queue: more flexible -> fire and forget! and resistent(when some services broke)
+ Compared to traditional SOA + ESB
