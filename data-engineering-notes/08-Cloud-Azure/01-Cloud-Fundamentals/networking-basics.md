# Cloud Networking Basics

## 1. What Is Cloud Networking?

Cloud networking is the process of connecting cloud resources—such as virtual machines, databases, storage accounts, and applications—so they can communicate securely over networks.

In cloud environments, networking controls:

- How resources communicate with each other.
- How applications connect to the internet.
- How users access cloud services.
- How private networks connect to cloud infrastructure.
- How network traffic is secured and routed.

**Example:**

Suppose a company has an Azure Data Factory pipeline that needs to access an Azure SQL Database.

Networking determines whether the pipeline can reach the database, which route the traffic follows, and whether firewalls or network security rules allow the connection.

For a data engineer, networking knowledge is important when troubleshooting pipeline connectivity, accessing private storage, and securing data platforms.

---

## 2. Basic Networking Concepts

Before learning Azure networking, understand these foundational terms.

### 2.1 IP Address

An IP address identifies a device or network interface so that it can communicate over an IP network.

Two main versions are:

- IPv4
- IPv6

Example IPv4 address:

```text
192.168.1.10
```

An IP address can be:

- **Private IP:** Used for communication within private networks.
- **Public IP:** Used for communication over the public internet.

Private IP addresses are not directly routable over the public internet.

Common IPv4 private ranges:

| Range | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

### 2.2 MAC Address

A MAC address identifies a network interface at the data-link layer.

It is commonly used for communication within a local network.

### 2.3 DNS (Domain Name System)

DNS translates domain names into IP addresses.

Example:

```text
www.example.com
```

A DNS lookup resolves the hostname to an IP address so the client can connect to the destination.

**Remember:** DNS resolves names; it does not itself establish the network connection.

### 2.4 Port

A port identifies a communication endpoint associated with a service on a host.

Common ports:

| Port | Protocol / Service |
|---:|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 1433 | Microsoft SQL Server |
| 5432 | PostgreSQL |
| 3306 | MySQL |
| 8080 | Common alternative HTTP application port |

A service must be listening on the relevant port, and network rules must allow the connection.

### 2.5 Protocol

A protocol defines rules for communication between systems.

Common examples:

- TCP
- UDP
- HTTP
- HTTPS
- DNS
- SSH

---

## 3. TCP vs UDP

TCP and UDP are transport-layer protocols.

### TCP (Transmission Control Protocol)

TCP provides reliable, ordered delivery of data between applications.

Characteristics:

- Connection-oriented.
- Uses acknowledgments.
- Retransmits lost data.
- Maintains ordering.
- Includes flow and congestion control.

Examples:
- HTTPS traffic.
- SSH connections.
- Many database connections.

### UDP (User Datagram Protocol)

UDP sends datagrams without establishing a TCP-style connection.

Characteristics:

- Connectionless.
- No built-in delivery acknowledgment.
- No built-in retransmission.
- Lower protocol overhead.
- Delivery and ordering are not guaranteed by UDP itself.

Examples:
- DNS queries in common configurations.
- Real-time voice and video.
- Some streaming and gaming traffic.

| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable ordered byte stream | Datagram delivery without built-in reliability |
| Acknowledgments and retransmissions | No built-in acknowledgments or retransmissions |
| More protocol overhead | Less protocol overhead |
| Common for database connections | Common for real-time communication |

**Interview point:** TCP is not automatically better than UDP. The correct choice depends on application requirements.

---

## 4. HTTP vs HTTPS

HTTP is an application-layer protocol used for communication between clients and web servers.

HTTPS is HTTP secured using TLS.

| HTTP | HTTPS |
|---|---|
| Data is not encrypted by HTTP itself | TLS protects communication |
| Common default port 80 | Common default port 443 |
| Does not provide TLS-based server authentication | Uses certificates and TLS authentication |
| Not suitable for sensitive data without another security layer | Commonly used for secure web and API traffic |

HTTPS provides confidentiality and integrity for the protected connection and supports server authentication through certificates.

It does not guarantee that the website or application itself is trustworthy.

---

## 5. Subnet and CIDR

