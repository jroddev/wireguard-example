# WireGuard Example

## Running Wireguard
For the client use `./client/wg0.conf`

### Start wireguard
```
$ sudo wg-quick up ./server/wg0.conf
```

### Status
You can automatically refresh this info with `watch -n 1 sudo wg`
```
$ sudo wg
interface: wg0
  public key: yWWnblG8B9NzDQXWDPGBzNOXbaXPGGJSd3CYF495ElA=
  private key: (hidden)
  listening port: 51820

peer: 1YbkY/CKdVlEbTSaFr2S1P12WI3cE/ZQTZft+Zg/BDo=
  endpoint: 192.168.20.71:57904
  allowed ips: 10.0.0.2/32
  latest handshake: 9 minutes, 49 seconds ago
  transfer: 49.45 KiB received, 15.18 KiB sent
```

### Test the connection
```
// from client
$ ping 10.0.0.1
PING 10.0.0.2 (10.0.0.1) 56(84) bytes of data.
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=1.25 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=1.38 ms

// connect to a listening port on server
$ nc -vz 10.0.0.1 8080
Connection to 10.0.0.1 port 8080 [tcp/http-alt] suceeded!

// from server
$ ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=1.16 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=1.44 ms

// see the path taken (same network in my case)
$ traceroute 10.0.0.2
traceroute to 10.0.0.2 (10.0.0.2), 30 hops max, 60 byte packets
 1  * * 10.0.0.2 (10.0.0.2)  8.158 ms

// see that the subnet is routed through wg0
$ ip route | grep 10.0
10.0.0.0/24 dev wg0 proto kernel scope link src 10.0.0.1
```


### Stop wireguard
```
$ sudo wg-quick down ./server/wg0.conf
```

--- 

### IP Forwarding
<b>Untested</b>
- Enable IPv4 and IPv6 forwarding
```
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1
sudo sysctl -p
```

- Setup NAT rules to forward wg0 traffic to eth0
```
sudo systemctl start nftables
sudo systemctl enable nftables
```
- Add this config to /etc/nftables.conf
```
#!/usr/sbin/nft -f

flush ruleset

table ip nat {
    chain prerouting {
        type nat hook prerouting priority 0; policy accept;
    }

    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        oifname "eth0" masquerade
    }
}
```
- Reload the rules
```
sudo nft -f /etc/nftables.conf
```

----

### Mac Issues
`AllowedIPs = 0.0.0.0/0, ::/0` doesn't work on MacOS with the App Store GUI App as it creates a routing loop.  
Use `AllowedIPs = 0.0.0.0/1, 128.0.0.0/1` instead to enable all IPv4.
IPv6 is more complicated to cover the address range.

### NordVPN Issues
You may need to whitelist the Wireguard subnet and the two VPNs may conflict even when not running
```
nordvpn whitelist add subnet 10.0.0.0/24
```

---

### More Info
- https://www.wireguard.com/
- https://www.wireguard.com/quickstart
- https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04
