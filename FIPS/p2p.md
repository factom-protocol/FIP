| FIP   | Title         | Status | Category               | Author                                     | Created    |
| ----- | ------------- | ------ | ---------------------- | ------------------------------------------ | ---------- |
| -     | P2P 2.0  | Draft  | Core       | Who Soup \<<who.soup@gmail.com>\>       | 20190506   |


# Summary

The current implementation of the P2P code has several design flaws that limit expandability. This is a complete rework of the code that aims to get rid of uncertainty of connections as well as polling strategies and also add support for handling multiple protocol versions. This will open up opportunities to improve network flow through generational increments without breaking backward compatibility.

Goals:
* Fully configurable, independent, isolated instances
* Add handshake process to connections to distinguish and identify nodes
* Ability to handle multiple protocol schemes starting from version 9
* Reduced network packet overhead in scheme version 10
* Simple integration
* Easier to read code with comprehensive documentation

# Motivation

The P2P package is already fairly isolated from the rest of factomd code but unfortunately there are a number of glaring roadblocks:
* Peers are not identifiable beyond ip address. Multiple connections from the same node are not differentiated and assigned random numbers
* Peers have inconsistent information (some data is only available after the first transmission)
* Peers depend on the seed file to join the network even across reboots; peers are not persisted
* Connections are polled in a single loop to check for new data causing unnecessary delays
* Connections have a complicated state machine and correlation with a Peer object
* Complicated program flow with a lot of mixed channels, some of which are polled

Tackling some of these would require significant overhaul of the existing code to the point where it seemed like it would be easier to start from scratch. 


# Specification

Since there are sweeping structural changes, the first introduction should be the new structure: 

https://i.imgur.com/lyLXsVX.png

An **Endpoint** is an internet location to which a node can connect to. It consists of an ip address with a port.

A **Peer** is an active TCP connection between two nodes. They are identified by their **Peer Hash**, a combination of endpoint and a self-reported nonce called **NodeID**. There exists one and only one Peer instance per Hash. Each peer has a protocol adapter.

Peers and Endpoints are in an `n:m` relation to each other. One endpoint has multiple peers and each peer has zero or one endpoint.

**Network** is the package's public interface. It has a channel for sending messages, a channel for receiving messages, methods to start/stop the network, and methods for simple peer management via peer hash. **Messages** are a combination of peer hash and payload.

The **Controller** is responsible for management of peers (accepting new connections, dialing, persistence etc) as well as routing messages to the right peer. 

The biggest change is that new connections will now handshake before being accepted. This gives the network far greater control of which connections to accept and which to reject. After a successful handshake (more on that below), all Peers in the system will have a unique but identifiable peer hash as well as a version specific protocol adapter. 

### Peers.json

The persistence of peers across reboots is valuable but dangerous, potentially leading to partitioned networks or eclipsed nodes. Discovery from the seed is supplemented but not replaced by the peer file. 

There are five configuration settings related to persistence:
* `PersistFile`: the filepath of the json database to save the peers to
* `PersistInterval`: the duration of how often to save the file to disk
* `PersistMinimum`: how long a peer has to stay connected to an endpoint for it to count as a connection
* `PersistAgeLimit`: how long a dormant peer (no connections in or out) stays in the system before being removed
* `PersistLevel`: a scale of how restrictive the persistence should be
    0. all endpoints are saved
    1. only endpoints that we have connected to are persisted
    2. only endpoints that we have successfully dialed are persisted

### Special Endpoints

It's possible to configure a list of special endpoints by either passing them directly via the Configuration or calling Network's AddSpecial method. The format is ip and port, separated by comma: `ip:port,ip2:port2,ip3:port3`. Designating an endpoint as special will mostly affect dialing out.

The network will always try to dial out to a special endpoint until it has a connection and these connections don't count toward the regular outgoing limit. For incoming connections, any connection from an ip address that has a special endpoint will bypass the maximum incoming limit, since the port is not known at this stage. Nodes that have each other as mutual special endpoints should always have exactly one connection between each other. 

Additionally, when fanning out a broadcast, special peers will always be sent parcels in addition to the regular fanout. 

## Protocol

### Parcel Types

|Val|Name|Payload|Description|
|----|----|-----------|---|
|0|Heartbeat|-|Deprecated|
|1|Ping|"Ping"|Sent after a period of inactivity to check for timeouts. Node should reply with a Pong.|
|2|Pong|"Pong"|Response to a Ping|
|3|PeerRequest|"Peer Request"|Indicates the sender wants to be informed of peers|
|4|PeerResponse|protocol specific|Sends a partial peer view|
|5|Alert|-|Deprecated|
|6|Message|app data|Carrier for application specific data|
|7|MessagePart|app data|A partial message for a payload split over multiple messageparts|
|8|Handshake|"Handshake"|The first message sent across the wire|

