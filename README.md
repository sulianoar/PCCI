# Parallel Computer Communication Interface

> Note:
> - As this PoC has no security to authenticate requests, you should not leave it available on the Internet.
> - I designed it for my own use with my raspberry pi cluster. It work well but i cannot be responsible for any problem with it.
> 
> **Use and modify it at your own risk.**

### Description

This project allows a MASTER machine to manage batche jobs (run / status / kill) command to one or more WORKER.

A Rest API is available for manually or using script (check the example folder) control jobs.
MASTER will communicate silently with WORKER using the TCP socket interface so that you can talk with your cluster as a single machine.

This is a low-level task management implementation, which means you need to implement your own queuing system based on what works best for your project. For example, you can:
- wait for a job to be done on a worker before starting a new one
- check resources on the worker while the job is running to see if you can run a second job on this same one
This project does not cover this part of the queue except for healthCheck example to show you a time loop based implementation to get stats of the nodes ressources and save them in innodb dataBase.

Some more technical details:

- Since TCP SOCKET is a data stream, i implemented an "End Of Message" (EOM) code to identify each socket message end
- Due to the "not waiting for response" on job submission, a caching system allows you to check the result. It is automatically cleaned on the "poolRetention" variable.
- A job is a command to be executed on a unique worker
- A pool is a list of the same job executed to each worker
- Pool ID is based on the date and time of the pool in milliseconds
- Worker ID is based on the hostname of the machine
- Status code is based on exit code C99

### Installation
- Place "master" folder on a master machine and "worker" folder on each worker you want (can be run in master too)
- Enter in each folder and edit each index.js to set MASTER ip address
- run npm install & node index.js


### Available API request

- Execute a new job to listed worker in body, return a pool Id

[POST] http://[MASTER-IP]/job 

BODY : {"cmd": "ping google.com -n 1", "workers": ["node1", "node2"]}

- Get detailled data from a pool

[GET] http://[MASTER-IP]/pool/[pool-id]

- Get all pools

[GET] http://[MASTER-IP]/pools

- Get all workers

[GET] http://[MASTER-IP]/workers

- Kill jobs running on workers listed in body

[PUT] http://[MASTER-IP]/kill/[pool-id]

BODY : {"workers":["master", "node1", "node2", "node3"]}

### TODO

- Add uuid generation for ID (job, pool, worker) 
- Change worker id to node
- Add a start time on pool & job
- Change "cached" by "endTime" on pool & job
- Add authentification to API