A subnet is a smaller logical network created from a larger IP address range.

Subnetting helps organize networks and control how resources communicate.

### What Is CIDR?

CIDR stands for Classless Inter-Domain Routing.

It represents an IP network using an address and prefix length.

Example:

```text
10.0.0.0/16
```

The `/16` means the first 16 bits represent the network prefix.

Another example:

```text
10.0.1.0/24
```

A `/24` IPv4 subnet contains 256 total addresses, including network and broadcast addresses.

In Azure, some addresses in every subnet are reserved by the platform, so the usable address count is lower than the total.

### Example Network

```text
Virtual Network: 10.0.0.0/16
        |
        +-- Subnet: 10.0.1.0/24
        |
        +-- Subnet: 10.0.2.0/24
        |
        +-- Subnet: 10.0.3.0/24
```

Possible organization:

- Subnet 1: Application resources.
- Subnet 2: Database-related resources.
- Subnet 3: Private endpoints or other services, where supported.

Subnets must use non-overlapping address ranges within the virtual network.

---

## 6. What Is a Virtual Network (VNet)?

An Azure Virtual Network (VNet) is a logically isolated network in Azure.

It allows Azure resources to communicate securely with one another, the internet, and on-premises networks through supported connectivity options.

A VNet provides:

- Private IP address ranges.
- Subnets.
- Routing.
- Network security controls.
- Connectivity options.
- Integration with supported Azure services.

### Example

Suppose a company has the following resources:

- Azure Virtual Machine.
- Azure SQL Database.
- Application server.
- Private endpoint for storage.

These resources can use a VNet and related networking services to establish private connectivity where supported.

### VNet Architecture

```text
Azure Virtual Network: 10.0.0.0/16
             |
     +-------+--------+
     |       |        |
     v       v        v
  Subnet A Subnet B Subnet C
     |       |        |
     v       v        v
   VM     App Tier  Private
                    Endpoint
```

A VNet is a logical network boundary. It does not automatically make every resource private or secure. Correct routing, DNS, firewall, and access configuration are still required.

---

## 7. Public IP vs Private IP

### Public IP

A public IP address is used for communication over the public internet.

Example:

A web application may expose a public endpoint so customers can access it.

### Private IP

A private IP address is used for communication within private network environments.

Example:

An application server communicates with a database using a private IP address.

| Public IP | Private IP |
|---|---|
| Used for internet-facing connectivity | Used for private network communication |
| Must be globally unique within its scope | Can be reused in separate private networks |
| May expose a service to internet traffic | Not directly routable over the public internet |
| Requires appropriate security controls | Also requires appropriate security controls |

Having a public IP does not mean every port is open. Firewalls and network security rules still control traffic.

---

## 8. Network Security Group (NSG)

A Network Security Group (NSG) filters inbound and outbound network traffic using security rules.

Rules can include:

- Source IP or source range.
- Destination IP or destination range.
- Source port.
- Destination port.
- Protocol.
- Direction.
- Allow or deny action.
- Priority.

### Example

Suppose a virtual machine hosts an application.

You may create rules such as:

| Direction | Port | Protocol | Action |
|---|---:|---|---|
| Inbound | 443 | TCP | Allow |
| Inbound | 22 | TCP | Allow only from an approved admin IP range |
| Inbound | Any | Any | Deny, subject to rule evaluation |

An NSG supports inbound and outbound filtering.

### Important points

- NSGs are stateful.
- Rules are evaluated by priority, with lower priority numbers evaluated first.
- A matching rule determines the action.
- Azure also provides default NSG rules.
- An NSG can be associated with a subnet or a network interface, subject to platform behavior.

**Interview point:** An NSG controls network traffic. It does not replace identity-based authorization such as Azure RBAC.

---

## 9. Firewall

A firewall monitors and controls network traffic based on configured security rules.

Azure provides services such as Azure Firewall and other network security capabilities.

### NSG vs Firewall

| NSG | Azure Firewall |
|---|---|
| Network traffic filtering at subnet or NIC level | Centralized managed network firewall |
| Uses network security rules | Supports network and application rules, plus other capabilities |
| Commonly used for subnet or VM traffic control | Commonly used for centralized network traffic inspection and control |
| Does not provide full firewall platform capabilities | Provides advanced firewall capabilities |

