### CIDR (Classless Inter-Domain Routing) Notation Overview
CIDR notation is used to define the size of a network (subnet) by specifying an IP address range with a prefix length. The format looks like this:

```
<IP Address>/<Prefix Length>
```

- **IP Address**: The starting address in the network range (e.g., `192.168.1.0`).
- **Prefix Length**: The number of bits used for the network portion of the address, typically ranging from `/8` to `/32` for IPv4.

The remaining bits after the prefix define how many host addresses (IP addresses) the subnet can accommodate. The larger the prefix length (closer to `/32`), the smaller the subnet.

### How to Calculate CIDR Changes
To calculate a CIDR change, you can adjust the prefix length to either increase or decrease the number of available IP addresses. Changing the prefix length directly impacts how many subnets or hosts you can define within a given range.

#### **Key Formula**:
- Number of IP addresses in the subnet = \( 2^{(32 - \text{Prefix Length})} \)

### **Examples of CIDR Calculation**
Let's go through some practical examples of changing the CIDR notation and calculating the number of available IP addresses.

#### 1. **CIDR Range `/24`** (Example: `192.168.1.0/24`)
- **Prefix Length**: `/24` (24 bits are reserved for the network, leaving 8 bits for the host).
- **Calculation**: \( 2^{(32 - 24)} = 2^8 = 256 \)
- **Usable IP Addresses**: 256 addresses, but typically **254** are usable (excluding the network and broadcast addresses).
  
#### 2. **CIDR Range `/28`** (Example: `192.168.1.0/28`)
- **Prefix Length**: `/28` (28 bits are reserved for the network, leaving 4 bits for the host).
- **Calculation**: \( 2^{(32 - 28)} = 2^4 = 16 \)
- **Usable IP Addresses**: 16 addresses, but typically **14** are usable (excluding the network and broadcast addresses).

#### 3. **CIDR Range `/16`** (Example: `192.168.0.0/16`)
- **Prefix Length**: `/16` (16 bits are reserved for the network, leaving 16 bits for the host).
- **Calculation**: \( 2^{(32 - 16)} = 2^{16} = 65,536 \)
- **Usable IP Addresses**: 65,536 addresses, but typically **65,534** are usable.

#### 4. **CIDR Range `/32`** (Example: `192.168.1.10/32`)
- **Prefix Length**: `/32` (all 32 bits are used for the network).
- **Calculation**: \( 2^{(32 - 32)} = 2^0 = 1 \)
- **Usable IP Addresses**: Only 1 IP address, used for a single host.

### **Understanding CIDR Changes**
#### 1. **Expanding the Network**
- If you decrease the prefix length (e.g., from `/24` to `/16`), you increase the number of host addresses:
  - Example: Changing `192.168.1.0/24` to `192.168.0.0/16` expands the available IP address range from **256** to **65,536** addresses.

#### 2. **Narrowing the Network**
- If you increase the prefix length (e.g., from `/16` to `/24`), you reduce the number of available addresses.
  - Example: Changing `192.168.0.0/16` to `192.168.1.0/24` reduces the number of available IP addresses from **65,536** to **256**.

### **CIDR Examples for Different Subnets**

| **CIDR Notation** | **Prefix Length** | **Number of IP Addresses** | **Usable Addresses** |
|-------------------|-------------------|----------------------------|----------------------|
| `192.168.1.0/30`  | 30                | 4                          | 2                    |
| `192.168.1.0/29`  | 29                | 8                          | 6                    |
| `192.168.1.0/28`  | 28                | 16                         | 14                   |
| `192.168.1.0/27`  | 27                | 32                         | 30                   |
| `192.168.1.0/24`  | 24                | 256                        | 254                  |
| `192.168.0.0/16`  | 16                | 65,536                     | 65,534               |

### **Use Cases for Different CIDR Sizes**
1. **/32 (1 IP Address)**:
   - Use for a single device or instance (e.g., when assigning a public IP to a specific EC2 instance).
   
2. **/28 or /29 (14-6 Usable IPs)**:
   - Ideal for small subnets where only a few hosts are needed (e.g., a small VPC subnet for a handful of servers).

3. **/24 (254 Usable IPs)**:
   - Common in corporate networks for creating subnets in a VPC.

4. **/16 (65,534 Usable IPs)**:
   - Use for larger networks, such as when creating a VPC that needs to accommodate thousands of hosts.

---

By understanding CIDR and its impact on IP ranges, you can make better decisions about subnetting, especially when designing VPCs or managing complex networking environments in AWS.
