# 4. The Current Public Key Infrastructure with X.509 Digital Certificates

A client that wants to connect to a web server obtains the server's IP address via a DNS resolution as explained in [Section 2](DNS.md) and establishes a Hyper Text Transfer Protocol (HTTP) session with the server. The integrity of the DNS resolution is guaratneed by using [DNSSEC](DNSSEC.md) which allows the client to be sure that the DNS responses come from the legitimate name server, and that they have not been tampered with along the way. A TLS client connecting to a TLS server will receive that server's X.509 certificate. The elements of the certificate include information about the certificate issuer, information about the recipient of the certificate and the certificate's digital signature. The client ensures that the server is whom it claims to be by verifying that the certificate the server sent is legitimate. 

The Transport Layer Security (TLS) protocol is used to secure the session between the client and the server. TLS allows the client and the server to authenticate each other and negotiate an encryption algorithm and cryptographic keys before the data is exchanged. TLS ensures that data cannot be tampered with during transit since the data is encrypted. The TLS protocol is one of the main elements of security in today's Internet that relies on the Public Key Infrastructure (PKI). For the most part, secure communications over the Internet start with a TLS handshake. See *Figures 4* and *5.*

<p align="center">
  <img src="/images/tls13-handshake.jpg" />
</p>
<p align = "center">
Figure 4 - TLS Handshake
</p>

<p align="center">
  <img src="/images/tls-session-setup.jpg" />
</p>
<p align = "center">
Figure 5 - TLS Session Setup
</p>

During the TLS handshake, data is exchanged securely between the client and server using asymmetric cryptography. Asymmetric cryptography is based on using a pair of public/private keys. Data encrypted using a public key can only be decrypted using the public key's corresponding private key. A mathematical relation between the public and private keys ensures that the private key can not be inferred from the public key. A website (e.g., that of a bank) publishes its public key for anyone to download. An account holder in the bank, Alice, encrypts a message using the public key and sends it to the bank. Only the bank can decrypt the message using its private key. Thus, Alice is sure that her message is accessed only by the bank and not by anyone else. 

However, simply using a public key whose owner claims to be the bank is hazardous. With the current model that only relies on public/private key pairs, the risk is high that an impersonator publishes their public key posing as Alice's bank. Alice, trusting that she is contacting her bank, will encrypt the message using the public key and send it to the impersonator. The impersonator will be able to decrypt the messages sent by Alice, and will therefore have access to Alice's banking details and any other information she shares. It is clear that simply relying on public/private key pairs is not sufficient as anyone can generate such key pairs and can impersonate other entities. Hence, binding between the identity (e.g., the domain name) and the public key is necessary. The [X.509 standard](https://www.itu.int/rec/T-REC-X.509) proposed by the ITU and ISO provides a mechanism to bind a public key to a specific identity. The domain holder may do this binding on her own, and in that case, it is called a self-signed certificate. If the self-signed certificate is obtained from a trusted source by the application using the certificate for authentication, then it is accepted. Otherwise there is no guarantee of the certificate's authenticity.

A chain of trust exists to help TLS clients verify the authenticity of certificates. This chain of trust starts at the top with root CAs. Root CAs are trusted by default and have self-signed certificates that allow them to issue other certificates creating intermediary CAs. Intermediary CAs, which the root CAs trust, can issue certificates for servers and websites. An X.509 certificate contains several fields describing the issuing CA and its owner. The issuing CA signs the certificate by encrypting the hash (digest) of the certificate fields with its private key. The problem with the current model is that certificates are not required to be verified by the CA that issued them. Moreover, a CA that wants its certificates to be trusted must be added to the root store containing the list of trusted root CA certificates. Different vendors and browsers have different root stores. Moreover, the root and intermediary CAs are not immune to [security breaches](https://www.enisa.europa.eu/media/news-items/operation-black-tulip/), and could, in theory, issue certificates for any domain like the root authority Diginotar did when it issued a [fraudulent Google certificate](https://security.googleblog.com/2011/08/update-on-attempted-man-in-middle.htm) in 2011. 








\subsubsection{The Certification Authorities (CAs) role}
This is where the need for a \textbf{\textit{trusted third party}} arises. It is just like the passport wherein it can be issued only by a trusted third party delegated by a country's government. The trusted authority attests that the person in the photo is identified by a particular name, surname and other credentials.

In web browsing, a \textbf{\textit{digital certificate}} issued for a domain name, is like the passport.  In the PKIX ecosystem, the role of the trusted authority is played by organizations called CAs. A certificate issued by a given CA binds the given domain name with information such as the certificate assignee, the entity which has requested the certificate, its validity period etc. The CA is used to assert to the browser that the web server represents the domain asked by the client. A TLS connection is established between the browser and the web server on successful authentication of the certificate. With the TLS connection established, the traffic between the browser and the web server is encrypted and is protected against any third-party eavesdropping.

Similar to how a passport attested by one country's trusted authority is accepted by other countries as a validated document for authenticating a person, browser vendors (such as Firefox, Chrome, Internet explorer, Safari etc.,) accept digital certificates created only by certain CAs. The browser vendors authorize an organization to be a CA only after understanding that they are trustworthy and they follow strict principles and procedures to provide certificates only for correct domain holders. Once the browser vendors authorize an organization to be a CA, the latter's digital certificate is added to the list of trusted CAs in the browser library. Thus, once a client using a browser accesses a domain name which has a digital certificate generated by one of the CAs among its pre-installed list, the certificate is implicitly trusted, as shown in Fig. \ref{Secured Communication between the browser and the web server using the PKIX ecosystem and TLS}. 


### &#8592; [3. DNSSEC](DNSSEC.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Main Menu](README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [5. SOMETHING](.md) &#8594;
