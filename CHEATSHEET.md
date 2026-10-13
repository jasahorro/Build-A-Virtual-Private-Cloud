# Build a Virtual Private Cloud — Cheat Sheet

Lesson notes for reviewing later. The project write-up is in [README.md](README.md). This is the **first project of the NextWork networking series**.

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [VPCs: your private city](#vpcs-your-private-city)
4. [The default VPC](#the-default-vpc)
5. [IP addresses and IPv4](#ip-addresses-and-ipv4)
6. [CIDR blocks](#cidr-blocks)
7. [Subnets and Availability Zones](#subnets-and-availability-zones)
8. [Public vs private subnets](#public-vs-private-subnets)
9. [Internet gateways](#internet-gateways)
10. [Extension: CloudShell and the AWS CLI](#extension-cloudshell-and-the-aws-cli)
11. [Troubleshooting CIDR errors](#troubleshooting-cidr-errors)
12. [Build order and cleanup](#build-order-and-cleanup)
13. [Self-quiz](#self-quiz)

---

## What I built

- 🏙️ **A VPC** called `NextWork VPC` with the CIDR block `10.0.0.0/16`.
- 🏘️ **A subnet** called `Public 1` (`10.0.0.0/24`), with **auto-assign public IPv4** turned on.
- 🚪 **An internet gateway** called `NextWork IG`, attached to the VPC.

```
                          Internet 🌐
                              │
                     ┌────────┴────────┐
                     │   NextWork IG   │  internet gateway (the city gate)
                     └────────┬────────┘
  ┌───────────── NextWork VPC 10.0.0.0/16 (the city) ─────────────┐
  │                           │                                   │
  │   ┌─────────── Public 1 10.0.0.0/24 (a neighbourhood) ──┐     │
  │   │  auto-assign public IPv4: ON                        │     │
  │   │  (EC2 instances launched here get a public IP)      │     │
  │   └─────────────────────────────────────────────────────┘     │
  │                         one Availability Zone                 │
  └───────────────────────────────────────────────────────────────┘
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **AWS Region** | A geographic area where AWS has data centres (e.g. `us-east-1`) | A country |
| **Availability Zone (AZ)** | One cluster of data centres inside a Region | A separate power station in that country |
| **VPC** | Your own private, isolated network inside a Region | Your private city |
| **Subnet** | A slice of the VPC's IP range, inside **one** AZ | A neighbourhood in the city |
| **IP address** | The unique address a resource uses to talk to others | A street address |
| **CIDR block** | A range of IP addresses, written like `10.0.0.0/16` | The postcode area for a neighbourhood |
| **Internet gateway** | Connects a VPC to the public internet | The city gate |
| **Route table** | Rules that decide where a subnet's traffic goes | Road signs |
| **Public IPv4 address** | An address reachable from the internet | A phone number anyone can call |
| **Private IP address** | An address only reachable inside the VPC | An internal office extension |
| **CloudShell** | A terminal in the AWS console with the AWS CLI already installed | A ready-made workbench with the tools laid out |

## VPCs: your private city

- If an **AWS Region is a country**, a **VPC is your own private city** in it. You design its neighbourhoods (**subnets**), traffic rules (**route tables**) and security.
- **Without VPCs**, everyone's resources would sit in one giant open space with no privacy. Think of a Google Drive where every file from every account is dumped into one folder.
- A VPC is **not the internet**. It's private and isolated by default. You connect it to the internet with an **internet gateway**.
- **"Online" ≠ "the internet".** Being online just means being connected to *a* network. The internet is one network among many, alongside corporate networks and cloud networks like AWS.

**Do I always need a VPC?**

| Needs a VPC | Works without one |
|---|---|
| Amazon EC2, and databases like RDS and Aurora | Amazon S3, AWS Lambda (by default) |

## The default VPC

- AWS creates a **default VPC in every Region** when you open your account. It comes with:
  - **one default subnet per AZ** (3 AZs = 3 subnets)
  - **an internet gateway** already attached
- That's why you could launch internet-connected EC2 instances from day one.
- It's a handy starting point, but you build **custom VPCs** for specific needs like strict security.

## IP addresses and IPv4

- **IPv4** addresses are four numbers separated by dots, like `192.168.0.1`. Each number goes from **0 to 255**.
- Each number is **8 bits** (`00000000` = 0, `11111111` = 255), so a full IPv4 address is **4 × 8 = 32 bits**.
- That gives **4,294,967,296** (2³²) possible addresses, from `0.0.0.0` to `255.255.255.255`.
- **Two devices can't share an IP address in the same network**, for example in the same VPC.
- It's not just AWS: laptops, phones, printers and cameras all have IP addresses. Domain names like `nextwork.org` map to IP addresses.

## CIDR blocks

- **CIDR** (Classless Inter-Domain Routing) describes a **block of IP addresses**.
- The number after the slash is how many bits are **fixed**. The rest can vary.
- **The smaller the number after the slash, the bigger the block.**

| CIDR | Fixed part | Range | Addresses |
|---|---|---|---|
| `10.0.0.0/8` | `10` | `10.0.0.0` – `10.255.255.255` | 16,777,216 (2²⁴) |
| `10.0.0.0/16` — *my VPC* | `10.0` | `10.0.0.0` – `10.0.255.255` | 65,536 (2¹⁶) |
| `10.0.0.0/24` — *my subnet* | `10.0.0` | `10.0.0.0` – `10.0.0.255` | 256 (2⁸) |
| `10.0.0.0/28` | 28 bits | `10.0.0.0` – `10.0.0.15` | 16 (2⁴) |

**Quick formula:** number of addresses = 2^(32 − the number after the slash).

- CIDR can be **any number from /0 to /32**. Multiples of 8 are just popular because each part of an IP address is 8 bits.
- **AWS limits:** VPCs and subnets can be from **/16** (biggest) to **/28** (smallest).
- **AWS reserves 5 addresses in every subnet** (the first four and the last one), so a `/24` subnet has **251** usable addresses, not 256.

## Subnets and Availability Zones

- A **subnet** groups resources that share the same access rules. A VPC can have as many as you need.
- **Subnets in the same VPC can't overlap.** Each one needs its own unique IP range.
- A subnet's range must sit **inside** the VPC's range.
- **Each subnet lives in exactly one AZ.** It can't span AZs.
- **Why spread across AZs:** if one AZ has problems, resources in another AZ keep your app running. Every AZ is a backup for the others.

## Public vs private subnets

| | Public subnet | Private subnet |
|---|---|---|
| Internet access | Direct, to and from the internet | No direct internet access |
| Typical use | Public-facing web servers | Databases and other sensitive internal resources |
| Needs | A route `0.0.0.0/0 → internet gateway` (and public IPs on its instances) | No route to an internet gateway |

- **Example web app:** the website's EC2 instance goes in the **public** subnet. The database only needs to talk to that instance, so it goes in a **private** subnet.
- **A name doesn't make a subnet public.** `Public 1` wasn't public until it was connected to an internet gateway.

> **Correction to the lesson:** attaching the internet gateway to the VPC isn't quite enough. A subnet is only public when **its route table** has a route sending `0.0.0.0/0` (all internet traffic) to the internet gateway. That's what the README's extension step covers, and it's covered in more depth in the VPC-Traffic-Flow-and-Security project.

**Auto-assign public IPv4:**
- Every resource gets a **private IP** automatically, but that only works **inside** the VPC.
- To reach the internet, or be reachable from it, an instance needs a **public IP**.
- Turning on **auto-assign public IPv4** for a subnet gives every EC2 instance launched there a public IP automatically, so you don't have to add one by hand.

## Internet gateways

- An **internet gateway** connects your city (VPC) to the outside world (internet). It lets instances reach the internet and lets users reach your apps.
- **Creating ≠ attaching.** A new internet gateway does nothing until you **attach** it to a VPC.
- **One VPC can have only one internet gateway attached** at a time.
- It's **highly available and scales by itself**, so it's not a bottleneck or single point of failure.
- It also **translates between public and private IPs** for instances that have a public IPv4 address.
- The **default VPC already has one**. That's the existing gateway you see in the console.

**Network connections** (the ways a VPC talks to other networks):

| Connection | Connects to | Project |
|---|---|---|
| Internet gateway | The public internet | This one |
| VPC peering | Another VPC | VPC-Peering |
| VPN connection | An external private network, like an office | — |
| VPC endpoint | AWS services such as S3, privately | VPC-Endpoints |

## Extension: CloudShell and the AWS CLI

- **AWS CloudShell** is a terminal in the AWS console. It's already logged in as you and has the **AWS CLI pre-installed**, so there's nothing to set up.
- **The AWS CLI** creates, updates and deletes AWS resources with commands instead of clicks. It's faster, repeatable and scriptable.
- **Why do VPC commands start with `aws ec2`?** VPCs were originally built for EC2 instances, so their commands still live under EC2.

| Command | What it does |
|---|---|
| `aws ec2 create-vpc --cidr-block 10.0.0.0/24 --query Vpc.VpcId --output text` | Creates a VPC and prints **only** its ID |
| `aws ec2 create-subnet --vpc-id vpc-xxxx --cidr-block 10.0.0.0/25` | Creates a subnet inside that VPC |
| `aws ec2 create-internet-gateway` | Creates an internet gateway (not attached yet!) |
| `aws ec2 attach-internet-gateway --internet-gateway-id igw-xxxx --vpc-id vpc-xxxx` | Attaches it to the VPC |
| `aws ec2 create-route --route-table-id rtb-xxxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxxx` | Sends internet traffic to the gateway |
| `aws ec2 modify-subnet-attribute --subnet-id subnet-xxxx --map-public-ip-on-launch` | Turns on auto-assign public IPv4 |

- **`--query Vpc.VpcId --output text`** filters the response down to just the ID, as plain text. Without it, the CLI prints a big block of JSON (AZ, account ID, ARN, settings...), as it did for `create-subnet`.
- **Error `MissingParameters`** means a required option was left out. For `create-subnet`, `--cidr-block` is required, because the CLI can't guess which addresses to use.
- **"Why can't I see the internet gateway?"** Because it was created but **not attached** yet.

**CLI vs console:**

| CLI / CloudShell | Console |
|---|---|
| Fast, repeatable, scriptable | Visual, easy to check what you built |
| What professionals use for complex or repeated tasks | Best for learning |

*I used the console for this project, because CloudShell was on a temporary account verification hold.*

## Troubleshooting CIDR errors

- **Is the format right?** It's four numbers from 0 to 255, separated by three dots, then `/xx`. *(The lesson's example `123.456.78.9` isn't valid, because 456 is more than 255.)*
- **Is the subnet inside the VPC's range?** If the VPC is `10.0.0.0/24` (`10.0.0.0`–`10.0.0.255`), the subnet must start with `10.0.0` as well.
- **Is the subnet too big?** The subnet's slash number must be **equal to or bigger than** the VPC's. For a `/24` VPC, that means `/24` to `/28`. *(The lesson says `/25` to `/32`, but AWS's smallest subnet is `/28`, and a subnet can be the same size as the VPC.)*
- **Does it overlap another subnet** in the same VPC? If so, pick another range.

## Build order and cleanup

1. **VPC** → Your VPCs → Create VPC → **VPC only** → name `NextWork VPC`, IPv4 CIDR `10.0.0.0/16`.
2. **Subnets** → Create subnet → VPC: `NextWork VPC` → name `Public 1`, CIDR `10.0.0.0/24`.
3. Select `Public 1` → Actions → **Edit subnet settings** → tick **Enable auto-assign public IPv4 address**.
4. **Internet gateways** → Create → name `NextWork IG` → Actions → **Attach to VPC** → `NextWork VPC`.
5. *(Extension)* Add a route `0.0.0.0/0 → NextWork IG` to `Public 1`'s route table, and try the same steps in CloudShell.

**Cleanup:** detach and delete the internet gateway → delete the subnet → delete the VPC. Deleting the VPC also removes the subnets and route tables inside it. VPCs, subnets and internet gateways are free, but a clean account is easier to work in. **Don't delete the default VPC.**

---

## Self-quiz

<details><summary>1. Using the city analogy, what are a Region, a VPC, a subnet and an internet gateway?</summary>

The Region is the country, the VPC is your private city, a subnet is a neighbourhood, and the internet gateway is the city gate to the outside world.
</details>

<details><summary>2. Why does every AWS account already have a VPC?</summary>

AWS creates a **default VPC** (with subnets and an internet gateway) in every Region, so you can launch resources from day one.
</details>

<details><summary>3. Name a service that needs a VPC and one that doesn't.</summary>

EC2 (or RDS/Aurora) needs one. S3 or Lambda work without one.
</details>

<details><summary>4. How many bits are in an IPv4 address, and why does each number go up to 255?</summary>

**32 bits**: four numbers of 8 bits each. 8 bits can represent 0 (`00000000`) to 255 (`11111111`).
</details>

<details><summary>5. How many addresses are in a /16, a /24 and a /28?</summary>

65,536 (2¹⁶), 256 (2⁸) and 16 (2⁴). Use 2^(32 − the number after the slash).
</details>

<details><summary>6. Which is bigger, a /8 or a /24?</summary>

**/8**. The smaller the number after the slash, the bigger the block.
</details>

<details><summary>7. How many usable addresses does a /24 subnet have in AWS, and why?</summary>

**251**. AWS reserves 5 addresses in every subnet.
</details>

<details><summary>8. Can two subnets in the same VPC overlap? Can a subnet span two AZs?</summary>

No to both. Subnet ranges must be unique, and each subnet lives in exactly one AZ.
</details>

<details><summary>9. Why spread resources across multiple AZs?</summary>

So that if one AZ has problems, resources in another keep the app running.
</details>

<details><summary>10. In a web app, which part goes in a public subnet and which in a private one?</summary>

The web server goes in the **public** subnet. The database goes in a **private** subnet, because it only needs to talk to the web server.
</details>

<details><summary>11. What actually makes a subnet public?</summary>

Its **route table** sends `0.0.0.0/0` to an **internet gateway** that's attached to the VPC. Instances also need public IPs to be reachable. The subnet's name doesn't matter.
</details>

<details><summary>12. What does "auto-assign public IPv4" do?</summary>

It automatically gives every EC2 instance launched in that subnet a public IP, so it can reach the internet and be reached from it.
</details>

<details><summary>13. You created an internet gateway but it doesn't show on your VPC. Why?</summary>

Creating isn't attaching. You still need to **attach** it to the VPC.
</details>

<details><summary>14. Why do VPC CLI commands start with `aws ec2`?</summary>

VPCs were originally built for EC2 instances, so their commands still live under EC2.
</details>

<details><summary>15. What does `--query Vpc.VpcId --output text` do?</summary>

It filters the CLI's response to show **only the VPC ID**, as plain text, instead of a big block of JSON.
</details>

<details><summary>16. You get `MissingParameters` running `create-subnet`. What's wrong?</summary>

A required option is missing, here `--cidr-block` (or `--vpc-id`).
</details>

<details><summary>17. Your VPC is `10.0.0.0/24`. Is `10.0.1.0/26` a valid subnet? What about `10.0.0.0/30`?</summary>

Neither works. `10.0.1.0/26` is **outside** the VPC's range (`10.0.0.0`–`10.0.0.255`). `/30` is **too small**, because AWS's smallest subnet is `/28`.
</details>

<details><summary>18. What is CloudShell, and what's its advantage?</summary>

A terminal in the AWS console with the AWS CLI pre-installed and already logged in as you, so there's nothing to install or configure.
</details>
