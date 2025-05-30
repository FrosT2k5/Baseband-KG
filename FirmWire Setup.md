Requirements:
Docker, tested on `Docker version 28.1.1, build 4eba377`
Python3, Java 

reference: 
https://firmwire.github.io/docs/installation.html

```
git clone https://github.com/FirmWire/FirmWire.git
cd FirmWire
git clone https://github.com/FirmWire/panda.git

# This will take some time
docker build -t firmwire .

```

Execute the FirmWire Docker container:
```
docker run --rm -it -v $(pwd):/firmwire firmwire
```

Firmwire is located at 
```
cd /firmwire
```
directory, primary folder for executing firmwire

# Connect to existing container

drop shell to existing firmwire container:

```
1. List Containers
host $ docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED       STATUS 
36de91227b5e   firmwire                    "/bin/bash"              3 hours ago   Up 3 hours

2. connect your firmwire container.
host $ docker exec -it 36de91227b5e /bin/bash
```

