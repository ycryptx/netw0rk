# netw0rk

## Protocol Outline
### Participants
- Peers   (Raspberry Pi's w/ WiFi and Bluetooth)
- Clients (smartphones via app)

### Parameters
- T = client authentication time interval (say 24 hours)
- n = the number of peers a client needs to receive authentications from

### Bluetooth Authentication
1. Client establishes connection to a peer via BLE
    - *Research: range of BLE and whether information can be transmitted without forming a connection (via changing the device name)*
2. Peer produces a signature: Sign(PeerID (?) || ClientPubKey || Date) and transmits it back to the client.
3. Client collects n such signatures from n peers.
4. Upon getting S = \[S1,S2,...,Sn] such signatures, the client sends S (SigSet) to the last signing peer.
5. Peers verify the signature set as follows:
    - If the hash, H, of the sorted PeerID list (P = \[P1, P2,...Pn]) exists in SigSet Table, do nothing.
    - Else: Produce a signature, Ši, for the SigSet. Š = (Ši + all other received signatures on SigSet).
    - Add client_id, current_timestamp, and H to SigSet_Table
    - If len(Š) >= n, send S + Š to authenticating client via the mesh network.
    - Else:
        - Directly send S + Š to a peer p ∈ P which can be immediately connected with, and whose SigSet signature ∉ Š.
        - If such a peer cannot be found, drop S + Š.
          - *Research: How to efficiently 'directly' connect with a peer (socketing?)*
          - *Research: can we rectify the scenario when len(Š) < n and an immediate peer cannot be found*

 #### SigSet_Table
  | ClientID        | Timestamp     | SigSet Peer Hash  |
  | --------------- |:-------------:| -----------------:|
  | 1234            | 1548718597    | afh13roidn9       |
  | 5678            | 1548703652    | grepk1239fnz      |

    - Remove row_i from table if current_timestamp - timestamp_i > n

### Mesh Networking

#### Forwarding
- ~~Packets are only forwarded if BT Auth is valid.~~ Inter-peer communication is unrestricted, that is, peers can forward packets between each other with no restrictions.
- Entry and Exit peers (the peer who first receives a packet from a client, and the peer who forwards the packet to its destination), are the only ones who perform authentication verification. This includes:
    - BT Auth verification, once per session.
    - Client Signature verification on every packet
- A valid BT Auth:
    - for s ∈ SigSet, packet_userid == s(sigset_userid)
    - is a SigSet which is itself signed by all signing peers.
    - has current_timestamp - min_timestamp(SigSet) < n. min_timestamp is the timestamp of the oldest signature ∈ SigSet
    - for s ∈ SigSet, s(PeerID) ∈ Peer_Table, and s is a valid signature for PeerID
- *Research: can the BT Auth be made smaller? SigSet + Š seems quite big*
- Use CJDNS for forwarding?


#### Peering
-

#### Peer_Table
  | Peer Public Key |
  | --------------- |
  | xzalskd-0       |
  | zxqwfpo-2       |







## Resources
https://www.ralfebert.de/ios/tutorials/multipeer-connectivity/

https://www.bluetooth.com/bluetooth-technology/topology-options/le-mesh/mesh-tech

https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/NetServices/Introduction.html

http://zeromq.org/bindings:swift-binding

https://github.com/ethereum/devp2p/blob/master/rlpx.md

https://developer.apple.com/documentation/networkextension/nehotspothelper

https://crypto.stanford.edu/~dabo/pubs/papers/BLSmultisig.html

https://github.com/asonnino/bls

https://github.com/warner/python-ecdsa