Use the appropriate control based on the network architecture and security requirements.

---

## 10. Routing and Route Tables

Routing determines where network packets should go.

A route table contains routes that determine the next hop for network traffic.

Common next-hop types in Azure include:

- Virtual network.
- Internet.
- Virtual network gateway.
- Virtual appliance.
- None (drop traffic).

### Example

Suppose a subnet sends traffic to an on-premises network.

A user-defined route can direct traffic to a supported network virtual appliance or gateway, depending on the design.

### Why is routing important?

Routing helps control:

- Communication between subnets.
- Internet connectivity.
- On-premises connectivity.
- Network inspection.
- Traffic flow through virtual appliances.

**Remember:** NSGs filter traffic; route tables determine where traffic is directed.

---

## 11. VPN and ExpressRoute

Organizations often need to connect their on-premises networks to Azure.

Two common connectivity options are VPN and ExpressRoute.

### VPN (Virtual Private Network)

A VPN creates an encrypted connection over an existing network, commonly the public internet.

Azure VPN Gateway can support site-to-site and point-to-site connectivity.

Examples:

- Site-to-site: Connect an office network to an Azure VNet.
- Point-to-site: Connect an individual user's device to an Azure VNet.

### ExpressRoute

ExpressRoute provides private connectivity between an organization's network and Microsoft cloud services through a connectivity provider.

It does not send traffic over the public internet in the same way as a typical internet-based VPN.

### VPN vs ExpressRoute

| VPN | ExpressRoute |
|---|---|
| Commonly uses encrypted tunnels over the internet | Uses private connectivity through a provider |
| Often quicker and less expensive to establish | Often involves more planning and provider coordination |
| Internet path can affect performance | Offers more predictable connectivity characteristics |
| Suitable for many remote and site-to-site needs | Suitable for enterprise connectivity requirements |

ExpressRoute is not automatically encrypted at the application level. Additional encryption may be required depending on security requirements.

---

## 12. Private Endpoint

A private endpoint provides a private IP address from a VNet for accessing a supported Azure service through Azure Private Link.

It allows clients to connect to the service through a private network path.

### Example

An organization stores sensitive data in ADLS Gen2.

Instead of relying on a publicly reachable storage endpoint, the organization can configure a private endpoint for the storage service.

The private endpoint receives an IP address from the selected subnet.

### Architecture

```text
Azure Virtual Network
        |
        v
Application / Databricks
        |
        v
Private Endpoint
        |
        v
Azure Private Link
        |
        v
Storage Account
```

### Benefits

- Private connectivity to supported services.
- Reduced exposure to public network access.
- Better control over network access.
- Integration with private DNS.

**Important:** A private endpoint does not automatically grant data access. Identity permissions, service authorization, DNS, and network configuration must also be correct.

---

## 13. Service Endpoints vs Private Endpoints

These two concepts are often confused.

### Service Endpoint

A VNet service endpoint extends the VNet identity to a supported Azure service over the Azure backbone.

The service continues to use its service endpoint address rather than receiving a private IP from the subnet.

Access can be restricted through the service's virtual network rules.

### Private Endpoint

A private endpoint creates a network interface with a private IP address in the VNet for the supported service.

### Comparison

| Service Endpoint | Private Endpoint |
|---|---|
| Extends VNet identity to supported service | Provides a private IP in the VNet |
| Service endpoint address remains in use | Client connects to private endpoint IP |
| Uses service firewall/VNet rules | Uses Azure Private Link |
| Does not create a private endpoint NIC | Creates a private endpoint network interface |
| Does not provide the same private IP connectivity model | Supports private access to the service |

A private endpoint commonly requires appropriate private DNS configuration so the service hostname resolves to the private IP from the intended network.

---

## 14. Load Balancer

A load balancer distributes incoming network traffic across multiple backend resources.

### Example

Suppose an application runs on three virtual machines.

Instead of sending all requests to one machine, a load balancer distributes traffic across the available backend instances according to its configuration.

