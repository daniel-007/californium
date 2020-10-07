# Californium - Extended Plugtest Server

Californium contains a plugtest server, that implements the test specification for the ETSI IoT, CoAP Plugtests, London, UK, 7--9 Mar 2014.

That plugtest server is extended by this example-module with

- benchmarks
- receive test
- built-in DTLS connection ID load-balancer-cluster

The additional functions are available at ports 5783 and 5784 instead of the standard ports 5683 and 5684.

## General Usage

Start the server with:

```sh
java -jar cf-extplugtest-server-2.5.0.jar -h

Usage: ExtendedTestServer [-h] [--[no-]benchmark] [--dtls-only] [--[no-]
                          external] [--[no-]ipv4] [--[no-]ipv6] [--[no-]
                          loopback] [--[no-]plugtest] [--[no-]tcp]
                          [--trust-all] [--client-auth=<clientAuth>]
                          [--interfaces=<interfaceNames>[,
                          <interfaceNames>...]]...
                          [--interfaces-pattern=<interfacePatterns>[,
                          <interfacePatterns>...]]...
                          [--k8s-dtls-cluster=<k8sCluster> |
                          [[--dtls-cluster=<dtlsClusterNodes>[,
                          <dtlsClusterNodes>...]...]...
                          [--dtls-cluster-group=<dtlsClusterGroup>[,
                          <dtlsClusterGroup>...]]...]]
      --[no-]benchmark   enable benchmark resource.
      --client-auth=<clientAuth>
                         client authentication.
      --dtls-cluster=<dtlsClusterNodes>[,<dtlsClusterNodes>...]...
                         configure DTLS-cluster-node. <dtls-interface>;
                           <mgmt-interface>;<node-id>. --- for
                           <dtls-interface>, for other cluster-nodes
      --dtls-cluster-group=<dtlsClusterGroup>[,<dtlsClusterGroup>...]
                         enable dynamic DTLS-cluster mode.
      --dtls-only        only dtls endpoints.
  -h, --help             display a help message
      --interfaces=<interfaceNames>[,<interfaceNames>...]
                         interfaces for endpoints.
      --interfaces-pattern=<interfacePatterns>[,<interfacePatterns>...]
                         interface patterns for endpoints.
      --k8s-dtls-cluster=<k8sCluster>
                         enable k8s DTLS-cluster mode.
      --[no-]external    enable endpoints on external network.
      --[no-]ipv4        enable endpoints for ipv4.
      --[no-]ipv6        enable endpoints for ipv6.
      --[no-]loopback    enable endpoints on loopback network.
      --[no-]plugtest    enable plugtest server.
      --[no-]tcp         enable endpoints for tcp.
      --trust-all        trust all valid certificates.
```

To see the set of options and arguments.

## Benchmarks

Requires to start the server with 

```sh
java -Xmx6g -XX:+UseG1GC -jar cf-extplugtest-server-2.5.0.jar --benchmark --no-plugtest
```

The performance with enabled deduplication for CON requests depends a lot on heap management. Especially, if the performance goes down after a while, that is frequently caused by an exhausted  heap. Therefore using explicit heap-options is recommended. Use the benchmark client from "cf-extplugtest-client", normally started with the shell script "benchmark.sh" there.

```sh
Benchmark clients, first request successful.
Benchmark clients created. 671 ms, 2979 clients/s
Benchmark started.
373309 requests (37331 reqs/s, 3676 retransmissions (0,98%), 0 transmission errors (0,00%), 2000 clients)
823525 requests (45022 reqs/s, 4861 retransmissions (1,08%), 0 transmission errors (0,00%), 2000 clients)
1282190 requests (45867 reqs/s, 5180 retransmissions (1,13%), 0 transmission errors (0,00%), 2000 clients)
1746097 requests (46391 reqs/s, 4983 retransmissions (1,07%), 0 transmission errors (0,00%), 2000 clients)
2205815 requests (45972 reqs/s, 4989 retransmissions (1,09%), 0 transmission errors (0,00%), 2000 clients)
2660792 requests (45498 reqs/s, 5142 retransmissions (1,13%), 0 transmission errors (0,00%), 2000 clients)
3116271 requests (45548 reqs/s, 5426 retransmissions (1,19%), 0 transmission errors (0,00%), 2000 clients)
3563583 requests (44731 reqs/s, 6005 retransmissions (1,34%), 0 transmission errors (0,00%), 2000 clients)
```


## Receive Test

