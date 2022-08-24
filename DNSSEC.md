# 3. DNSSEC
DNSSEC is used to insure integrity but not privacy. Its main trust anchor is the root name server. It is defined in [[RFC4033]](https://datatracker.ietf.org/doc/rfc4033/), [[RFC4034]](https://datatracker.ietf.org/doc/rfc4034/), and [[RFC4035]](https://datatracker.ietf.org/doc/rfc4035/). Below is a demonstration of how DNSSEC works. 

We first define the following terms:
1. **Digital signature:** Digital signatures are done when, for example, a document is hashed and then encrypted with one's private key before being sent along side the original document. The receiver hashes the documents and decrypts the digital signature using the sender's public key. If they match, the receiver can be sure that the document was sent by the authentic sender and that it has not been tampered with on the way.
2. **Fingerprint:** hash of a public key
3. **KSK - Key Signing Key:** used to sign or verify a zone's key records
4. **ZSK - Zone Signing Key:** used to sign or verify a zone's non-key records 
5. **RRSet - Resource Record Set:** a set of records with same type and same zone
6. **RRSig - Resource Record Signature:** a record containing an RRSet's digital signature 
7. **DS Record - Delegation of Signing:** a record containing the hash of a child zone's PubKSK (the fingerprint of a child's PubKSK)

In addition to providing data integrity and data origin authentication, DNSSEC can explicitly answer in case of record non-existence. In each DNSSEC-enabled zone, there should be two pairs of keys: a private/public Key Signing Key (KSK) pair and a private/public Zone Signing Key (ZSK) pair. KSKs are used to sign keys, while ZSKs are used to sign non-key data. The anchor of trust that DNSSEC depends on is the root zone's public KSK. This key should be known to all resolvers using the DNSSEC chain of trust. he basic DNSSEC operation is depicted in *Figure 3*. 

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
    - a non secure referral to the authoritative server of the .fr zone
    - a DNSKey RRSet for the root zone (the root zone's **PubZSK** and **PubKSK**)
    - An RRSig of the DNS RRset (the above RRSET) signed with the root zone's **PvtKSK**
    - DS Records of the fr zone (hash/digest of the fr zone's **PubKSK**) 
    - RRSig of the above DS record (signed using the root zone's **PvtZSK**)

3. The recursive DNS resolver:
    - Verifies the root zone's DNSKey RRset by successfully decrypting the RRset's RRsig using the rootzone's **PubKSK**
    - Verifies the root zone's DS records for the fr zone by successfully decrypting the record's RRSig using the root zone's **PubZSK**
    - Verifies the root zone by comparing its own copy of the root **PubKSK** with that provided to it by the root DNS server 

4. The recursive DNS resolver sends a query to the .fr zone. 
    
5. The fr server sends back: 
    - a non secure referral to the authoritative name server requested 
    - an RRset of DNSKey records for the fr zone (the fr zone's **PubZSK** and **PubKSK**)
    - An RRSig of the above record set (signed with the fr zone's **PvtKSK**)
    - DS record for the example zone (the hash of the example zone **PubKSK**)
    - RRSig of the above DS record (signed using the fr zone's **PvtZSK**)

6. The recursive DNS resolver:
    - verifies the fr zone's DNSKey RRset by successfully decrypting the RRset's RRSig using the fr zone's **PubKSK**
    - verifies the fr zone's DS record for the example zone by successfully decrypting the record's RRSig using the fr zone's **PubZSK** 
    - Verifies the fr zone by comparing the hash of the fr zone's **PubKSK** for the fr zone with the previously obtained DS record from the root zone for the fr zone  
    
7. The recursive resolver sends a query to the example authoritative name server
    
8. The example server sends:
    - RRset of DNSKey records for example zone (example zone **PubZSK** and **PubKSK**)
    - RRSig of above record set (signed using example zone **PvtKSK**)
    - RRSet of zone A records from the example zone
    - RRSig of above record set (signed with example zone **PvtZSK**)
    
9. The recursive resolver: 
    - Verifies the example zone DNSKey RRSet by successfully decrypting the RRSet's RRSig using the example **PubKSK**. 
    - The example zone A records by successfully decrypting the RRSet's RRSig using the example zone's **PubZSK**
    - Verifies the zone by comparing the hash values of example zone's **PubKSK** from the example zone with the previously obtained DS record for the example zone from the fr zone

10. The recursive resolver responds to the original client with the IP address of example.fr

### &#8592; [2. Domain Name System](DNS.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Main Menu](README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [4. DANE](DANE.md) &#8594;
