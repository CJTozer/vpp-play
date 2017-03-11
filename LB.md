# Load-balancer module

See `Notes.md` for basic attempts to get things working.

`sudo service vpp restart` to clear config from that.

LB is maglev basically.  Uses GRE to send out packets to AS.

## Set-up LB Plugin

* `ip4-src-address` is the address to send encapsulated packets from
* `buckets` is the number of connections cached (I think?) - must be 2^n

```bash
vppctl lb conf ip4-src-address 127.0.0.3 buckets 32
```

To view config: `vppctl show lb`

* Set up the VIP (the receiving end) and the encapsulation type

```bash
vppctl lb vip 127.0.0.2/32 encap gre4
```

* Add an AS for the VIP

```bash
vppctl lb as 127.0.0.2/32 127.0.0.3
```

To view that config: `vppctl show lb vip verbose`

## Set up interfaces

Just like in normal set-up before...

```bash
vppctl set interface ip address host-lo:1 127.0.0.2/8
vppctl set interface ip address host-lo:2 127.0.0.3/8
vppctl create host-interface name lo:1
vppctl create host-interface name lo:2
vppctl show interface
vppctl set interface state host-lo:1 up
vppctl set interface state host-lo:2 up
vppctl show interface
```

## Debug (1)

Trace now shows what looks like LB module getting involved - picking `127.0.0.3` to send the packet out, then failling.

Presumably the problem may be the lack of GRE tunnel - needs setting up?

```
00:08:17:127578: af-packet-input
  af_packet: hw_if_index 1 next-index 4
    tpacket2_hdr:
      status 0x9 len 74 snaplen 74 mac 66 net 80
      sec 0x58c410dc nsec 0x365d4918 vlan 0 vlan_tpid 0
00:08:17:127587: ethernet-input
  IP4: 00:00:00:00:00:00 -> 00:00:00:00:00:00
00:08:17:127591: ip4-input
  TCP: 127.0.0.1 -> 127.0.0.2
    tos 0x00, ttl 64, length 60, checksum 0xeb25
    fragment id 0x5193, flags DONT_FRAGMENT
00:08:17:127595: ip4-lookup
  fib 0 dpo-idx 0 flow hash: 0x00000000
  TCP: 127.0.0.1 -> 127.0.0.2
    tos 0x00, ttl 64, length 60, checksum 0xeb25
    fragment id 0x5193, flags DONT_FRAGMENT
00:08:17:127601: lb4-gre4
  lb vip[0]: ip4-gre4 127.0.0.2/32 new_size:1024 #as:1
lb as[1]: 127.0.0.3 used

00:08:17:127639: ip4-load-balance
  fib 0 dpo-idx 9 flow hash: 0x00000000
  GRE: 127.0.0.3 -> 127.0.0.3
    tos 0x00, ttl 128, length 84, checksum 0x3c75
    fragment id 0x0000
  GRE ip4
00:08:17:127648: ip4-drop
    GRE: 127.0.0.3 -> 127.0.0.3
      tos 0x00, ttl 128, length 84, checksum 0x3c75
      fragment id 0x0000
    GRE ip4
00:08:17:127651: error-drop
  ip4-input: ip4 adjacency drop
```

## GRE

From [this site](http://ask.xmodulo.com/create-gre-tunnel-linux.html)...

Set up the receiving end of the GRE tunnel (assuming for now that VPP sets up the sending end).

Target at the end of the tunnel gets the `127.0.0.2` address (on another namespace?)...

```bash
sudo ip tunnel add gre1 mode gre remote 127.0.0.3 local 127.0.0.4 ttl 255
sudo ip link set gre0 up
sudo ip addr add 10.10.10.2/24 dev gre0
```

Now need the LB to send to `127.0.0.4` using `127.0.0.3` - at the moment the AS is `127.0.0.3`...

...moving all this to `build_lb_config` script...