A service, which uses requests with a device UUID to record these requests along with the source-ip and report them in a response. A client then analyze, if requests or responses may get lost. Used for long term communication tests. An example client is contained in "cf-extplugtest-client".

```sh
java -jar target/cf-extplugtest-client-2.5.0-SNAPSHOT.jar ReceivetestClient --cbor -v

Response: Payload: 491 bytes
RTT: 1107ms

Server's system start: 10:49:30 25.09.2020
Request: 13:25:17 09.10.2020, received: 79 ms
    (88.65.148.189:44876)
Request: 13:25:02 09.10.2020, received: 82 ms
    (88.65.148.189:44719)
Request: 13:17:33 09.10.2020, received: 77 ms
    (88.65.148.189:39082)
Request: 13:16:52 09.10.2020, received: 75 ms
    (88.65.148.189:49398)
Request: 13:16:45 09.10.2020, received: 217 ms
    (88.65.148.189:58456)
Request: 13:06:28 09.10.2020, received: 75 ms
    (88.65.148.189:49915)
Request: 13:06:19 09.10.2020, received: 207 ms
    (88.65.148.189:45148)
Request: 13:01:04 09.10.2020, received: 76 ms
    (88.65.148.189:37379)
Request: 12:59:21 09.10.2020, received: 79 ms
    (88.65.148.189:35699)
```

## Built-in DTLS Connection ID Load-Balancer-Cluster

Currently several UDP load-balancer-cluster ideas are investigated.

