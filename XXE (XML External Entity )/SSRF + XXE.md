**Server-Side Request Forgery (SSRF)** attacks occur when an attacker abuses functionality on a server, causing the server to make requests to an unintended location. In the context of XXE, an attacker can manipulate XML input to make the server issue requests to internal services or access internal files. This technique can be used to scan internal networks, access restricted endpoints, or interact with services that are only accessible from the server’s local network.

### Internal Network Scanning

Consider a scenario where a vulnerable server hosts another web application internally on a non-standard port. An attacker can exploit an XXE vulnerability that makes the server send a request to its own internal network resource.

For example, using the captured request from the in-band XXE task, send the captured request to Burp Intruder and use the payload below:

```xml
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "http://localhost:§10§/" >
]>
<contact>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <message>test</message>
</contact>
```

The external entity is set to fetch data from `http://localhost:§10§/`. Intruder will then reiterate the request and search for an internal service running on the server.

**Steps to brute force for open ports:**

1. Once the captured request from the In-Band XXE is in Intruder, click the Add `§` button while highlighting the port.

![HTTP request in intruder](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/593a4fc9649f7eb2fec803104b3a897b.png)  

2. In the Payloads tab, set the payload type to Numbers with the Payload settings from 1 to 65535.

![Payloads tab with the right settings](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/e5358d3d6bde1dda471a34d9e207598e.png)

1. Once done, click the Start attack button and click the Length column to sort which item has the largest size. The difference in the server's response size is worth further investigation since it might contain information that is different compared to the other intruder requests.

![Successful attack showing the data](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/6f184ed4700daf4859c6d4314a0205f5.png)  

![Flag in the response](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/634833be32484b2bfeb5761d3de57611.png)  

**How the Server Processes This**:

The entity `&xxe;` is referenced within the `<name>` tag, triggering the server to make an HTTP request to the specified URL when the XML is parsed. The response of the requested resource will then be included in the server response. If an application contains secret keys, API keys, or hardcoded passwords, this information can then be used in another form of attack, such as password reuse.

### Potential Security Implications

- **Reconnaissance**: Attackers can discover services running on internal network ports and gain insights into the server's internal architecture.
- **Data Leakage**: If the internal service returns sensitive information, it could be exposed externally through errors or XML data output.
- **Elevation of Privilege**: Accessing internal services could lead to further exploits, potentially escalating an attacker's capabilities within the network.