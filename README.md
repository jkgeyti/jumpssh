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
jumpssh bob@gateway clustergateway clusternode --exec "df -h"
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
user@mycomputer:~ $ jumpssh --echo -L 22:2222 -L 81:81 -R 8080:8080 bob@gateway clustergateway clusternode
ssh -t -p 22 -L 22:localhost:35929 -L 81:localhost:30808 -R 35673:localhost:8080 bob@gateway ssh -t -p 22 -L 35929:localhost:35929 -L 30808:localhost:30808 -R 35673:localhost:35673 clustergateway "ssh -t -p 22 -L 35929:localhost:2222 -L 30808:localhost:81 -R 8080:localhost:35673 clusternode"
```

Full Readme
-----------

```
jumpssh [--echo] [--exec command] [-h] [--help] [-L port:[intport:]hostport]
[-R port:[intport:]hostport] [-???] [user@]hostname[:port]
[user@]hostname[:port] [user@]hostname[:port] ...

    [user@]hostname1
        [[user@]hostname2] ... [[user@]hostnameN
Usage:
jumpssh
  --echo                      Print the generated ssh command instead of
                              executing it
  --exec command              Execute command on remote server instead of a
                              login shell
  -h --help                   This help screen
  -L port:[intport:]hostport  Traffic on client port  is forwarded to port
                               on the remote. If connecting through
                              intermediate hosts, port  is used as the
                              intermediate port to forward the connection
                              through. If  is unspecified, a random port
                              will be used. The same ports is used on
                              consecutive invocations of the same jumpssh
                              command, as command line arguments are used to
                              seed the random generator
  -R port:intport:hostport    Traffic on remote port  is forwarded to port
                               on the client. Otherwise works as -L
  -???                        All other flags (beginning with a dash) are
                              applied to every ssh 'jump' command generated.
  [user@]hostname[:port]      Host(s) to connect to/through. Will do a "jump"
                              to the next server specified, until connecting to
                              the last host in the list

Examples:

Create an ssh connection to a host through other hosts, and set up ssh port
forwards from source to target and back

Examples:

jumpssh -L 2222:22 user@hostname
  connects to hostname. Remote (hostname) port 22 will be accessible from
  client machine through port 2222.

jumpssh -L 2222:22 -R 2223:22 user@host1 host2 host3 host4
  connects to host4 through host1, 2 and 3. Client port 22 will be accessible
  from host4 through port 2223, and host4 port 22 will be accessible from the
  client through port 22. The ports are forwarded through random ports on
  host 2 and 3

jumpssh -L 8080:8181:80 user@host1 host2
  connects to host2 through host1. host2 port 80 will be available on
  client port 8080. host1 port 8181 is used as the intermediate port for this
  port forward.

Note that the "randomly" assigned ports will always be the same for the same
command. 'jumpssh -L 2222:22 user@hostname' will use the same random ports
upon every invocation, as the random generator is seeded using the jumpssh
arguments
```