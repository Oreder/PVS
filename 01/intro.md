# Static routing

## 1. Run all virtual machines in same time
```bash
    vstart ws1 --con0=xterm --eth0=networkA
    vstart ws2 --con0=xterm --eth0=networkA
```

## 2. Get network interface information
```bash
    ip link     # full way
    ip l        # short way
```
Here:
- lo: loopback
- teqlo0: support combining multiple interfaces
- eth0: MAC-address

## 3. Get introduction of IP address
```bash
    ip addr     # full way
    ip a        # short way
    ip -4 a     # IPv4
```

## 4. Utility Ping
```bash
    ping 127.0.0.1
```
Here:
- ttl: time to live of packet

## 5. Set IP address and mask network
```bash
    # Setting address of ws1
    ip a add 10.0.0.11/16 dev eth0
    ip l set eth0 up

    # Setting address of ws2
    ip a add 10.0.0.22/16 dev eth0
    ip l set eth0 up

    # When wrong address is given
    ip addr del

    # Cancel all settings
    reboot
```

## 6. Search MAC address and protocol ARP
```bash
    # How to know neighbour address 
    # 1. By packet iproute2:
    ip neighbour
    ip n

    # 2. By packet net-tools
    arp -n
```

## 7. Intercept messages by ARP
```bash
    # ws1
    ip n flush dev eth0     # clear caches
    ping 10.0.0.22 -c 2     # send 2 times

    # ws2
    tcpdump -nt
```
Here:
- seq: sequency (index of request)
- id: identification of eth0 request
- length: length of requested packet

```bash
    # ws2
    tcpdump -e
```
- ethertype: show which protocol is attached in Ethernet frame
- Flag -e in this command to also show  MAC-address of sender, not only IP address

In case of sending to a new IP address of our local network:

```bash
    # ws1
    ping 10.0.0.23 -c 3
    # ws2
    tcpdump -e
```
And we get:
```bash
    # Result from ws1
    --- 10.0.0.23 ping statistics ---
    3 packets trasnmitted, 0 received, +3 errors, 100% packet loss, time 2018ms, pipe 3
    
    # Result from ws2
    a6:f9:52:b6:1e:69 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x806), length 42: arp who-has 10.0.0.23 tell 10.0.0.11
```
We notice that:
- ff:ff:ff:ff:ff:ff means sending to all segment network
- in ws1 we see 100% loss packet

## 8. Display routing table
```bash
   ip route     # full way
   ip r         # short way
```
We get this line below:
```bash
    # ws1
    10.0.0.0/16     dev eth0    proto   kernel  scope   link    src 10.0.0.11
    
    # ws2
    10.0.0.0/16     dev eth0    proto   kernel  scope   link    src 10.0.0.22
```
Here,
- Network access: 10.0.0.0/16
- Interface of our network: eth0
- src [station]: we see source of table routing
- scope: working region, keyword **link** says that this network can only happen when access the given network interface
- **proto kernel**: confirms this line appears due to the actions of kernel

## 9. Ping to unreachable network
```bash
    # Case 1: Out of LAN
    ping 10.10.10.10
    # Result 1
    connect: Network is unreachable

    # Case 2: May be in LAN
    ping 10.0.0.23 -c 5
    # Result 2
    ...
    from 10.0.0.11  icmp_seq=3  Destination Host Unreachable
```

## 10. Clean up
```bash
    # Restart ws
    reboot

    # Shutdown ws
    halt            # First way
    vcrash ws       # Second way

    # Remove virtual disks
    rm *.disk
```

# Test our configuration

## 1. Interfaces
```sh
    # Interface lo is used to connect computer to itself
    auto lo

    # Type of lo is LOOPBACK
    iface lo inet loopback

    # Define Network interface eth0
    auto eth0
    
    # Type of eth0 is STATIC
    iface eth0 inet static
        # Next is IP address
        address 10.X.0.Y
        # Network Mask (long form), short from is just 16
        netmask 255.255.0.0

    # Add gateway to other working station
    gateway 10.X.0.Z
```

## 2. Start
```bash
    ltabstart -d net    # Gnome
    lstart -o "--con0=-xterm" -d net    # Xterm
```

## 3. Change interface from working station
```bash
    mcedit /etc/network/interfaces      # F2 - save, F10 - quit
    service networking restart          # update configuration (or just run everytime when ws is started)
```

## 4. Check routing by default
```bash
    # r2 (after add gateway to interface of ws1)
    tcpdump -tne -i eth0     # check packet is sent to eth0 or not 
```

## 5. Show ip routing from list
```bash
    # default
    ip r list X.Y.Z.T/M
```

## 6. Disconnect network
```bash
    # r2
    ip link set eth1 down
```
After that, adding one more route in r2:
```bash
    # r2
    ip r add 10.20.0.0/16 via 10.100.0.11 dev eth0  # like a cycle
```

## 7. More information of packet
```bash
    # r2 
    tcpdump -tnv -i eth0     # check packet is sent to eth0 or not 
```

## 8. Show trace routing:
```bash
    # ws11 (10.10.0.2) send a packet to ws21 (10.20.0.2)
    # ws11 terminal (to check route add ttl)
    traceroute -n 10.20.0.2
```

## 9. Change MTU segment
```bash
    ip l set dev eth1 mtu 600
```

## 10. Disable PMTU (Path MTU)
```bash
    echo 1 > /proc/sys/net/ipv4/ip_no_pmtu_disc
```

## 11. Send packet with given size
```bash
    # ws11
    ping -c 1 -s 1000 10.20.0.3

    # r1 (hear packet)
    tcpdump -tnv -i eth0 icmp
```