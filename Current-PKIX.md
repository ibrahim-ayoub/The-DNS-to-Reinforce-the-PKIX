# 4. The Current Public Key Infrastructure with X.509 Digital Certificates
The Transport Layer Security (TLS) protocol is one of the main elements of security in today's Internet that relies on the Public Key Infrastructure (PKI). For the most part, secure communications over the Internet start with a TLS handshake, as depicted in *Figure 4*.
<!--- ---------------------------------------------------------------------------------------------------------------- -->
<p align="center">
  <img src="/images/tls13-handshake.jpg" />
</p>
<p align = "center">
Figure 4 - TLS Handshake
</p>
<!--- ---------------------------------------------------------------------------------------------------------------- -->
A client that wants to connect to a web server obtains the server's IP address via a DNS resolution as explained in [2](DNS.md) and \textbf{\textit{secondly}} it connects to the domain’s web server using the IP address via Hyper Text Transfer Protocol (HTTP) connection.A TLS client connecting to a TLS server will receive that server's X.509 certificate. The elements of the certificate include information about the certificate issuer, information about the recipient of the certificate and the certificate's digital signature. The client ensures that the server is whom it claims to be by verifying that the certificate the server sent is legitimate. 

<!--- ---------------------------------------------------------------------------------------------------------------- -->

<p align="center">
  <img src="/images/tls-session-setup.jpg" />
</p>
<p align = "center">
Figure 5 - TLS Session Setup
</p>
<!--- ---------------------------------------------------------------------------------------------------------------- -->

A chain of trust exists to help TLS clients verify the authenticity of certificates. This chain of trust starts at the top with root CAs. Root CAs are trusted by default and have self-signed certificates that allow them to issue other certificates creating intermediary CAs. Intermediary CAs, which the root CAs trust, can issue certificates for servers and websites. An X.509 certificate contains several fields describing the issuing CA and its owner. The issuing CA signs the certificate by encrypting the hash (digest) of the certificate fields with its private key.  
The problem with the current model is that certificates are not required to be verified by the CA that issued them. Moreover, a CA that wants its certificates to be trusted must be added to the root store containing the list of trusted root CA certificates. Different vendors and browsers have different root stores. Moreover, the root and intermediary CAs are not immune to security breaches \cite{enisa, billcorbitt}, and could, in theory, issue certificates for any domain like the root authority Diginotar did when it issued a fraudulent Google certificate in 2011 \cite{diginotargoogle}. 


 

For the first operation, securing the communication during DNS resolution could be provided by DNS Security (DNSSEC) \cite{rfc4398} described in subsection \ref{Using DNS infrastructure and its security extensions as PKI}. For the second operation, the Transport Layer Security (TLS) protocol comes to the rescue, allowing the client and the server to authenticate each other and negotiate an encryption algorithm and cryptographic keys before the data is exchanged. TLS ensures that data cannot be tampered with during transit since the data is encrypted. Details of the second operation is explained in subsection \ref{Public-Key Infrastructure X.509} 

\subsection{Public-Key Infrastructure X.509 (PKIX)}
\label{Public-Key Infrastructure X.509}
Encrypting and decrypting the data in the TLS protocol is done by a matching pair of cryptographic keys: \textbf{\textit{public}} and \textbf{\textit{private key}}. The Data encrypted by a public key can be decrypted only by the corresponding private key and vice versa, enabling secure communication with unknown users. 

A website (e.g., a bank) publishes its public key for anyone to download. An account holder in the bank, Alice, encrypts a message using the public key and sends it to the bank. Only the bank can decrypt the message using its private key. Thus, Alice is sure that her message is accessed only by the bank and not by anyone else. 

On the other hand, there is a possibility that an impersonator publishes their public key posing as Alice's bank. Alice will encrypt the message using the public key and send it to the impersonator, thinking she is communicating with her bank. The impersonator could do a man in the middle and copy the message. As the impersonator is the owner of the public and the private key, it will enable them to decrypt and read the message sent by Alice. Thus, there arises a possibility that anyone can create a public key for accessing any domain name. 

Hence, binding between the identity (e.g., the domain name) and the public key is necessary. The X.509 standard \cite{x.509-ITU} proposed by the ITU and ISO provides a mechanism to bind a particular public key to a specific identity. The domain holder can do this binding, and in that case, it is called a self-signed certificate. If the self-signed certificate is obtained from a trusted source by the application using the certificate for authentication, then it is accepted. Otherwise there is no guarantee of the certificate's authenticity.

\subsubsection{The Certification Authorities (CAs) role}
This is where the need for a \textbf{\textit{trusted third party}} arises. It is just like the passport wherein it can be issued only by a trusted third party delegated by a country's government. The trusted authority attests that the person in the photo is identified by a particular name, surname and other credentials.

In web browsing, a \textbf{\textit{digital certificate}} issued for a domain name, is like the passport.  In the PKIX ecosystem, the role of the trusted authority is played by organizations called CAs. A certificate issued by a given CA binds the given domain name with information such as the certificate assignee, the entity which has requested the certificate, its validity period etc. The CA is used to assert to the browser that the web server represents the domain asked by the client. A TLS connection is established between the browser and the web server on successful authentication of the certificate. With the TLS connection established, the traffic between the browser and the web server is encrypted and is protected against any third-party eavesdropping.

Similar to how a passport attested by one country's trusted authority is accepted by other countries as a validated document for authenticating a person, browser vendors (such as Firefox, Chrome, Internet explorer, Safari etc.,) accept digital certificates created only by certain CAs. The browser vendors authorize an organization to be a CA only after understanding that they are trustworthy and they follow strict principles and procedures to provide certificates only for correct domain holders. Once the browser vendors authorize an organization to be a CA, the latter's digital certificate is added to the list of trusted CAs in the browser library. Thus, once a client using a browser accesses a domain name which has a digital certificate generated by one of the CAs among its pre-installed list, the certificate is implicitly trusted, as shown in Fig. \ref{Secured Communication between the browser and the web server using the PKIX ecosystem and TLS}. 


### &#8592; [1. Introduction](Introduction.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Main Menu](README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [3. DNSSEC](DNSSEC.md) &#8594;
