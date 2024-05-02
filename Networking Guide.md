Networking cheatsheet for dummies (blurb here)

---
# What Even Is the Internet?

The Internet is made up of computers each forming networks, and smaller networks combining to form bigger networks. There's your computer's internal network, the connection from your computer to the nearest switch or access point, to your router, your ISP, and to the wider internet.

It's often helpful to think of things in multiple layers. There's the physical layer of ethernet or wifi, the protocols used to organize and send data on your local network or LAN, and the IP and BGP routing layer used between networks. And then the application layer, where your HTTP requests or emails go.


# The Link Layer

The first 'layer' of a network is the physical medium by which the information is carried, like radio, copper wire, or fiberglass, but that is a bit beyond the scope of this article, so let's skip to the second layer, the `link layer`. 

The link layer is the protocols used to interpret whatever data was sent over the physical hardware, the physical layer handles converting electrical impulses or light in a fibre to bits and bytes, the link layer is what reads and interprets those bits

Although there are many diffrent options, Ethernet is one of the most common link layer protocols being used, and also one of the easiest to explain, so let's start there. 
## Packets 

When you connect to another computer over the internet, your data isn't sent all at once over one path, it's split into many chunks called `packets`. Each packet can take its own path through the network, and get recombined on the other end.

 Packet switched networks allow one physical link to carry more than one connection at a time.

>sidenote 
>This is because the internet was designed to be resilient to failure. Some of the core components are inventions by the US military, and one of the primary design constraints was being able to survive an attack that took out connections. 

Packets usually consist of `headers`, `data`, and a `checksum`. Headers are the metadata added by each layer handling the packet, like address information, or timeouts. The data is, well, the data to be sent to the other computer, the checksum is a small number at the end that can be used to verify the data got there uncorrupted.

//todo link to thing about ecc l8r'

Ethernet headers usually include the destination and source MAC addresses, along with a type. 
//todo find out wtf the type does
// TODO diagram here of packet
## Ethernet Switching

MAC addresses are (hopefully) unique IDs for every networking card in the world, each manufacturer is assigned a block of addresses they can assign to the cards they made, and are used to tell what computer is at the other end of any connection.

In unswitched networks, all packets are sent to all network cards, and it is up to the network card to filter any packets sent to the host computer. Network cards can cheat, however, and read any incoming packets regardless of the intended destination. This used to be quite the security hole before switching and encryption became common

>sidenote: 
>The act of snooping on all packets sent through a network is called `packet sniffing`, and WIFI is still vunrable to this as it isnt a switched protocol

In switched networks, there is a computer whose job it is to remember where each MAC address is in a network and send it on, called a `switch`. Switches can still snoop on your packets, but atleast it isnt everyone on the network.
### Datagram Forwarding

To do that, they use internal `routing tables`. Routing tables are tables that take a destination address, and give the `next hop` or the address of the closest machine to the destination it is connected to.

// todo diagram & table example here

They can also come with default routes, where they send any traffic they are unsure of. These usually lead to other switches closer to the center of the network, which would have a more complete view of the where all the computers are.

// todo, diagram here
// todo, 'defaultless routing' explenation

There is also the special case of the `broadcast address`, which a switch would forward to everyone. These are generally used when you are first connecting to a network and are unsure 
of what addresses to use

>sidenote
>you might have noticed that nowhere in here have i described a method of dealing with errors or packet loss, thats because there isnt any, Ethernet (and IP) are send and pray protocols. Packets go missing all the time.

## LANs and vLANs

The part of a network accessible by only link layer switches is called your `Local Area Network` or LAN. There is also your `broadcast domain`, or whatever would recive a broadcast packet sent by you.

Switch software often comes with the ability to logically separate different physical parts of a LAN into separate broadcast domains or `vLANS`.  These are most often used to segment networks based on security level, you don't want your guest wifi to be able to access your security cameras, and logical separation allows them to all share one physical network. 

You can also do the opposite, tying together multiple physical LANs into one vLAN. This is less common but is often helpful for making older pieces of equipment like office printers available from across a campus where it would otherwise be unreachable.
# The Network Layer

In my opinion, the `Network Layer` is badly named. It should be the `Inter-Network` layer. It's the layer that handles routing between various LANs across an internet. To do that, we use the `Internet Protocol` or IP for short. IP gives a way to address and route to any computer connected, and is designed to be compatible with a wide veriaty of link layer protocols.
## IP Addresses

