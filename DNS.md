The history of the Internet goes back to the 1960's where it started out as a military project funded by the United States Department of Defense. The goal was to connect several pentagon institutions across the country using the existing telephone network. The project undertaken by the Advanced Research Projects Agency (ARPA) lead to the creation of ARPANET, the rudimentary ancestor of the Internet which later evolved in hardware, software and protocols to become the modern-day Internet. Given its small-scale initial deployment, one could justly say that the Internet was not expected to become the huge global network we know today. Therefore, the network functions designed back then were in line with the existing small-scale network. One function that stands out when talking about poor design is naming.

Previously, the mapping between host names and addresses was kept in a text file called HOSTS.TXT which was retrieved by all hosts via FTP. The bandwidth required to download this file was proportional to $N^2$ for a network of $N$ hosts [[RFC1034]](https://datatracker.ietf.org/doc/rfc1034/). This solution worked properly when $N$ was small, but the growth of the Internet mandated that a more scalable solution is found. This solution was the Domain Name System (DNS). 

The DNS was presented in RFC 1034 [[RFC1034]](https://datatracker.ietf.org/doc/rfc1034/) and RFC 1035 [[RFC1035]](https://datatracker.ietf.org/doc/rfc1035/). The goal was to develop a scalable and distributed system that performs the mapping between hostnames to addresses. In today's Internet, a crucial role of DNS is to map human-friendly domain names like *example.com* to machine-friendly IPv4 addresses like *192.0.2.1* or the less friendly alpha-numeric IPv6 addresses like *2001:db8:ff00:3f:d898:469f:2b95:162e*. Moreover, DNS moved from being a simple tool that had the sole purpose of referring to resources' location to becoming a crucial part of the Internet. Almost any interaction with the Internet today requires DNS. 

The main components of the DNS are:
\begin{itemize}
    \item The Domain Name Space:
    The domain name space is structured as an inverted tree and it evolves from the root at the top from which the resolving begins. Just below the root are the top-level domains (TLD). The Internet Assigned Numbers Authority (IANA) specifies four types of TLDS. TLDs could be generic TLDs (gTLD) e.g., .com, .net, country-code TLDs (ccTLD) e.g., .uk, .fr, sponsored TLDs (sTLD) e.g., .edu, .gov, and the infrastructre TLD (iTLD) .arpa. below top-level domains come the DNS domains. \note{Figure}
    
\begin{figure*}
    \Tree[.root 
            [..com [.example 
                    [.delegate ] 
                    [.subdomain ]]
                [.google [.domain subdomain ]]]
            [..uk [.domain subdomain ]]
            [..gov [.domain subdomain ]]]
    \caption{Caption}
    \label{fig:my_label}
\end{figure*}
    
    %\fi
    \item Name server:
    Name servers are programs that have the structure of a subset of the domain name. Name servers are said to be authoritative over a domain if they can give an answer about that domain. For example, the name server example is authoritative over the domain name example.com and could return a definitive answer for it. Name servers which are authoritative over a certain domain could have subdomains over which they are authoritative. In the figure, subdomain.example.com is a subdomain of example.com and the example name server is authoritative over it. Moreover, a name server could delegate authority to lower name servers. In the figure, example.com delegates authority to delegate.example.com which means that the example name server does not answer for domains ending in delegate.example.com. Name servers have text files called $Master Files$ that hold information about the zone or domain over which each name server is authoritative.
    
    \item Resource Records:
    A unit of information in a DNS $Master File$ is called a $Resource Record$ (RR). Each RR is a piece of information about a certain domain (or node in the domain name space tree) which a DNS client may ask for. Each RR has 6 fields as specified in RFC 1035 \cite{rfc1035}:
    \begin{itemize}
        \item NAME: specifies the owner of the RR which is the domain this RR belongs to or the node where it resides.
        \item TYPE: specifies the type of information the RR holds. For example, RR TYPE $A$ will hold the 32-bit IPv4 address of the domain, RR TYPE $AAAA$ will hold the 128-bit IPv6 address of the domain \cite{rfc3596}, RR TYPE MX returns the mail transfer agents of the domains and so on. 
        \item CLASS: defines the protocol family for the RR. For example, CLASS IN stands for Internet.
        \item TTL: specifies the duration the RR could be cached for before the node holding the RR is consulted again. 
        \item RDLENGTH: specifies the length in bytes of the RDATA field.
        \item RDATA: a string that describes the resource record.
    \end{itemize}
    
    \item Resolvers: act as the interface between user programs and the DNS servers. Whenever a DNS query is to be executed, a user consults a predefined resolver. This resolver could be a local one on the user's own machine or in the user's home/office, that of the user's ISP, or one of the Internet's public resolvers like google's 8.8.8.8 or cloudflare's 1.1.1.1 public DNS resolvers. 
    
\subsubsection{The DNS Lookup Process}
Each DNS lookup process starts when a certain resource on the Internet is requested. A DNS query is constructed and sent to a resolver. DNS Queries ask for a 1) QNAME or query name which is the domain name to be resolved. QNAME will contain a Fully Qualified Domain Name (FQDN) which is the complete name of the host/resource on the Internet. 2) QTYPE which specifies the type of information requested about the QNAME. 3) QCLASS which specifies the class of the query. For example, QCLASS IN is for the Internet. Queries could be handled either recursively or iteratively. Iterative servers will either return a definitive answer to a query if they are authoritative over the requested domain or simply inform the client that an answer was not found and if possible give referral to other servers. Recursive servers will return a definitive answer if they are authoritative over the requested domain or carry on with the query and ask other servers to finally return the answer to the client. Figure~\ref{dnsresolution} explains the process. In most cases the query is sent from the user's or DNS client's stub resolver, usually part of the browser or OS, to the recursive resolver. The recursive resolver will send the query to the root server which will give its referral to one of the TLDs. The resolver then sends the query to the TLD name server referred to it by the root server. The process repeats as the TLD refers to an authoritative server below it. This process goes on until an authoritative server of the QNAME contained in the query is reached after which this authoritative server returns a definitive answer, which is usually an address, to the recursive resolver. Finally, the recursive resolver delivers the answer to the DNS client. The process described assumes a cold cache in the resolver. This could be the case if the server's cache has been cleared or the server has just been started. In almost all other cases, and to expedite the resolving process and reduce DNS traffic on the network, the resolvers, and depending on the TTL field of every RR, temporarily cache the queries they resolve and answer related queries without consulting any authoritative servers.
\begin{figure}
    \centering
    \includegraphics[scale=0.7]{dns-resolution.pdf}
    \caption{DNS Lookup Process}
    \label{dnsresolution}
\end{figure}
\end{itemize}

### &#8592; [1. Introduction](Introduction.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [Main Menu](README.md) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [2. Domain Name System](DNS.md) &#8594;