**Note 1**: V9 sends all messages as MessagePart but does not split up messages into parts for payloads smaller than 1 GB. 

**Node 2**: The Handshake type requires a V9 Parcel


### Handshake

The handshake starts with an already established TCP connection.

1. A deadline is set for both reading and writing according to the configuration
2. Generate a Handshake containing our preferred version, listen port, network id, and node id and send it across the wire
3. Blocking read the first message
4. Verify that we are in the same network
5. Calculate the minimum of both our and their version
6. Check if we can handle that version and initialize the protocol adapter

If any step fails, the handshake will fail. 

For backward compatibility, the Handshake message is in the same format as protocol V9 requests but it uses the parcel type "Handshake". Nodes running the old software will just drop the invalid message without affecting the node's status in any way. For specific format, see below.

### V9

Protocol 9 is the current (Factomd V6.2.2) protocol with the ability to split messages into parts disabled. V9 has the disadvantage of sending unwanted overhead with every message, namely Network, Version, Length, Address, Part info, NodeID, Address, Port. In the old p2p system this was used to post-load information but now has been shifted to the handshake.

Data is serialized via Golang's gob.

#### V9Msg

|Name|Type|Description|
|----|----|-----------|
|Header|V9Header|The header sub-structure. See below|
|Payload|[]byte|Binary payload|

#### V9Header
|Name|Type|Description|
|----|----|-----------|
|Network|NetworkID (uint32)|Identifies the network the message is for|
|Version|uint16|The version of the protocol: `9`|
|Type|ParcelType (uint16)|The parcel's type (0-7)|
|Length|uint32|The byte-length of the payload|
|TargetPeer|string|The sender's internal identifier for the recipient's node, used for internal routing|
|Crc32|uint32|The CRC32 checksum of the payload|
|PartNo|uint16|The index of the parcel's position in a MessagePart type parcel. 0 otherwise|
|PartsTotal|uint16|The total number of parts in a MessagePart type parcel, 1 for single-part messages|
|NodeID|uint64|The internal node id the sender has selected|
|PeerAddress|string|The public ip address that the sender sees the recipient as|
|PeerPort|string|The port the sender is listening on for new connections|
|AppHash|string|Set by the application|
|AppType|string|Set by the application|

#### V9Share
The payload of a PeerShare parcel is a json-encoded list of V9Share structs:

|Name|Type|Description|
|----|----|-----------|
|QualityScore|int32|The sender's quality score of the endpoint|
|Address|string|The IP address|
|Port|string|The listening port|
|NodeID|uint64|The sender's internal id for the peer connected to the endpoint|
|Hash|string|The sender's internal hash for the peer connected to the endpoint|
|Location|uint32|The ip address in decimal format|
|Network|NetworkID (uint32)|The network identifier|
|Type|uint8|The endpoint type. See below|
|Connections|int|Deprecated|
|LastContact|time.Time|Time of last contact|
|Source|map[string]time.Time|Map of where and when the sender has seen the endpoint|

Types:

|Value|Description|
|-----|-----------|
|0|Regular endpoint|
|1|Special endpoint added by the config file|
|2|Special endpoint added by the command line|

**Note 1**: Most of the fields, especially QualityScore and Source, are both ignored by the recipient in the legacy code


### V10

Protocol 10 is the slimmed down version of V9, containing only the Type, CRC32 of the payload, and the payload itself. Data is also serialized via Golang's gob.

The goal was to move all data that is only required to be checked once and doesn't change during the lifetime of a node to the handshake, reducing the overhead.

#### V10Msg

|Name|Type|Description|
|----|----|-----------|
|Type|ParcelType (uint16)|The parcel's type (0-8)|
|Crc32|uint32|The CRC32 checksum of the payload|
|Payload|[]byte|Binary payload|

#### V10Share
The payload of a PeerShare parcel is a json-encoded list of V10Share:

|Name|Type|Description|
|----|----|-----------|
|Address|string|The ip address of the endpoint|
|Port|string|The port of the endpoint|


# Implementation

The code is written in Go to integrate into factomd. It is a standalone, MIT Licensed library that can be used by other applications that want to operate their own semi-gossip network or use it as a base for other types of nodes. 


The current state of the code is functional and can be seen at https://github.com/WhoSoup/factom-p2p. Integration into factomd can be seen at https://github.com/WhoSoup/factomd/tree/FACTOMIZE_new_p2p

An overview of the implementation is available in a loose chart form: https://i.imgur.com/bwvSRXg.png.

Olive arrows indicate there is a 1:n duplication loop. Red arrows indicate a new goroutine is started for this process. 

# Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
