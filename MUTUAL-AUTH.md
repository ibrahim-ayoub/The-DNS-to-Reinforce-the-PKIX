# 6. Mutual Authentication for IoT Backend Servers
### 6.1. IoT System Overview

IoT connectivity technologies could be classified broadly into three categories: Short Range (Bluetooth, Zigbee, Zwave), Medium range (WiFi) and Long-range (LoRa, NB-IoT, Wi-Sun, Sigfox). We plan to validate our hypothesis of using DNS-based PKI on LPWAN (Low-Power Wide Area Networks), considered the most resource-constrained IoT networks, and then extend them to other types of networks. If the DNS-based PKI work for LPWAN, it should also work for other less constrained networks. 

In the LPWAN category, we short-listed LoRaWAN since, compared to other IoT connectivity technologies, the LoRaWAN ecosystem provides the freedom to its stakeholders to make their own choices in choosing the ED manufacturers, Service providers, and applications providers. Since the radio connectivity uses a license-free spectrum, the freedom of choice in LoRaWAN extends to deployment options. Public LoRaWAN has nationwide coverage;  private LoRaWAN focuses on specific use-cases and community networks that can be used for free by end-users. 

LoRaWAN is an asymmetric protocol with a star topology  as shown in the *Figure 7*. Data transmitted by the IoT End-Device (ED) is received by a Radio Gateway (RG), which relays it to a Network Server (NS). The NS decides on further processing the incoming data based on the ED’s unique identifier (DevEUI). The NS has multiple responsibilities like forwarding the uplink from the ED to the Application Server (AS), queuing the downlink from the AS to the ED, forwarding the ED onboarding request to the appropriate AA (Authentication & Authorization ) servers, named as Join Server (JS) in LoRaWAN terminology. While the ED is connected to the RG via LoRa modulated *RF messages*, the connection between the RG, the NS and the AS is done through *IP traffic* and can be backhauled via Wi-Fi, hardwired Ethernet or Cellular connection.

<p align="center">
  <img src="/images/lw-key-dist.jpg" />
</p>
<p align = "center">
Figure 7 - LoRaWAN Key Distribution
</p>

The JS acting as the AA server controls the terms on how the ED gets activated (i.e., onboarded) to a selected LoRaWAN.  There are two types of ED activation: ***Over the Air Activation (OTAA)*** and ***Activation by Personalization (ABP)***. With ABP, the ED is directly connected to a LoRaWAN by hardcoding the cryptographic keys and other parameters required for secured communication. With OTAA, the parameters necessary to create a secured session between the ED and the servers in the Internet are created dynamically. This secured session is similar to the TLS handshake used in the HTTPS connection. OTAA is preferred over ABP since it is dynamic, decouples the ED and the backend infrastructure, and doesn’t need session keys to be hardcoded. 

The ED performs a Join procedure (i.e. the onboarding process) with the JS during OTAA by sending the Join Request (JR). The JR payload contains the ED’s unique identifier (i.e. the ***DevEUI***), the cryptographic AES-128 root keys: ***NwkKey***, ***AppKey*** and ***JoinEUI*** (unique identifier pointing to the JS). The JS associated with the ED  has preliminary information such as the ED’s ***DevEUI***, the cryptographic keys: ***NwkKey*** and ***AppKey*** (as shown in the *Figure 7*) required for generating session keys to secure the communication between the ED and the NS and AS.  

### 6.2. Key Sharing challenge between multiple Stakeholders

As shown in the *Figure 8*, the IoT ED manufacturer injects the root keys - ***NwkKey*** and ***AppKey*** into the ED. Since LoRaWAN has the potential of multiple stakeholders, there is a need for these root keys to be pre-shared with them. The process of sharing the root keys is currently done by printing the keys behind the device, sending them via mail etc. which is not secure.  

<p align="center">
  <img src="/images/lw-key-injection-sharing.jpg" />
</p>
<p align = "center">
Figure 8 - LoRaWAN Key injection & sharing
</p>

One method of solving this operational nightmare is to use asymmetric keys based on a PKI as used to secure web traffic.

### 6.3. PKI Challenges in IoT

#### 6.3.1. Constrained IoT devices and network

LoRa-based IoT devices are highly constrained: they have little memory, limited processing capacity, and limited power. We plan to experiment with our hypothesis on Class 0 and 1 \cite{rfc7228}, which are very constrained with RAM size much less than 10 KB and flash memory much less than 100 KB. For Class 1, they are around 10 KB and 100 KB, respectively.

The LoRaWAN specification \cite{lora_spec} defines a maximum data payload for each worldwide region. This full payload size varies by DataRate (DR) because of the maximum on-air transmission time allowed for each regional specification. While 52 bytes is the maximum payload in Europe, in the U.S. and other regions that operate in the 900MHz ISM band, the lowest DR is restricted to 11 bytes. These limits are chosen to meet the requirements of the regulatory agencies of the region.

Due to the ED and the network constraints, The CA model for issuing the X.509 digital certificates is not operationally feasible for LoRaWAN. The primary issue with the X.509 digital certificates is its size. Thus not compatible with resource-constrained IoT networks. 
Asymmetric Keys using PKI, which have worked well for secure Internet communication, cannot be used due to its size. It is impossible to send a 2048 byte X.509 digital certificate over a LoRaWAN Communication which has an MTU (Maximum Transfer Unit) of 52 bytes. 

#### 6.3.2. Non Availability of dedicated CA infrastructure for IoTs

In the web, the browser client (such as Chrome or Firefox) has a certificate store containing thousands of Root CA certificates. The browser authenticates any server that delivers an X.509 certificate digitally signed by anyone of the Root CA in its certificate store. Such certificate store infrastructure is unavailable in the LoRaWAN backend network elements or any IoT backend infrastructures. 

