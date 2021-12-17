# web security

- PAGE 10, hypertext transfer protocol

- todo
  - move the security related stuff in the old azz webtech file into this file

## basics

### links

- envs
  - [kali linux](https://www.kali.org/)
  - [caine](https://www.caine-live.net/)
  - [blackbox](https://www.backbox.org/)
  - [parrot](https://www.parrotsec.org/)
  - [demon](https://www.demonlinux.com/)
- tools
  - [metasploit](https://www.metasploit.com/)
  - [samurai](https://owasp.org/www-project-samuraiwtf/#SamuraiWTF_Project)
  - [nessus](https://www.tenable.com/products/nessus)
  - [portswigger](https://portswigger.net/burp)
  - [wireshark](https://www.wireshark.org/)
  - [cobalt strike](https://www.cobaltstrike.com/)
  - wmap
  - nmap

### terminology

- exploit: a piece of code that illustrates how to take advantage of a secuirty flaw
- 0 day: type of exploit that has be publicized for less than a day/not publicized at all
- white hat: discovery security holes and will advise owners of the exploits before making them public
- black hat: hoard exploits to maximize the time windows during which they can use vulnerabilities
- dark web: websites available oly via special network nodes that anonymize incoming IP address

- ICANN: internet corporation for assigned names and numbers
  - alotts blocks of IP addresses to regional authorities
- regional authorities
  - grant blocks of addresses to internet service prviders and hosting companies within their region
  - when you connect to the net, your ISP assigned an IP to your computer
    - however the IP is rotated periodically
  - similary: companies that host content are assigned an IP for each server they connect to the network

#### internet protocol suite

- internet protocol suite: dictates how computers exchange data over the web
  - there are over 20 protocols collectively under this umbrella

- internet protocol layers
  - network layer
    - ARP
    - MAC
    - NDP
    - OSPF
    - PPP
  - internet layer
    - IPv4
    - IPv6
  - transport layer
    - TCP
    - UDP
  - application Layer
    - TLS
    - SSL
    - SSL
    - DNS
    - FTP
    - HTTP
    - IMAP
    - POP
    - SMTP
    - SSH
    - XMPP

##### internet layer

- IP: internet protocol addresses
  - destination for data packets
  - unique binary numbers assigned to individual internet-connected computers
  - IPv4: 2x32 addresses
  - IPv6: represented as 8 groups of 4 hexadecimal digits separated by colons

##### Transport Layer Protocols

- TCP: transmission control protocol
  - enables two computers to reliably exchange data over the internet
  - created in response to ARPANET (predecessor to the internet)
  - the first msg sent (was on ARPANET) was a LOGIN command destined for a remote computer at stanford university, but crashed after the first two letters (reason for TCP)

  - high level workflow
    - messages sent via TCP are split into data packets
    - the servers that make up the internet push these packets from sender to receiver without having to read the entire msg
    - the receiver reassembles all the data packets into a usable order according to the sequence number on each packet
      - each packet the receiver gets, it responds with a receipt back to the sender
      - without the receipt, the sender will resend the packet
        - possibly along a different network path
        - possibly at an adjusted speed based on the speed of consumption by receiver
    - this send & receipt workflow guarantees msg delivery

- UDP: User Dataram Protocol
  - newer than TCP
  - commonly used with video/situations where dropped data packets are expected/msg guarantee isnt required, but the data packets can be streamed at a constant rate

##### Application Layer Protocols

- TLS: transport layer security
  - arguable what fkn layer this is actually in
  - method of encryption that provides both privacy and data integry
  - ensures that
    - privacy: packets intercepted by a third party can be decrypted without the appropriate encryption keys
    - data integrity: any attempt to tamper with the packets will be detectable

  - workflow
    - HTTP conversations using TLS are called HTTP secure
    - HTTPS requires the client & server to perform a TLS handshake
    - both parties agree on an encyption method (cypher) and exchange encryption keys
    - any subsequent data packets (request & responses) will be opaque to outsiders

- SMTP: simple mail transport protocol
  - for sending emails
- XMPP: extensible messaging and presence protocol
  - instant messaging
- FTP: file transfer protocol
  - downloading files from servers

- HTTP: hypertext transfer protocol
  - transport webpages and their resources to user agents like web browsers

  - workflow: general
    - user agents generate requests for specific resources
    - web servers expecting those requests, return responses containing either the requested resource, or an error code
    - both requests & responses are plain text msgs, but can be delivered as compressed &/ encrypted
    - the majority of web exploits use http in some fashion

- DNS: domain name system
  - a global directory that translated IP addrs to unique human readable domains e.g. nirv.ai
  - domain registrars: private organizations that register domains before they can be used in DNS

  - workflow
    - when a browser encounters a domain for the first time
      - check the local domain name server (typically hosted by an ISP) to get the associated IP (and various other data) and cache the result

  - terms
    - TTL: time to live: how long a domain name server will cache the IP addr associated with a domain
      - i.e. DNS caching
    - CNAME: canonical name records
      - i.e. aliases for domain names
      - enable multiple domain names to point ot the same IP address
    - MX: mail exchange records
      - help route email

## http focus

- http requests
  - method: akak verb; the action that the user agent wants the server to perform
    - GET: fetch
    - POST: create/update
    - PUT: update/upload
    - PATCH: edit
    - DELETE: delete
    - HEAD: retrieves same info as GET, but instructs the server to return the response without a body
    - CONNECT: initiates two-way comms; e.g. connecting through a proxy
    - OPTIONS: lets a user agent ask what other methods are supported by a resource
    - TRACE: will contain an exact copy of the original HTTP request, for the user agent to see what (if any) alterations were made by intermediate servers

  - URL: universal resource locator: describes the resource being manipulated/fetched

  - Headers: metadata; e.g. type of content the user agent is epcting/whether it accepts compressed responses

  - Body: optional component contains any extra data that needs to be sent to the server

- HTTP responses
  - protocol:
  - code: 3 digit status code
    - 2xx: understood, accepted, and responded to
    - 3xx: redirect
    - 4xx: client error; user agent genreated an invalid request
    - 5xx: server error; request was valid, but the server was unable to fullfil the request

  - msg: status msg
  - headers: instruct the user agent how to treat the content
    - content-type
    - cache-control

  - body: if a resource was requested

- stateful connections:
  - when a client and server perform a handhsake and continue to send packets back n fourth until one of the communicate parties decides to terminate

- http session: the entire conversation (stateless/stateful) between a specific user agent & server
  - server could send a set-cookie header in the initial HTTP response containing data that identifies the user agent
    - the user agent will store & send back the same cookie on each subsequent response

- encryption:
  - method of desguising the contents of messages from prying eyes by encoding them during transmission

## attack vectors

- need to flush out
  - TRACE requests
    - can allow javascript injected into a page to access cookies that have been deliberately made inaccessible to javascript
  - session cookies
    - enables an attacker to impersonate a user agent to a web server
  - man in the middle attacks
    - plain text msgs an be read by anyone intercepting the data packets

### DNS poisoning

- a local DNS cache is deliberately corrupted so that data is routed to a server controlled by an attacker

### cross-site request forgery

- exposure
  - using GET requests for anything other than retrieving resources