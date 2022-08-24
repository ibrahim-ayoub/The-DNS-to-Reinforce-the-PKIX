# 3. DNSSEC
DNSSEC is used to insure integrity but not privacy. Its main trust anchor is the root name server. First version was in RFC 2065 \cite{rfc2065}. Later obsoleted by RFC 2535 \cite{rfc2535} which in turn was obsoleted with three RFCs 4033, 4034 and 4035 \cite{rfc4033}, \cite{rfc4034}, \cite{rfc4035}.

We define the following terms:
1. **Digital signature:** Digital signatures are done when, for example, a document is hashed and then encrypted with one's private key before being sent along side the original document. The receiver hashes the documents and decrypts the digital signature using the sender's public key. If they match, the receiver can be sure that the document was sent by the authentic sender and that it has not been tampered with on the way.
2. **Fingerprint** hash of a public key
3. **KSK - Key Signing Key:** used to sign or verify a domain's/zone's keys
4. **ZSK - Zone Signing Key:** used to sign or verify a domain's/zone's non-key records 
5. **RRSet - Resource Record Set:** a set of records with same type and same domain/zone
6. **RRSig - Resource Record Signature:** a record containing an RRSet's digital signature 
7. **DS Record - Delegation of Signing:** a record containing the hash of a child domain's/zone's PubKSK (the fingerprint of a child's PubKSK)

The resolver already has a copy of the root public KSK. The basic DNSSEC operation is depicted in *Figure 3*. 

<!--- ---------------------------------------------------------------------------------------------------------------- -->
<p align="center">
  <img src="/images/dnssec-symbols.jpg" />
</p>
<p align = "center">
Figure 3 - DNSSEC Basic Operation
</p>
<!--- ---------------------------------------------------------------------------------------------------------------- -->

1. The client sends a recursive query to the resolver to obtain the address of *example.fr*. Following the regular DNS query process, the resolver sends a query to the root server. 

2. The root server sends back:
    - a non secure referral to the authoritative server of the (net zone)
    - an RRSet DNS Key Records for the root zone (the root zone's pubZSK and PubKSK)
    - An RRSig of the DNS RRset (the above set .2) signed with the root zone's PvtKSK
    - DS Records of the net zone (hash/digest of the netzone's PubKSK) 
    - RRSig of the above DS record (signed using the root zone's PvtZSK)

    \item The recursive server does (3 things):
    1. Verifies the root zone's DNSKey RRset by successfully decrypting the RRset's RRsig using the rootzone's PubKSK
    2. Verifies the root zone's DS records for the net zone by successfully decrypting the record's RRSig using the root zone's PubZSK
    3. Verifies the root zone by comparing its own copy of the root PubKSK with that provided to it by the root DNS server 
    \item The recursive resolver sends a query to the net zone (TLD)
    
    \item The net server (TLD) sends (5 things): 
    1. a non secure referral to the authoritative name server requested (example if asking for example.com)
    2. an RRset of DNSKey records for the net zone (the net zone's PubZSK and PubKSK)
    3. An RRSig of the above record set (signed with the net zone's PvtKSK)
    4. DS record for the example zone (the hash of the example zone PubKSK)
    5. RRSig of the above DS record (signed using the net zone's PvtZSK)
    \item The resolver does (3 things):
    1. verifies the net zone's DNSKey RRset by successfully decrypting the RRset's RRSig using the net zone's PubKSK
    2. verifies the net zone's DS record for the example zone by successfully decrypting the record's RRSig using the net zone's PubZSK 
    3. Verifies the net zone by comparing the hash of the net zone's PubKSK for the net zone with the previously obtained DS record from the root zone for the net zone  
    
    \item The recursive resolver sends a query to the example authoritative name server
    
    \item The example server sends:
    1. RRset of DNSKey records for example zone (example zone PubZSK and PubKSK)
    2. RRSig of above record set (signed using example zone PvtKSK)
    3. RRSet of zone A records from the example zone
    4. RRSig of above record set (signed with example zone PvtZSK)
    
    \item The recursive resolver: 
    1. Verify the example zone DNSKey RRSet by successfully decrypting the RRSet's RRSig using the example PubKSK. 
    2. The example zone A records by successfully decrypting the RRSet's RRSig using the example zone's PubZSK
    3. Verify the zone by comparing the hash values of example zone's PubKSK from the example zone with the previously obtained DS record for the example zone from the net zone
    \item The recursive resolver responds to the original client with the IP address of example.net
\end{itemize}

DNSSEC Signed ROOT by July 1, 2010 making DNSSEC usable 

### &#8592; [2. Domain Name System](DNS.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Main Menu](README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [4. DANE](DANE.md) &#8594;