#### 6.3.3. Cost
Even if we assume the infrastructure exists, the digital certificates come at a cost, which is not viable for most IoT services. We tried "Let’s encrypt", which provides X.509 digital certificates for free. However, it was not possible to benefit since they do not offer certificates for domain names with more than ten labels (JoinEUI has more than 16 labels). A viable solution to resolve the operational and cost issue is to generate our Self-Signed certificates.

### 6.4. Using DNS-based PKI in IoT

We propose designing an open PKI for IoT using the DNS protocol and its security extensions - DNSSEC and DANE.

For secure ED onboarding, the interface between the ***backend network elements*** (the NS, JS and the AS) in the IP space (*Figure 7*) should be mutually authenticated (i.e., both the client and the server authenticate each other), as per the [LoRaWAN backend specifications](https://lora-alliance.org/sites/default/files/2018-04/lorawantm_specification_-v1.1.pdf). Nevertheless, how mutual authentication should be done is left to the implementer’s choice and is not normative.

A viable solution to resolve the operational and cost issue is to generate Self-Signed certificates.

<p align="center">
  <img src="/images/CA_Provisioning_Architecture.png" />
</p>
<p align = "center">
Figure 9 - CA Provisioning Architecture
</p>

A CA provisioning infrastructure (*Figure 9*) was set up where Afnic emulated the Root CA role and generated intermediate certificates for two LoRaWAN - TSP (Telecom Sud Paris) and Afnic Labs. Complete details on setting up the infrastructure is provided in the [Quick Start guide](https://github.com/AFNIC/IoTRoam-Tutorial/blob/master/QuickStart.md). 

During testing, we identified that combining the intermediate and the server leaf certificate (a combined trust chain - *Figure 10*) during a TLS handshake could bypass the need for having a certificate store with all intermediate certificates. The validating server needs to store only the root CA certificate. The certificate validation process is done by sending the combined trust chain to the server’s IP address. On receiving the combined trust chain, the server first verifies the leaf certificate in the combined trust chain. When the leaf certificate is unknown, it checks the following certificate in the chain, the intermediate certificate. Since the root CA signs the intermediate certificate, the combined certificate chain becomes trusted. Thus, the backend network elements (NS, AS and the JS) could be mutually authenticated even if they are in different networks since they have a common root CA at the top of the chain of trust.

Since the infrastructure uses self-signed certificates, there is no cost involved. 

<p align="center">
  <img src="/images/CA_Validation_Architecture.png" />
</p>
<p align = "center">
Figure 10 - Certificate Validation Process
</p>

### 6.4. Need for DANE?

The issue with the certificate provisioning infrastructure using a single root CA (*Figure 9*) creates a single private PKI. This approach fails from an operational feasibility perspective as a number of stakeholders are not willing to be restricted to a single CA. 

The [IETF DANCE (DNS Authentication of Named Clients Everywhere)](https://datatracker.ietf.org/wg/dance/about/) WG (Working Group) seeks to make PKI-based IoT device identity universally discoverable, more broadly recognized, and less expensive to maintain by using DNS as the constraining namespace and lookup mechanism. DANCE builds on patterns established by the original DANE RFCs ([[RFC6698]](https://datatracker.ietf.org/doc/rfc6698/), [[RFC7218]](https://datatracker.ietf.org/doc/rfc7218/), [[RFC7671]](https://datatracker.ietf.org/doc/rfc7671/)) to enable client and sending entity certificate, public key, and trust anchor discovery. DANCE allows entities to possess a first-class identity, which, thanks to DNS, may be trusted by any application also relying on the DNS. A first-class identity is an application-independent identity.

The IETF DANCE WG is discussing two Internet drafts: [TLS Extension for DANE Client Identity](https://www.ietf.org/archive/id/draft-huque-tls-dane-clientid-06.txt) and [TLS Client Authentication via DANE TLSA records](https://www.ietf.org/archive/id/draft-huque-dane-client-cert-08.txt) based on DANE, which possibly solve the single root CA issue. For this to work, a TLS Client should have a signed DNS [TLSA record](DANE.md) published in the DNS zone corresponding to its DNS name and X.509 certificate or public key.

The draft [TLS Client Authentication via DANE TLSA records](https://www.ietf.org/archive/id/draft-huque-dane-client-cert-08.txt) specifies a TLS extension to convey a DANE Client Identity to a TLS server. The extension contain the client identity in the form of the DNS domain name that is expected to have a DANE TLSA record published for it as shown in the example below:

```
 light_sensor._device.example.com. IN TLSA (
        3 1 2
        0f8b48ff5fd94117f21b6550aaee89c8
        d8adbc3f433c8e587a85a14e54667b25
        f4dcd8c4ae6162121ea9166984831b57
        b408534451fd1b9702f8de0532ecd03c )   
```
During the TLS handshake, the server requests a client certificate (via the *Client Certificate Request* message). The server then extracts the DANE client identity, constructs the DNS query name for the corresponding TLSA record and authenticates the client's certificate or public key. During mutual authentication, both the client and the server could be authenticated as shown in the Fig.\ref{Mutual authentication facilitated by DANE} 

DANE-based mutual authentication enables using a self-signed certificate with different Root CA's. Thus, each institution can choose its Root CA to sign the certificates and validate dynamically based on DANE client identity.   

<p align="center">
  <img src="/images/mutual-auth.jpg" />
</p>
<p align = "center">
Figure 11 - Mutual Authentication Facilitated by DANE
</p>