### Benefits

- Distributes traffic.
- Improves application availability.
- Helps scale applications horizontally.
- Supports health probes and backend health evaluation.

Azure Load Balancer operates at Layer 4 (TCP/UDP).

For HTTP/HTTPS application-layer routing, services such as Azure Application Gateway or Azure Front Door may be more appropriate, depending on the architecture.

---

## 15. DNS in Azure

DNS resolves domain names to IP addresses.

In Azure, DNS is important for:

- Resolving public application endpoints.
- Resolving private endpoints.
- Connecting applications to databases.
- Supporting hybrid network connectivity.

### Private DNS Zone

Azure Private DNS provides DNS resolution within linked virtual networks.

Example:

A storage account has a private endpoint.

The application must resolve the storage hostname to the private endpoint's private IP address.

A correctly configured private DNS zone and VNet link can support this resolution.

**Important:** A private endpoint can exist and still fail to work as expected if DNS is not configured correctly.

---

## 16. Network Troubleshooting for Data Engineers

A data pipeline may fail even when its code is correct.

Possible causes include:

- Incorrect hostname.
- DNS resolution failure.
- Wrong port.
- NSG rule blocking traffic.
- Firewall restriction.
- Missing route.
- Private endpoint configuration issue.
- Incorrect private DNS setup.
- Target service not listening or not available.
- Authentication or authorization failure.

### Basic troubleshooting sequence

1. Confirm the hostname and endpoint.
2. Check DNS resolution.
3. Verify the destination IP address.
4. Confirm the required port and protocol.
5. Check NSG and firewall rules.
6. Review route tables and network connectivity.
7. Verify private endpoint and DNS configuration.
8. Check service health and listener configuration.
9. Check identity authentication and authorization separately.

### Example

A Databricks workload cannot connect to an Azure SQL Database.

Investigate:

- Is the database hostname correct?
- Does DNS resolve to the expected address?
- Is the required TCP port reachable?
- Is the workspace network configured correctly?
- Are firewall rules allowing the connection?
- Is private endpoint DNS configured?
- Does the identity have database permissions?

Do not assume every connection failure is a networking problem. Authentication and authorization failures can produce similar symptoms.

---

## 17. Networking in Azure Data Engineering

Networking becomes important when data services are deployed in restricted or private environments.

### Example Architecture

```text
On-Premises Data Center
          |
          | VPN / ExpressRoute
          |
          v
Azure Virtual Network
          |
    +-----+----------------+
    |                      |
    v                      v
Azure Databricks       Private Endpoint
                           |
                           v
                     ADLS Gen2
                           |
                           v
                    Data Storage
```

A possible secure data engineering setup includes:

1. Azure Databricks or another processing service.
2. ADLS Gen2 for storage.
3. Private endpoints for supported storage access.
4. Private DNS for hostname resolution.
5. NSGs and firewall rules for traffic control.
6. Managed identity or another supported identity mechanism for authentication.
7. Azure RBAC, ACLs, or service-specific permissions for authorization.

**Key distinction:**

- Networking determines whether a connection can reach the service.
- Authentication verifies the identity.
- Authorization determines what the identity can do.

All three must work for a successful data operation.

---

## 18. Common Interview Questions

### Q1. What is cloud networking?

Cloud networking connects cloud resources and controls how they communicate securely.

### Q2. What is an IP address?

An IP address identifies a device or network interface for communication over an IP network.

### Q3. What is the difference between public and private IP addresses?

Public IP addresses support internet-facing communication. Private IP addresses are used within private network environments.

### Q4. What is DNS?

DNS translates domain names into IP addresses.

### Q5. What is a port?

A port identifies a communication endpoint for an application or service on a host.

### Q6. What is the difference between TCP and UDP?

TCP provides a reliable, ordered byte stream. UDP sends datagrams without built-in delivery or ordering guarantees.

### Q7. What is a subnet?

A subnet is a smaller logical network created from a larger IP address range.

### Q8. What is a CIDR notation?

CIDR represents an IP network using an address and prefix length, such as `10.0.0.0/16`.

