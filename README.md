# 🔐 Telnet vs SSH: Network Security Analysis with Wireshark
A hands-on network security project demonstrating the vulnerability of Telnet protocol versus the encryption strength of SSH. Built using GNS3 network simulator with Cisco IOS routers, and analyzed using Wireshark packet capture.

There's a moment in networking that changes how you think about protocols forever.

You type a password into a terminal. Then you open Wireshark, follow a TCP stream, and there it is — your password, sitting in plain text, completely readable by anyone on the network.

That's Telnet. And that's exactly why this experiment is worth doing.

In this post, I'll walk you through how I set up a 4-router network in GNS3, configured Telnet and SSH on Cisco routers, and used Wireshark to visually prove why one protocol is dangerous and the other is essential.

---

## The Problem: Why Does This Matter?

Most people learn that "SSH is better than Telnet" from documentation. But seeing it live is different.

Telnet operates on **Port 23** and sends everything — usernames, passwords, commands — as unencrypted plain text over the network. SSH operates on **Port 22** and encrypts every single byte using RSA key pairs.

In a real-world scenario, any attacker with access to your network traffic (via ARP spoofing, a rogue device, or misconfigured infrastructure) can intercept Telnet credentials instantly. SSH prevents this entirely.

---

## Network Topology

Here's the topology I built in GNS3:

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8764xkfomy07gkdt3o2z.png)

**IP Address Summary:**

| Device | Interface | IP Address |
|--------|-----------|------------|
| R1 | f0/0 | 172.16.2.16/16 |
| R2 | f0/0 | 172.16.2.33/16 |
| R2 | f2/0 | 172.25.177.254/16 |
| R2 | f3/0 | 172.6.16.6/16 |
| R3 | f0/0 | 172.6.16.17/16 |
| R3 | f1/0 | 172.34.5.10/16 |
| R4 | f0/0 | 172.25.4.192/16 |
| R4 | f1/0 | 172.2.2.2/16 |

---

## Start Wireshark Before Anything Else

Before touching any configuration, start packet capture on every router link in GNS3. Right-click on each link → **Start Capture**. This ensures Wireshark records all configuration traffic from the very beginning.

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qvuymj32j0giy5f01qfo.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gmeopwbylt9qtjgrj8zn.png)

---

# PHASE 1 — Configure IP + RIP on All Routers (Direct Console)

## R1 Console:


```
Router> enable
Router# configure terminal

Router(config)# hostname R1
R1(config)# interface fastethernet 0/0
R1(config-if)# ip address 172.16.2.16 255.255.0.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 172.16.0.0
R1(config-router)# no auto-summary
R1(config-router)# exit
R1(config)# exit

R1# write memory
```

R1 is our "management router" — everything else gets configured from here via Telnet.

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pdluhzor3xcgoydfuut2.png)

The `no shutdown` command is essential — Cisco router interfaces are **administratively down** by default. Without this, nothing works.

Configure RIP on R1 : 

>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pz9ksiaz34bq89m0rfps.png)

What Actually Happen in wireshark After Configure R1

>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zh567pkrm2gstb20h0aq.png)

## R2 Console:

```
Router> enable
Router# configure terminal

Router(config)# hostname R2
R2(config)# interface fastethernet 0/0
R2(config-if)# ip address 172.16.2.33 255.255.0.0
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 172.16.0.0
R2(config-router)# no auto-summary
R2(config-router)# exit

R2(config)# enable password cisco
R2(config)# line vty 0 4
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# transport input telnet
R2(config-line)# exit
R2(config)# exit

R2# write memory
```
>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/056gt65l0c05zx67k9q2.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wd8383qcz2shvx5nvv0z.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ipw7ar2tcfqnmyglnm1j.png)

**What are VTY lines?** They're virtual terminal lines — the "doors" through which remote users connect to a router. `line vty 0 4` means we're configuring 5 simultaneous connections (0 through 4).

## R3 Console:

```
Router> enable
Router# configure terminal

Router(config)# hostname R3
R3(config)# interface fastethernet 0/0
R3(config-if)# ip address 172.6.16.17 255.255.0.0
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# router rip
R3(config-router)# version 2
R3(config-router)# network 172.6.0.0
R3(config-router)# no auto-summary
R3(config-router)# exit

R3(config)# enable password cisco
R3(config)# line vty 0 4
R3(config-line)# password cisco
R3(config-line)# login
R3(config-line)# transport input telnet
R3(config-line)# exit
R3(config)# exit

R3# write memory
```

>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c3mbcpv9xjdp3lg34kmq.png)

## R4 Console:

