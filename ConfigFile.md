# Another go

## Interfaces

Set up non-loopback interfaces: in `/etc/network/interfaces`

```bash
auto eth0:1
iface eth0:1 inet static
  address 10.0.0.101
  netmask 255.255.255.0
  network 10.0.0.1
  broadcast 10.0.0.255
  gateway 10.0.0.1

auto eth0:2
iface eth0:2 inet static
  address 10.0.0.102
  netmask 255.255.255.0
  network 10.0.0.1
  broadcast 10.0.0.255
  gateway 10.0.0.1

auto eth0:3
iface eth0:3 inet static
  address 10.0.0.103
  netmask 255.255.255.0
  network 10.0.0.1
  broadcast 10.0.0.255
  gateway 10.0.0.1

auto eth0:4
iface eth0:4 inet static
  address 10.0.0.104
  netmask 255.255.255.0
  network 10.0.0.1
  broadcast 10.0.0.255
  gateway 10.0.0.1
```

Then `sudo /etc/init.d/networking restart`.

## Now use a config file

See `test_lb.conf` - taken from cribs in the `src/scripts/vnet` directory - minor tweaking required (putting it all in a `unix-cli` wrapper) to make it work.

Trying to tweak this for our purposes now...

...but it doesn't seem to work - VPP just ignores the config in `unix-cli` :(
