# 5. DNS-Based Authentication of Named Entities (DANE)



At a glance, if we look at the size of the list of CAs accepted by popular browsers such as Chrome, Firefox, Internet explorer, etc., it varies but is in the range of hundreds. For example, a browser such as Firefox trusts 1,482 CA Certificates (as per EFF SSL observatory in 2010) provided by 651 organizations. In addition, there is a practice of a CA providing authorization to other organizations or its branches to create certificates on its behalf. They are called subordinate CAs. A browser will trust the digital certificate produced by the subordinate CA also. Even if one CA or one of its subordinates is compromised, it can generate a certificate for any domain name, which could then be authenticated by a browser, thereby compromising a secure web communication. For instance, two different CAs (where one is a compromised CA) can issue two separate certificates for the same domain, which the browser will trust. The drawback here is that the domain owner has no way of telling the browser which CA certificate to be used to authenticate to connect to the server of the particular domain.

Different techniques were proposed to reduce the attack probability in the PKIX model such as Trust on First Use (ToFU) [[RFC8672]](https://datatracker.ietf.org/doc/rfc8672/), Certificate Transparency (CT) [[RFC6962]](https://datatracker.ietf.org/doc/rfc6962/), Certificate Authentication and Authorization (CAA) [[RFC6844]](https://datatracker.ietf.org/doc/rfc6844/) and DNS-Based Authentication of Named Entities (DANE) [[RFC6698]](https://datatracker.ietf.org/doc/rfc6698/), [[RFC7218]](https://datatracker.ietf.org/doc/rfc7218/), [[RFC7671]](https://datatracker.ietf.org/doc/rfc7671/). Among the different technologies proposed to limit the attack surface, ToFU is the easiest to implement because it needs only a browser to install the ToFU compatible browser add-on. Perspectives and CT are a system of Notary services which does not completely coexist with the current PKIX model and need additional services acting as notary services. CAA is like a hack which does not need any modifications and, in the short term, looks like a better option for limiting the attack surface. But looking at security from an end-to-end perspective and providing more options to the users (such as self-signed certificates), DANE ranks high. 






The DNS-Based Authentication of Named Entities (DANE) [[RFC6698]](https://datatracker.ietf.org/doc/rfc6698/), [[RFC7218]](https://datatracker.ietf.org/doc/rfc7218/), [[RFC7671]](https://datatracker.ietf.org/doc/rfc7671/) gives control back to zone owners regarding verifying their certificates. DANE's main feature is the TLSA Resource Record (RR) that owners can add to their zone. TLSA RRs form an association between certificates and the domains to which those certificates were given. A typical TLSA RR for a TLS server having the hostname www.example.com, using TCP, and listening on port 443 looks something like *Figure 6*. 

<!--- ---------------------------------------------------------------------------------------------------------------- -->
<p align="center">
  <img src="/images/tlsa.jpg" />
</p>
<p align = "center">
Figure 6 - Example of a TLSA Record
</p>
<!--- ---------------------------------------------------------------------------------------------------------------- -->

The fields of the TLSA RR are [[RFC6698]](https://datatracker.ietf.org/doc/rfc6698/): 
1. Certificate Usage Field (1 byte): could be '0' to specify a CA certificate or the public key of such a certificate limiting the CAs that can issue certificates for domains owning the TLSA RR, '1' to specify an end entity certificate or its public key, and '2' to specify a certificate or its public key. 
2. Selector Field (1 byte): this specifies which part of the certificate received from the server will be matched against the association data. It could be '0' for 'Full Certificate' or '1' for SubjectPublicKeyInfo.
3. Matching Type Field (1 byte): this specifies how the certificate association is presented. It could be '0' for 'Exact match', '1' for SHA-256 hash, or '2' for SHA-512 hash. 
4. Certificate Association Data Field: this specifies the 'certificate association data' to be matched depending on the Selector value. 

Clients intending to connect to a TLS server will receive a certificate, and instead of validating that certificate using a traditional CA, they will perform a DNS query to fetch the TLSA RR from the zone the server to which they are connecting belongs. If the information in the TLSA RR matches the information received from the TLS server, validation is complete, and a secure connection can be initiated. The question may arise: why would zone owners trust DANE since DNS's infrastructure is not designed with security capabilities? Furthermore, why would zone owners move from dealing with CAs to dealing with DNS? In fact, DANE uses [DNS Security Extensions (DNSSEC)](DNSSEC.md) to guarantee the authenticity and integrity of the TLSA RRs.

DNSSEC provides data integrity and data origin authentication. In addition, DNSSEC can explicitly answer in case of record non-existence. In each DNSSEC-enabled zone, there should be two pairs of keys: a private/public Key Signing Key (KSK) pair and a private/public Zone Signing Key (ZSK) pair. KSKs are used to sign keys, while ZSKs are used to sign non-key data. The anchor of trust that DNSSEC depends on is the root zone's public KSK. This key should be known to all resolvers using the DNSSEC chain of trust. More on DNSSEC in [Section 3](DNSSEC.md).


\subsection{Mutual Authentication via DANE}
\label{dance}
In its original form \cite{rfc6698, rfc7671}, DANE is concerned with authenticating TLS servers. TLS clients, on the other hand, were not thought to have certificates that needed verification. This was the main driver behind creating the DANE Authentication for Network Clients Everywhere (DANCE) IETF working group\cite{dance}. The Internet Engineering Task Force (IETF) is a standardization body responsible for issuing Internet Standards as Request for Comments (RFCs). IETF has several working groups, each with a primary theme that it addresses. The DANCE working group aims to extend DANE to enable TLS client authentication using certificates or raw public keys. Like TLS servers, TLS clients will have their raw public keys or certificates and corresponding TLSA RRs in the DNS zone they belong to. This allows mutual authentication via DANE between the TLS clients and servers where a client can check and authenticate the public key or certificate received from the server, which in turn could verify the public key of the certificate received from the client. The DANCE working group have issued two internet drafts so far, \emph{TLS Client Authentication via DANE TLSA RRs} \cite{ietf-dance-client-auth-00} and \emph{TLS Extension for DANE Client Identity} \cite{ietf-dance-tls-clientid-00}. Both drafts are active at the time of writing. The first draft \cite{ietf-dance-client-auth-00} describes how to publish TLSA RRs for TLS clients. Client identities here are assumed to be represented by a DNS domain name. Just as it is when verifying servers, TLS clients that plan on being authenticated via DANE must have a raw public key or a certificate binding them to a public key. The keys or certificates have corresponding TLSA RRs allowing verifying them via DNS. The clients should always signal their intent to be verified with DANE to spare the server from sending unnecessary DNS queries. The second draft \cite{ietf-dance-tls-clientid-00} addresses that as it specifies a TLS extension that allows clients to express their support for DANE and their intent to be verified and allows them to share their DANE identity with the server. 

### &#8592; [4. The Current Public Key Infrastructure with X.509 Digital Certificates](Current-PKIX.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Main Menu](README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [6. Mutual Authentication for IoT Backend Servers](MUTUAL-AUTH.md) &#8594;