```
Router> enable
Router# configure terminal
Router(config)# hostname R4
R4(config)# interface fastethernet 0/0
R4(config-if)# ip address 172.25.4.192 255.255.0.0
R4(config-if)# no shutdown
R4(config-if)# exit
R4(config)# router rip
R4(config-router)# version 2
R4(config-router)# network 172.25.0.0
R4(config-router)# no auto-summary
R4(config-router)# exit
R4(config)# enable password cisco
R4(config)# line vty 0 4
R4(config-line)# password cisco
R4(config-line)# login
R4(config-line)# transport input telnet
R4(config-line)# exit
R4(config)# exit
R4# write memory
```
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/etdwdo1ly1n8wx5cgbsi.png)

---
# PHASE 2 — Configure PCs (VPCS)


## PC1:

```
ip 172.2.3.3 255.255.0.0 172.2.2.2
save
```

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u4dbodqobtq42wrcxjkz.png)

## PC2:

```
ip 172.34.10.67 255.255.0.0 172.34.5.10
save
```
>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ihsfdf308nnm3xpvk8w4.png)

---

# PHASE 3 — From R1, Telnet into R2, R3, R4 and Configure Remaining Interfaces + RIP


## ✅ First Verify — Ping from R1:

```
R1# ping 172.16.2.33
R1# ping 172.6.16.17
R1# ping 172.25.4.192
```
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kb575pmf4v25dfafrzpu.png)

All three must succeed before proceeding.

## R1 → Telnet → R2:

```
R1# telnet 172.16.2.33
```

```
Password: cisco
R2> enable
Password: cisco
R2# configure terminal

R2(config)# interface fastethernet 2/0
R2(config-if)# ip address 172.25.177.254 255.255.0.0
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# interface fastethernet 3/0
R2(config-if)# ip address 172.6.16.6 255.255.0.0
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# router rip
R2(config-router)# network 172.25.0.0
R2(config-router)# network 172.6.0.0
R2(config-router)# exit

R2(config)# exit
R2# write memory
R2# exit
```

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gvrg5f64qdkqrv8381w6.png)
Wireshark will show the packets exchanged between the two routers. Right click on the 2nd last telnet packet and got to Follow -> TCP stream. You can see the console commands entered on router 1 to access router 2. It will also show the password as plaintext.  If we had configured SSH instead of Telnet then the password wouldn’t be captured as plaintex
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gss3aoggw9977wqicsx1.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mkz1bkt66cd1z1lj8har.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vezxy9ju6bxyl0qhcre3.png)

>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5na6bxylyonpguta2pgw.png)
>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/47u21rjobgczvrs168fv.png)
>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nmiv411pgoytr5d44oz2.png)

Full Command 
>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p3hf87ljzw11xef80cny.png)

## R1 → Telnet → R3:

```
R1# telnet 172.6.16.17
```

```
Password: cisco
R3> enable
Password: cisco
R3# configure terminal

R3(config)# interface fastethernet 1/0
R3(config-if)# ip address 172.34.5.10 255.255.0.0
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# router rip
R3(config-router)# network 172.34.0.0
R3(config-router)# exit

R3(config)# exit
R3# write memory
R3# exit
```

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0eh8mfo4oousi2v4yulj.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1uiw87encfovwq27502r.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qzja270fexeugpcpdqmp.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yjvqo01g4nraxypj893q.png)

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3enx8tc2pid5aj2k7l3n.png)

## R1 → Telnet → R4:

```
R1# telnet 172.25.4.192
```

```
Password: cisco
R4> enable
Password: cisco
R4# configure terminal

R4(config)# interface fastethernet 1/0
R4(config-if)# ip address 172.2.2.2 255.255.0.0
R4(config-if)# no shutdown
R4(config-if)# exit

R4(config)# router rip
R4(config-router)# network 172.2.0.0
R4(config-router)# exit

R4(config)# exit
R4# write memory
R4# exit
```

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1a8geuva9p9m1d2525ek.png)

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8or9ulfmuvmui2wh9w0r.png)
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/peab5228o1wfdcl4tfe9.png)

---

# PHASE 4 — Configure SSH on R4 (R4 Direct Console)

```
R4# configure terminal
R4(config)# ip domain-name lab.com
R4(config)# crypto key generate rsa
```

When prompted:

```
How many bits in the modulus [512]: 1024
```

```
R4(config)# ip ssh version 2
R4(config)# username admin password cisco123
R4(config)# line vty 0 3
R4(config-line)# transport input ssh telnet
R4(config-line)# login local
R4(config-line)# exit
R4(config)# exit
R4# write memory
```
**Why 1024 bits?** SSH version 2 requires a minimum key size. Less than 768 bits won't work. 1024 is the standard starting point, though 2048+ is recommended for production environments.

**`login local` vs `login`:** Using `login local` tells the router to check the local username database we just created. Plain `login` only checks the VTY password — less secure.