### Q9. What is an Azure VNet?

An Azure VNet is a logically isolated network that allows Azure resources to communicate with one another and with other networks.

### Q10. What is an NSG?

A Network Security Group filters inbound and outbound network traffic using security rules.

### Q11. What is the difference between an NSG and a firewall?

An NSG provides network traffic filtering at subnet or network interface scope. Azure Firewall is a centralized managed firewall service with broader network security capabilities.

### Q12. What is a private endpoint?

A private endpoint provides a private IP address in a VNet to access a supported Azure service through Private Link.

### Q13. What is the difference between a service endpoint and a private endpoint?

A service endpoint extends VNet identity to a supported service. A private endpoint provides a private IP address in the VNet for that service.

### Q14. What is a VPN?

A VPN creates an encrypted network connection over an existing network, commonly the internet.

### Q15. What is ExpressRoute?

ExpressRoute provides private connectivity between an organization's network and Microsoft cloud services through a connectivity provider.

### Q16. What is a route table?

A route table contains routes that determine the next hop for network traffic.

### Q17. What is a load balancer?

A load balancer distributes incoming traffic across backend resources.

### Q18. Why is DNS important for private endpoints?

DNS must resolve the service hostname to the private endpoint IP address from the intended network.

### Q19. Does a private endpoint automatically grant access to a storage account?

No. It provides private network connectivity. Identity permissions and storage authorization are still required.

### Q20. What would you check if a Databricks job cannot connect to ADLS Gen2?

Check the endpoint, DNS resolution, private endpoint configuration, network rules, routing, authentication, and storage permissions.

---

## 19. Common Mistakes to Avoid

- Confusing DNS resolution with network connectivity.
- Confusing an IP address with a port.
- Assuming a private IP automatically means secure access.
- Assuming an NSG replaces identity-based authorization.
- Confusing NSGs with Azure Firewall.
- Confusing service endpoints with private endpoints.
- Assuming private endpoints automatically configure DNS.
- Assuming private endpoints grant storage permissions.
- Treating VPN and ExpressRoute as identical.
- Assuming every connection failure is caused by networking.
- Forgetting that routing and firewall rules are separate concerns.
- Assuming a load balancer provides application-layer routing in every configuration.

---

## 20. Quick Revision

| Concept | Key point |
|---|---|
| IP address | Identifies a network interface |
| Public IP | Supports internet-facing communication |
| Private IP | Used within private networks |
| DNS | Resolves names to IP addresses |
| Port | Identifies an application communication endpoint |
| TCP | Reliable, ordered byte stream |
| UDP | Datagram protocol without built-in delivery guarantees |
| Subnet | Smaller logical network |
| CIDR | Network address and prefix notation |
| VNet | Azure logically isolated network |
| NSG | Filters inbound and outbound network traffic |
| Firewall | Controls and inspects network traffic |
| Route table | Determines traffic next hop |
| VPN | Encrypted network tunnel |
| ExpressRoute | Private cloud connectivity through a provider |
| Private endpoint | Private IP connectivity through Private Link |
| Service endpoint | Extends VNet identity to a supported service |
| Load balancer | Distributes traffic across backend resources |
| Private DNS | Resolves private service names within linked networks |

---

## 21. Self-Assessment

Try answering these questions without looking at the notes:

1. What is cloud networking?
2. Explain public and private IP addresses.
3. What is DNS and why is it important?
4. What is the difference between TCP and UDP?
5. What is a subnet and what does `/24` mean?
6. What is an Azure VNet?
7. What is an NSG?
8. What is the difference between an NSG and a firewall?
9. What is the difference between a route table and an NSG?
10. Explain VPN and ExpressRoute.
11. What is a private endpoint?
12. Explain service endpoint vs private endpoint.
13. Why is private DNS important?
14. How would you troubleshoot an Azure Databricks connection failure to ADLS Gen2?
15. What is the difference between network connectivity, authentication, and authorization?

**Interview readiness:** You should be able to explain IP addressing, DNS, ports, TCP/UDP, VNets, subnets, NSGs, private endpoints, and how networking affects secure data pipeline connectivity.