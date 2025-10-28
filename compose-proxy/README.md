#   ERT deployment using docker compose with HTTP proxy configuration

This section provides a docker compose stack to deploy an Edge Runtime with no direct internet connectivity.  
To connect to the Hybrid Control Plane, the ERT is configured to use a HTTP proxy (squid) which only allows specific URLs.
The stack also includes a network sniffer which captures the ERT trafic, for analysis purpose.
Finally, there's a SQL Server container to manage JDBC connectivity from the ERT container.

To start the stack:
```
docker compose -f docker-compose-proxy.yaml up -d
```

To stop it:
```
docker compose -f docker-compose-proxy.yaml down
```

To display the ERT logs:
```
docker logs -f edge
```


##  Docker network configuration

The stack defines an isolated_net network which as an internal attribute set to true. This means this network has no internet access.  
There is also another network named default, which does provide internet access.  

The ERT container (edge) is only attached to the isolated_net network, therefore it has no internet access at all. But it can talk to the HTTP proxy (squid) and SQL Server (sqlserver) containers, because these two containers are also attached to the isolated_net network.  
The HTTP proxy container (squid) is also attached to the default network, so it has internet connectivy. Same for the SQL Server container.  

Note: because the ERT is in an isolated network, it's not possible to directly access its admin console.

##  HTTP proxy configuration

Squid is used as the HTTP proxy here. Its configuration is done using a squid.conf file that is mounted into the container. This config file is provided in three flavours:
-   squid.conf does not block any traffic
-   squid-block.conf blocks specific domains and allows all the others
-   squid-allow.conf only allows specific domains and blocks all the others

The stack comes preconfigured with the squid-allow.conf file, which only allows:
-   The connection to the control plane: presalesemeaprod.int-aws-de.webmethods.io
-   The hybrid connectivity via UM/nhps: resalesemeaprod.um.int-aws-de.webmethods.io

These two domains are tenant/environment specific. The config file can easily be adapted.  

##  Edge Runtime configuration

The stack is preconfigured with the vanilla ERT image in version 11.2.1. This image can easily be replaced with another custom image.  

The connection to the HTTP proxy is two-fold:
-   we pass a set of Java properties to the runtime
-   we also define a default HTTP proxy alias using environment variables placed in the .env file

This configuration is NOT redundant, for the ERT needs both to correctly talk to the Control Plane.  

The three veriables needed to connect to the Control Plane are placed in the .env file.

Note: you get an env.example file in the repo, which you can use to create your own .env file.

##  Network sniffer

The stack embeds a network sniffer that captures the ERT network trafic. It generates a new binary pcap file each time the stack is started.  
Such a pcap file can then be analyzed using Wireshark.  
Alternatively, you can use the tshark.sh script to generate a text summary of the network traces, which you can then pass to Gen AI tools for analysis. I haven't seen confidential data in the captured traces, but be cautious there.

Note: this sniffer does not have any influence on the behaviour of the ERT. 

##  SQL Server

The password is also placed in the .env file. The default username is used here, "sa", the database name is "master". 
The database data is persistent in volumes, so you don't loose anything when you restart your containers.

##  Direct network access stack

There's also another docker compose stack provided, this time with an ERT that has direct internet access: docker-compose-direct.yaml  
It is equipped with the same network sniffer, so you can use it to compare the network traffic in both situations (access via proxy or direct access.)