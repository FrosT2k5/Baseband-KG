Requirements:
Docker, tested on `Docker version 28.1.1, build 4eba377`
Python3, Java 

reference: 
https://firmwire.github.io/docs/installation.html

drop shell to existing firmwire container:
```
1. List Containers
host $ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED       STATUS 
36de91227b5e   firmwire                    "/bin/bash"              3 hours ago   Up 3 hours```

2. connect your firmwire container.
host $ docker exec -it 36de91227b5e /bin/bash
```