>![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1yn8ceq6bsgcevqk56yd.png)![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o5ev6pig5zkd8poq3zfk.png)![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mcpglqzc7l0yya3cph3x.png)

After running configuration commands through Telnet, go to Wireshark:

1. Find any Telnet packet
2. Right-click → **Follow** → **TCP Stream**

You'll see the password typed at the prompt — completely readable. Every command. Every keystroke. All plain text.

This is the moment that makes the SSH comparison real.

---

# PHASE 5 — SSH from R2 into R4

```
R2# ssh -l admin 172.25.4.192
```

```
Password: cisco123
R4> enable
Password: cisco
R4# show running-config
```

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jeotjdxfufx8h0unww3w.png)

Follow the same steps as before — find an SSH packet, right-click → **Follow** → **TCP Stream**.

Compare this with the Telnet stream. The SSH stream shows only encrypted bytes — no readable text, no visible commands, no password.

Same network. Same routers. Completely different security story.

> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/axl06p8xhvc3cthxafd7.png)
> 
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yfksicr0y1wi5z046v1w.png)
> 
> All Encrypted Data : 
> ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/io7ohcke8879l631nakb.png)

---

# Final Verification

## Run on every router (R1, R2, R3, R4):

```
show ip interface brief
show ip route
show running-config
```

With RIP v2 configured on all routers, they share routing information automatically. After about 30–60 seconds for convergence:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3ql92dil4w132njodqba.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ua222160vrijpb7rb84k.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a87xra2rjav0ho3uu8bq.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gzvo74pu3gdgn8qmodrd.png)

---
## Ping PC2 from PC1:

```
ping 172.34.10.67
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p4ojb2ok4s5n54jwbl2s.png)

---

# 📝 Important Notes — Lessons Learned

### ⚠️ Note 1: RIP First, Telnet Later

Every router that you want to **telnet into** must have RIP configured on it beforehand — from its own console.

Reason: Telnet works over a TCP connection. For TCP to establish, packets must travel both ways. For packets to travel, routes must exist. Routes come from RIP. No RIP means no route, no route means no ping, no ping means no Telnet — connection will always time out.

> ✅ Correct order per router: Console → Assign IP → Configure RIP → Configure Telnet → Then telnet in from another router

### ⚠️ Note 2: The Chicken and Egg Trap

```
Want to Telnet in    → Need Ping to work
Need Ping to work    → Need a Route
Need a Route         → Need RIP
Want to configure RIP → Need Telnet  ← 🔴 INFINITE LOOP!
```

The only way out of this loop is to configure RIP from the console first, before attempting any telnet connection.

---
## Telnet vs SSH: Side by Side

| Feature               | Telnet        | SSH                   |
| --------------------- | ------------- | --------------------- |
| Port                  | 23            | 22                    |
| Encryption            | None          | RSA + AES             |
| Password in Wireshark | Fully visible | Encrypted             |
| Data Integrity        | No            | Yes                   |
| Authentication        | Password only | Password + Public Key |
| Compliance            | Fails PCI DSS | Meets PCI DSS, HIPAA  |
| Industry Status       | Deprecated    | Standard              |

---
## What We Learned

**The technical takeaways:**
- Cisco router interfaces are administratively down by default — always use `no shutdown`
- SSH requires a domain name and RSA key pair before it will function
- `transport input ssh` blocks Telnet entirely; `transport input ssh telnet` allows both
- RIP v2 needs 30–60 seconds to converge before routing works across the full network
- Wireshark's "Follow TCP Stream" feature is one of the most powerful tools for understanding protocol behavior

**The bigger picture:**
- Seeing credentials in plain text in a packet capture is more convincing than any documentation
- Most legacy network devices still run Telnet by default — knowing how to replace it with SSH is a practical skill
- Tools like Wireshark aren't just for attackers — they're essential for anyone building or defending networks

---
## Common Mistakes for Beginners

**1. Starting Wireshark after configuration**
Always start captures before any configuration. You can't capture traffic retroactively.

**2. Skipping `no shutdown`**
If interfaces stay down, nothing will connect. Make this a habit on every interface.

**3. Using less than 1024-bit RSA keys**
SSH v2 won't work. Always use 1024 minimum, 2048 for anything serious.

**4. Expecting instant ping success**
RIP needs time to advertise routes to all routers. Wait 30–60 seconds after configuration.

**5. Mixing up `login` and `login local`**
`login local` uses the username database. `login` uses only the VTY password. For SSH, always use `login local`.

---

*All configurations were done in GNS3 with Cisco IOS routers. Wireshark was used for packet analysis.*


---

## 🌐 Connect With Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://www.linkedin.com/in/almahmudkhalif/)
[![Dev.to](https://img.shields.io/badge/Dev.to-Articles-black?logo=devdotto)](https://dev.to/almahmudkhalif/)