-  [Leshan Server in a cluster](https://github.com/eclipse/leshan/wiki/Using-Leshan-server-in-a-cluster) general analysis of CoAP/DTLS load-balancer-cluster
-  [LVS](http://www.linuxvirtualserver.org/) UDP load-balancer-cluster, based on temporary mapped source addresses to cluster-nodes.
-  [AirVantage / sbulb](https://github.com/AirVantage/sbulb) UDP load-balancer-cluster, based on long-term mapped source addresses to cluster-nodes.
-  [DTLS 1.2 connection ID based load-balancer](https://github.com/eclipse/californium/wiki/DTLS-1.2-connection-ID-based-load-balancer) DTLS Connection ID based mapping to cluster-nodes.

Note:
Currently no idea above will be able to provide a high-availability for single message. On fail-over a new handshake is required.

Generally, if DTLS without Connection ID is used, the UDP load-balancer-cluster depends on the mapping of the source-address to the cluster-node. If that mapping expires, frequently new DTLS handshakes are required. That is also true, if somehow the source-address has changed.

That shows a parallel to the general issue of DTLS, that changing source-addresses usually cause message drops, because the crypto-context is identified by that. [DRAFT IETF TLS-DTLS Connection ID](https://www.ietf.org/archive/id/draft-ietf-tls-dtls-connection-id-07.txt) solves that by replacing the address with a connection ID (CID). The last of the above links points to a first experiment, which requires a special setup for a ip-tables based load-balancer. The extended-plugtest-server now comes with such CID based built-in load-balancer. The functional principle is the same: the CID not only identifies the crypto-context, it also identifies the node.

```sh
ID 01ab2345cd 
ID 02efd16790 
   ^^
   ||
   Node ID
```

A simple mapping would associate the first with the cluster node `01` and the second with node `02`. With that, a `DTLSConnector` is able to distinguish between dtls-cid-records for itself, and for other cluster-node's `DTLSConnector`. If a foreign dtls-cid-record is received, that dtls-cid-record is forwarded to the associated cluster-node's `DTLSConnector`. Unfortunately, forwarding messages on the java-application-layer comes with the downside, that all source-addresses are replaced by the forwarding `DTLSConnector`. In order to keep them, the built-in-cluster uses a simple cluster-management-protocol. That prepends a new cluster-management-header, containing a type, the ip-address-length, the source-port, and the ip-address to the original dtls-cid-record.

```sh
    +-----------------------------------+
    | Type:      in/out      (1 byte )  |
    | IP-Length: n           (1 byte )  | 
    | Port:      port        (2 bytes)  | 
    | IP:        addr        (n bytes)  | 
    +-----------------------------------+
    | (original dtls-cid-record)        |
    | content-type: tls12_cid (1 byte)  |
    | ProtocolVersion: 1.2    (2 bytes) |
    | ...                               |
    +-----------------------------------+
```

The receiving `DTLSConnector` is then decoding that cluster-management-record and start to process it, as it would have been received by the `DTLSConnector` itself. If outgoing response-messages are to be sent by this `DTLSConnector`, the message is prepended again by that cluster-management-header and send back to the original receiving `DTLSConnector`. That `DTLSConnector` forwards the the dtls-record to the addressed peer. To easier separate the traffic, cluster-management-traffic uses a different UDP port.

```
    +--------+     +------------+     +----------------------------+
    | peer 1 |     | IPa => IPb |     | DTLS Connector, IPb        |
    | IPa    | === +------------+ ==> | node 1, mgmt-intf IP1      |
    +--------+     | CID 02abcd |     +----------------------------+
    |        |     +------------+     |                            |
    |        |                        |                            |
    |        |     +------------+     |                            |
    |        |     | IPb => IPa |     |                            |
    |        | <== +------------+ === |                            |
    |        |     | ???        |     |                            |
    +--------+     +------------+     +----------------------------+
                                            ||              /\
                                            ||              ||
                                      +------------+  +------------+
                                      | IP1 => IP2 |  | IP2 => IP1 |
                                      +------------+  +------------+
                                      | IN,IPa     |  | OUT,IPa    |
                                      | CID 02abcd |  | ???        |
                                      +------------+  +------------+
                                            ||              ||
                                            \/              ||
                                      +----------------------------+
                                      | DTLS Connector, IPc        |
                                      | node 2, mgmt-intf IP2      |
                                      +----------------------------+
                                      | CID 02abcd: (keys)         |
                                      |                            |
                                      |                            |
                                      |                            |
                                      +----------------------------+
```

### Built-in Cluster Modes

The current build-in cluster comes with three modes:

-  static, the nodes are statically assigned to CIDs.
-  dynamic, the nodes are dynamically assigned to CIDs
-  k8s, the nodes are discovered using the k8s API and dynamically assigned to CIDs 

### Static Nodes

Start node 1 on port 15784, using `localhost:15884` as own cluster-management-interface. Provide `localhost:25884` as static cluster-management-interface for node 2:

```sh
java -jar target/cf-extplugtest-server-2.5.0-SNAPSHOT.jar --dtls-cluster ":15784;localhost:15884;1,---;localhost:25884;2"
```

Start node 2 on port 25784, using `localhost:25884` as own cluster-management-interface. Provide `localhost:15884` as static cluster-management-interface for node 1:

```sh
java -jar target/cf-extplugtest-server-2.5.0-SNAPSHOT.jar --dtls-cluster "---;localhost:15884;1,:25784;localhost:25884;2"
```

In that mode, the `address:cid` pairs of the other/foreign nodes are static.

### Dynamic Nodes

Start node 1 on port 15784, using `localhost:15884` as own cluster-management-interface. Provide `localhost:15884,localhost:25884` as cluster-management-interfaces for this cluster nodes group:

```sh
java -jar target/cf-extplugtest-server-2.5.0-SNAPSHOT.jar --dtls-cluster ":15784;localhost:15884;1" --dtls-cluster-group="localhost:15884,localhost:25884"
```

Start node 2 on port 25784, using `localhost:25884` as own cluster-management-interface. Provide `localhost:15884,localhost:25884` as cluster-management-interfaces for this cluster nodes group:

```sh
java -jar target/cf-extplugtest-server-2.5.0-SNAPSHOT.jar --dtls-cluster ":25784;localhost:25884;2" --dtls-cluster-group="localhost:15884,localhost:25884"
```

In that mode, the `address:cid` pairs of the other/foreign nodes are dynamically created using additional messages of the cluster-management-protocol.

```sh
    +-----------------------------------+
    | Type:      ping/pong   (1 byte )  |
    | Node-ID:   id          (4 bytes ) | 
    +-----------------------------------+
```

(This is currently WIP and is intended to be exchanged by a CoAP over DTLS implementation in the future.)

### k8s Node

Start nodes in a container using port `5784`, and `:5884` as own cluster-management-interface. Additionally provide the external port of the cluster-management-interface also with `5884`.

```sh
CMD ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75", "-jar", "/opt/app/cf-extplugtest-server-2.5.0-SNAPSHOT.jar", "--no-plugtest", "--no-tcp", "--benchmark", "--k8s-dtls-cluster", ":5784;:5884;5884"]
```

Example `CMD` statement for docker.

That requires to use a "statefulSet" for k8s. See scripts in folder "service" for more details.

The cluster-nodes-group is requested from the k8s management APIs for pods.
The pods in the example are marked with "metadata: labels: app: cf-extserver", so the "GET /api/v1/namespaces/<namespace>/pods/?labelSelector=app%3Dcf-extserver" can be used to get the right pods set.
