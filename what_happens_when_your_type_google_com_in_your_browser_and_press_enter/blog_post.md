# What Happens When You Type https://www.google.com in Your Browser and Press Enter?

It is one of the most popular interview questions for software engineers, and for a good reason. It touches upon many layers of the internet stack, testing a candidate's understanding of how the web works underneath the hood. When you type `https://www.google.com` in your browser and press Enter, a complex yet incredibly fast series of events takes place to fetch the web page and display it on your screen.

Let's break down this process step-by-step.

## Architecture Flow Diagram

Below is a diagram illustrating the entire flow of the request:

```mermaid
graph TD
    Client["Browser/Client"] -->|1. Request IP for google.com| DNS["DNS Resolver"]
    DNS -->|2. Returns IP: 142.250.190.4| Client
    Client -->|3. HTTPS Request over TCP Port 443| FW["Firewall"]
    FW -->|4. Encrypted Traffic Allowed| LB["Load Balancer"]
    LB -->|5. Distributes Traffic| WS1["Web Server 1"]
    LB -->|5. Distributes Traffic| WS2["Web Server 2"]
    WS1 -->|6. Requests Dynamic Content| AS["Application Server"]
    AS -->|7. Queries Data| DB[("Database")]
    DB -->|8. Returns Data| AS
    AS -->|9. Generates HTML| WS1
    WS1 -->|10. Serves Web Page (Encrypted)| Client
```

## 1. DNS Request
Before your browser can connect to Google's servers, it needs to know the server's IP address. Humans use domain names (like `www.google.com`), but computers communicate using IP addresses (like `142.250.190.4`).
- Your browser first checks its own cache, the operating system's cache, the router's cache, and the ISP's cache to see if it already knows the IP address for `www.google.com`.
- If not found, a **DNS (Domain Name System) request** is initiated. The OS queries a DNS resolver, which contacts the Root Server, the TLD (Top-Level Domain) Server (like `.com`), and finally the Authoritative Name Server for `google.com`. This server returns the IP address of `www.google.com` back to your browser.

## 2. TCP/IP
Once the browser has the IP address, it needs to establish a connection to the server. The internet uses the **TCP/IP** (Transmission Control Protocol / Internet Protocol) suite to route data packets between your computer and the server.
- The browser opens a TCP connection using the **TCP 3-way handshake** (SYN, SYN-ACK, ACK) to port 443 (the default port for HTTPS traffic). This ensures a reliable connection is established before sending any data.

## 3. Firewall
Before your packets reach the web server, or even when leaving your computer, they must pass through firewalls. A **firewall** is a network security system that monitors and controls incoming and outgoing network traffic based on predetermined security rules.
- Your computer's local firewall ensures the outgoing request is permitted.
- On Google's side, enterprise firewalls act as a barrier to protect their internal network from malicious traffic (like DDoS attacks), allowing only legitimate HTTPS requests on port 443 through.

## 4. HTTPS/SSL
Since you requested `https://` and not `http://`, the communication must be secure. This is where **HTTPS/SSL** (or TLS, the modern equivalent of SSL) comes into play.
- After the TCP connection is established, the client and server undergo an **SSL/TLS handshake**.
- The server sends its SSL certificate (containing its public key), which the browser verifies with a trusted Certificate Authority (CA).
- They then agree on a symmetric session key to encrypt all subsequent communication. This ensures that anyone intercepting the traffic cannot read the sensitive data being transmitted.

## 5. Load-Balancer
Google handles billions of searches per day; a single server could never handle all that traffic. Instead, the request reaches a **load-balancer** (often a hardware device or specialized software like HAProxy).
- The load-balancer distributes incoming network traffic across a cluster of servers.
- It uses algorithms (like Round Robin, Least Connections, or IP Hash) to decide which server should handle your specific request, ensuring high availability, reliability, and preventing any single server from becoming overwhelmed.

## 6. Web Server
Once the load-balancer selects a server, it forwards your HTTP(S) request to a **web server** (such as Nginx or Apache).
- The web server is responsible for serving static content (HTML, CSS, JavaScript, images).
- For a simple static page, the web server would retrieve the file from disk and send it back. But Google's search page is dynamic, so the web server passes the request to an application server.

## 7. Application Server
The **application server** handles the core business logic.
- It interprets the request (e.g., loading the specific localized Google homepage based on your location and account status).
- It runs backend code (often in languages like Python, Java, or C++) to dynamically generate the HTML that makes up the search page. If it needs data (like your user profile or search history), it talks to the database.

## 8. Database
The **database** is where persistent data is stored (like user accounts, search index data, etc.).
- The application server queries the database (using SQL or NoSQL databases depending on the architecture).
- The database returns the requested data to the application server, which then incorporates it into the final HTML page.

## The Response
Finally, the application server sends the generated HTML back to the web server, which sends it back through the load-balancer, over the internet, and into your browser. Your browser then parses the HTML, CSS, and JavaScript, rendering the familiar Google homepage on your screen.

All of this happens in a fraction of a second!
