# VPP Meddling

## Install/build

```bash
git clone ssh://CJTozer@gerrit.fd.io:29418/vpp.git
cd vpp
./build-root/vagrant/build.sh
sudo dpkg -i build-root/*.deb
sudo service vpp start
which vppctl
sudo vppctl show version
vppctl show interface
```

## Create a couple of loopback interfaces on the host
```bash
sudo ifconfig lo:0 127.0.0.2 netmask 255.0.0.0 up
sudo ifconfig lo:1 127.0.0.3 netmask 255.0.0.0 up
ip addr
```

## Create host interfaces for lo in VPP

```bash
vppctl create host-interface name lo:1
vppctl create host-interface name lo:2
vppctl show interface
```

## Bring these 'up'

```bash
vppctl set interface state host-lo:1 up
vppctl set interface state host-lo:2 up
```

## Netcat to send data

Aim is to get data sent to `127.0.0.2` to be forwarded to `127.0.0.3`...

### Sender
```bash
nc -v4 127.0.0.3 3000
```
(this does TCP, add `-uv4` instead for UDP)

### Listener
```bash
nc -v4lk 127.0.0.3 3000
```
(this does TCP, add `-uv4lk` instead for UDP)

## Bind(?) VPP interfaces to addresses

```bash
vppctl set interface ip address host-lo:1 127.0.0.2/8
vppctl set interface ip address host-lo:2 127.0.0.3/8
```

## Trace

```bash
vppctl trace add af-packet-input 8
vppctl show trace
vppctl clear trace
```

## Bridging

Aiming to bridge the 2 loopback interfaces - assume will need some routing later too...?

```bash
vppctl set interface l2 bridge host-lo:1 1
vppctl set interface l2 bridge host-lo:2 1
vppctl show bridge-domain
vppctl show bridge-domain 1 detail
vppctl loop create
vppctl set interface l2 bridge loop0 1 bvi
vppctl set interface state loop0 up
vppctl show bridge-domain 1 detail
```

<!-- Give the `loop0` interface an address (why?)

```bash
vppctl set interface ip address loop0 10.0.0.10/24
```
 -->
## Routing

<!-- ```bash
vppctl ip route table 0 127.0.0.0/24 via loop0
```
 -->

## Results

After the stuff above, looks like I have a packet going from `127.0.0.1` to `127.0.0.2` that is trying to be sent out of `127.0.0.3` - i.e. the other loopback.  But the IP & TCP headers are still unchanged.

```
Packet 1

00:06:05:520061: af-packet-input
  af_packet: hw_if_index 1 next-index 4
    tpacket2_hdr:
      status 0x5 len 74 snaplen 74 mac 66 net 80
      sec 0x58c3ed5d nsec 0x23afa210 vlan 0 vlan_tpid 0
00:06:05:520106: ethernet-input
  IP4: 00:00:00:00:00:00 -> 00:00:00:00:00:00
00:06:05:520122: l2-input
  l2-input: sw_if_index 1 dst 00:00:00:00:00:00 src 00:00:00:00:00:00
00:06:05:520137: l2-learn
  l2-learn: sw_if_index 1 dst 00:00:00:00:00:00 src 00:00:00:00:00:00 bd_index 1
00:06:05:520160: l2-fwd
  l2-fwd:   sw_if_index 1 dst 00:00:00:00:00:00 src 00:00:00:00:00:00 bd_index 1
00:06:05:520166: l2-output
  l2-output: sw_if_index 2 dst 00:00:00:00:00:00 src 00:00:00:00:00:00 data 08 00 45 00 00 3c 11 2a 40 00 40 06
00:06:05:520184: host-lo:2-output
  host-lo:2
  IP4: 00:00:00:00:00:00 -> 00:00:00:00:00:00
  TCP: 127.0.0.1 -> 127.0.0.2
    tos 0x00, ttl 64, length 60, checksum 0x2b8f
    fragment id 0x112a, flags DONT_FRAGMENT
```