IP address are numbers assigned to ian individual network interface, there are two main types of IP addresses, IPv4 and IPv6 addresses. IPv4 is 32bit, usually written out in decimel with each byte being sepereated by a period. ex, `192.168.100.1

IPv6 addresses are 128 bit, and usually written in hexidecimel and each 16 bits split by colons, ex, `1111:2222:3333:4444:5555:6666:7777:8888`

> sidenote
> IPv4, being 32 bit, has only around 4billion addresses available. Problem, there are *far* more than 4billion devices connected to the internet, IPv6 attemptes to solve that issue by making a 128 bit address space. It hasnt yet replaced IPv4 because its not backwards compatible, and that NAT got *really really* good. 
## Subnets

IP  was designed to be scaleable, no one router needs to know where everything is in a network, and no one authority needs to keep track of every IP. To help with this, IPs are split into blocks, and blocks are given to people to to assign as they see fit.

>sidenote
>This is how people can find where you live based on an IP, what blocks are assigned to an ISP is public information, although it is not the most precise form of geolocation. 

To describe a ip block or `subnet` you can put a slash at the end of any address followed by the number of bits that are fixed in the address block. For example, if you wanted to describe the range from `192.168.0.0` to `192.168.255.255`, the first 16 bits stay the same, so `192.168.0.0/16`

Subnets are *hierarchical*, for example, facebook might use the whole `123.0.0.0/8` block for their whole network, and assign various subnets to datacenters in different cities, say `123.100.0.0/16` for New York, and `123.200.0.0/16` for Boston. 

## IP Forwarding

IP Routers use Datagram Forwarding like Ethernet Switching, but instead of targeting individual computers, they route to whatever the closest subnet is. 

For example, if you are routing to `123.100.200.300`, the first few routers would go up the `default` path to some router at your ISP. The ISPs routers would then forward the traffic to whatever their routing tables say is closest to the `123.0.0.0/8` subdomain. When it reaches a server that knows where `123.100.0.0/16` is, it  would then route there and so on.

//todo we DEF need a digram here

There are also broadcast IP addresses, usually either `255.255.255.255` for 'brroadcast to this network' or the network address followed by all binary ones for 'broadcast to a specific network'
However, if the broadcast request comes from outside the network, it is usually ignored, if it even makes it there in the first place.


## NAT and CGNAT

IP originally assigned every computer a globally uniqe address, like MAC Addresses, and IPv6 still works that way. But the IPv4 address space is  far too small for that to be viable. To solve this the IPv4 standard sets aside 'Private Address Subnets' that cannot reach the public internet, and are free to be reused.

```
192.168.0.0/16
172.16.0.0/12
10.0.0.0/8
100.64.0.0/10 (Only for mobile carriers)
```

If you are assigned a private IP address, and try to make a request outside your local network, the router can let you borrow its public IP address on a certain port to temporarily use. When requests come in or out, the router 'translates' from the public shared address to the private addresses.

>sidenote
>Ports are a TCP thing, skip down to there if you need an explanation *right this seccond*, right now all you need to know is that they allow you to split traffic going to the same IP

Carrier Grade NAT or CGNAT is when an ISP does the same thing, meaning there are two layers of translation between your computer and the wider internet. 

Because the address you are given through NAT arent your own, you cannot wait and listen for requests coming *in* to the network, meaning that your computer is inaccessible to the outside world unless you initiate the connection first.

There are a few ways of getting around this, but the most common is `port forwarding`, where you tell the router to keep a port free for incomming requests, and pass any along to you. This only works if you have permission from the router owner to do so, and if there isnt another layer of NAT between you and the internet.

IP address collisions can still be a problem, however, especially when using multiple network connections at once, like cellular connection and wifi at once, or your local lan and a vpn to your office, (there isnt any real way to deal with this beyond manually fixing it each time it happens but i want some way to exp)

//diagram here
# The Transport Layer

This layer is badly named again, i think the `Connection Layer` would fit better.
# Reccommended Further Reading

An Introduction to Computer Networks by Peter L Doral, 
(or atleast chapter one, its a brilliant book)

Faster Than Lime ()******