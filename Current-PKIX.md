# 4. The Current Public Key Infrastructure with X.509 Digital Certificates
he Transport Layer Security (TLS) protocol \cite{rfc5246, rfc8446} is one of the main elements of security in today's Internet that relies on the Public Key Infrastructure (PKI). For the most part, secure communications over the Internet start with a TLS handshake, as depicted in Fig.~\ref{fig1:tlshandshake}.
\begin{figure}[]
\begin{center}
\includegraphics[scale=0.7]{tls13-handshake.pdf}
\caption{TLS-handshake}
\label{fig1:tlshandshake}
\end{center}
\end{figure}
A TLS client connecting to a TLS server will receive that server's X.509 certificate. The elements of the certificate include information about the certificate issuer, information about the recipient of the certificate and the certificate's digital signature. The client ensures that the server is whom it claims to be by verifying that the certificate the server sent is legitimate. 
A chain of trust exists to help TLS clients verify the authenticity of certificates. This chain of trust starts at the top with root CAs. Root CAs are trusted by default and have self-signed certificates that allow them to issue other certificates creating intermediary CAs. Intermediary CAs, which the root CAs trust, can issue certificates for servers and websites. An X.509 certificate contains several fields describing the issuing CA and its owner. The issuing CA signs the certificate by encrypting the hash (digest) of the certificate fields with its private key.  
The problem with the current model is that certificates are not required to be verified by the CA that issued them. Moreover, a CA that wants its certificates to be trusted must be added to the root store containing the list of trusted root CA certificates. Different vendors and browsers have different root stores. Moreover, the root and intermediary CAs are not immune to security breaches \cite{enisa, billcorbitt}, and could, in theory, issue certificates for any domain like the root authority Diginotar did when it issued a fraudulent Google certificate in 2011 \cite{diginotargoogle}. 
