- [Standard ACL](#standard-acl)
- [Extended ACL](#extended-acl)
  - [Extended ACL Syntax and Order](#extended-acl-syntax-and-order)
  - [Example of a Complete Extended ACL Entry](#example-of-a-complete-extended-acl-entry)

# Standard ACL

-   ACL entries have specific syntax, which generally includes:
    -   **Access-list number or name**: Identifies the ACL. Standard ACLs use numbers 1-99 and 1300-1999; Extended ACLs use numbers 100-199 and 2000-2699.
    -   **Permit or deny**: Defines the action if a match is found.
    -   **Protocol**: Defines the protocol to match, e.g., `ip`, `tcp`, `udp`.
    -   **Source and destination IP addresses**: Can be specific IPs or wildcard masks (e.g., `192.168.1.0 0.0.0.255`).
    -   **Ports** (for TCP/UDP): Specifies port numbers for services (e.g., `eq 80` for HTTP).

* IP and Port order is flexible
  * `20 permit tcp any eq 1719 any`
    -   **`any` (source IP)**: Allows traffic from any source IP address, meaning no specific source IP is required.      
    -   **`eq 1719` (source port)**: Matches traffic with a source TCP port of 1719. Port 1719 is commonly used by H.323 Gatekeeper for multimedia and VoIP signaling in some network configurations.
    -   **`any` (destination IP)**: Allows traffic to any destination IP address, meaning no specific destination IP is required.

# Extended ACL

In Cisco Extended ACLs, the elements must be in a specific order to be interpreted correctly by the router. Here’s the proper sequence for each element in an extended ACL entry:

## Extended ACL Syntax and Order

`access-list <ACL number or name> <permit | deny> <protocol> <source IP> <source wildcard> <destination IP> <destination wildcard> [operator <port>]`

Each element in the syntax has a specific purpose, explained below:

1.  **`access-list`**: Keyword to start the ACL definition.
2.  **ACL number or name**: Identifies the ACL. Extended ACLs can be numbered (100–199 and 2000–2699) or named.
3.  **`permit | deny`**: Specifies the action (allow or block) if a packet matches this ACL entry.
4.  **`protocol`**: Defines the protocol to match. Common protocols include:
    -   `ip`: Any IP protocol.
    -   `tcp`, `udp`: For TCP/UDP traffic.
    -   `icmp`: For ping and other ICMP messages.
5.  **Source IP address**: The IP address of the traffic’s source. Use `any` to allow traffic from any source, or `host` to specify a single IP.
6.  **Source wildcard mask**: Defines which parts of the source IP should be matched. For example:
    -   `0.0.0.0` with `host` means an exact IP.
    -   `0.0.0.255` matches any IP in the last octet.
    > LZ: 0 means exact, 255 means any
1.  **Destination IP address**: The destination IP for the traffic. Use `any` for any destination or `host` for a specific IP.
2.  **Destination wildcard mask**: Specifies which parts of the destination IP to match, similar to the source wildcard mask.
3.  **Operator** (optional): Specifies a port or range for `tcp` or `udp`. Common operators include:
    -   `eq` (equal to a specific port),
    -   `lt` (less than a port number),
    -   `gt` (greater than a port number),
    -   `range` (between two ports).
4.   **Port number** (optional): The port number or range of ports to match if the protocol is `tcp` or `udp`.

## Example of a Complete Extended ACL Entry

`access-list 110 permit tcp 192.168.1.0 0.0.0.255 10.10.10.0 0.0.0.255 eq 80`

**Explanation**:

-   **ACL number**: `110`
-   **Action**: `permit`
-   **Protocol**: `tcp`
-   **Source IP range**: `192.168.1.0` with wildcard `0.0.0.255`
-   **Destination IP range**: `10.10.10.0` with wildcard `0.0.0.255`
-   **Port**: `eq 80` (HTTP)

This ACL entry permits TCP traffic from the `192.168.1.0/24` subnet to reach the `10.10.10.0/24` subnet on port 80. Following this order is essential for extended ACLs to function as expected.