jumpssh
=======

ssh to a ssh server through another ssh server through another ssh server through... and forward ports too:

Features and examples
---------------------

#### Jump directly to server behind multiple gateways:
    
```bash
# ssh to gateway.domain.com port 2222 as user bob.
# From gateway, ssh to clustergateway port 22 as user clusteruser
# From clustergateway, ssh to clustermachine port 22 as user clusteruser    

jumpssh bob@gateway.domain.com:2222 clusteruser@clustergateway clusternode
```

#### Execute command on innermost ssh server:
    
```bash
# Report clusternode disk space usage
jumpssh -L 2222:22 bob@gateway clustergateway clusternode --exec "df -h"
````

#### Share ports between your client and innermost ssh server:

```bash
# Forward client port 2222 to clusternode port 22
jumpssh -L 2222:22 bob@gateway clustergateway clusternode

# Forward clusternode port 2222 to client port 22
jumpssh -R 2222:22 bob@gateway clustergateway clusternode
```

#### Use forwarded ports in scripts

Forward a port from client to clusternode with a timeout, so you can use it in a script

```bash
# you've got 60 seconds to get your work done!
jumpssh -L 22:2222 bob@gateway clustergateway clusternode --exec "sleep 60" &            
# wait a bit for ports to open, then do something with the open port        
sleep 5 ; scp -P 2222 localhost:~/important ~/backup
```

*Note: See `jumpsshthen` and `jumpscp` for better solutions to this problem.*

#### Just echo the ssh command

Don't rely on jumpssh directly, but use it to build complex ssh forwards for your scripts.

```bash
# Forward some ports from clusternode to client, and some from client to clusternode.
# Only don't actually execute it, but print the ssh command required to do so.
jumpssh --echo -L 22:2222 -L 81:81 -R 8080:8080 bob@gateway clustergateway clusternode
```