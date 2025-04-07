<h2><p align="center">
    <strong>Development of 5G stand-alone network environment with network slicing using open5GS and UERANSIM</strong></h2>
    <br>

<br/>

## 1. Overview
This repository presents a comprehensive 5G ecosystem that includes various User Equipments (UEs), next-generation base stations (gNBs), Network Functions (NFs), network slices, and Data Networks (DNs). The project aims to simulate realistic 5G scenarios where UEs can connect to multiple gNBs and run different applications, such as Voice over IP (VoIP), video streaming, and file sharing.

**Open5GS** is an open-source, software-defined framework written in C that implements both the Evolved Packet Core (EPC) and the 5G Core Network. It includes essential network components like the Access and Mobility Management Function (AMF), User Plane Function (UPF), Session Management Function (SMF), and Network Slice Selection Function (NSSF). Open5GS supports 3GPP Release-17 specifications for both 4G LTE and 5G NR networks and can be installed using package managers on systems like Debian, Ubuntu, and openSUSE. For testing and management, it provides a WebUI built with Node.js.

**UERANSIM** is an open-source project designed to emulate 5G network components in line with 3GPP standards, particularly focusing on standalone mode operation of both 5G UEs and gNBs. Integrating UERANSIM with the Open5GS core requires some configuration adjustments to align with the specific setup of the test environment.

## 2. General Objectives
The main aim of this project is to connect the Open5GS 5G Core with the UERANSIM 5G Radio Access Network (RAN) to develop a fully operational 5G Standalone (SA) network. This network will incorporate multiple User Equipments (UEs), gNodeBs (gNBs), and Network Functions (NFs), and will utilize network slicing to support various services, including VoIP, video streaming, and file sharing.

## 3. Architecture and realization
The diagram below represents the 5G network architecture employed in this project. As depicted, the setup includes multiple User Equipments (UEs), gNodeBs (gNBs), and Network Functions (NFs), along with Network Slicing (NS) to simulate a complete 5G environment. This architecture supports the implementation of various services such as VoIP, video streaming, and file transfers using network slicing.

![Project Architecture with IP Details](https://github.com/user-attachments/assets/ead0e312-8ae3-4dca-b2e3-0fbc513e68af)

- In this setup, we made 4 VMs with the following considerations:
  - VM1 is for the Control Plane (CP) of open5gs, including network functions like multiple AMFs, SMFs, UDM, NSSF, etc.
  - VM2 is for the User Plane Functions (UPFs) of Open5gs.
  - VM3 is for multiple gNBs of UERANSIM.
  - VM4 is for multiple UEs of UERANSIM and the Application Server where Kamailio, Owncast, and Next Cloud are installed.

## 4. Setting of 5QI values for UEs
- 5QI stands for 5G QoS Identifier and it is equivalent to QCI in LTE. It is an indicator that represents the level of Quality of Service. 5QI value is set for specific UEs as shown below:
  | UE # | IMSI | Network slices | DNN | 5QI value as per TS 23.501 |
  | --- | --- | --- | --- | --- |
  | UE1 | 001010000000000 | sst: 1 ; sd: 000001 | internet | 6 |
  | UE4 | 001010000000001 | sst: 1 ; sd: 000002 | internet2 | 6 |
  | UE2 | 001010000000002 | sst: 2 ; sd: 000001 | voip <br> voip2 | 1 |
  | UE3 | 001010000000003 | sst: 2 ; sd: 000002 | voip3 | 1 |

- The 5QI value of 1 is considered suitable for services such as Conversational voice as per TS 23.501 whereas the 5QI value of 6 is considered suitable for services such as Video (Buffered Streaming), and TCP-based (e.g., www, e-mail, chat, ftp, p2p file sharing, progressive video, etc.) as per TS 23.501.

- As video streaming and file transfer were tested between UE1 and UE4, the value is set to 6 in the profile of UE1 and UE4 in the open5gs GUI. Similarly, for VOIP testing between UE2 and UE3, the value is set to 1 in the profile of UE2 and UE3 in the open5gs GUI.<br>

## 5. Application Testing

### 5.1. VoIP Testing with Kamailio Server
- The VOIP call testing can be performed by using the external SIP server. The external SIP server installed in the VM is Kamailio which is an open-source SIP server written in C language and runs on the Linux/Unix operating systems (OS). The Kamilio server is installed in VM4 i.e., Application Server.
- Moreover, the SIP client is required and for this setup, ‘Zoiper 5’ i.e., free VOIP softphone is used as a SIP client.The "Zoiper5" is downloaded in the VM and SIP accounts are created for both UE2 and UE3 in Zoiper5 using:
  - SIP URI for UE2: ue2@192.168.50.59
  - SIP URI for UE3: ue3@192.168.50.59
  - SIP server domain is 192.168.50.59:5060.
- Now the Zoiper SIP softphone application is binded to respective UE's i.e., UE2 and UE3 by using the command "nr-binder".
- Afterwards, call from UE3 is made to UE2 in Zoiper App.

### 5.2. Video Streaming Testing with OwnCast and OBS
- OBS (Open Broadcaster Software) is a popular open-source software used for live streaming and recording video content. When used in conjunction with an Owncast server, OBS serves as the client-side software responsible for capturing, encoding, and streaming video content to the Owncast server, which then distributes the video stream to viewers.
  <br>
- For the Video Streaming testing, a sample video is streamed through the DNN 'internet' of UE1 from the OBS software, and the stream is received in the Owncast server through the DNN 'internet2' of UE4. Now OBS is started through the DN of UE1 by using the nr-binder command. Moreover, google-chrome is started through the DN of UE4 by using the nr-binder command.
- Video Streaming Started in OBS.
- Afterwards, Owncast video stream URL "http://192.168.50.59:8080/" is opened in the google-chrome browser accessed through the DN of UE4 & successful video streaming is received at the owncast server end.

### 5.3. File Transferring Testing with NextCloud
- For hosting the Nextcloud server, VM1 is used where the Open5GS Control Plane installation was performed. An IP address of `192.168.50.60` was assigned to the Nextcloud server. NAT handles the inbound and outbound traffic to the Nextcloud.

## 6. Limitations and Challenges Encountered During Project Execution
- The limited RAM available on our personal computer posed challenges in running multiple virtual machines simultaneously, often leading to system freezes and performance issues.
- The free version of the Zoiper5 application allows only one SIP user per account at a time. However, VoIP testing requires at least two UEs, each with its own SIP identity. As a result, Zoiper5 had to be installed and configured on two separate VMs, each representing a UE. Furthermore, to support Zoiper functionality on these VMs, UERANSIM also had to be built and configured individually on both machines.
## 7. Learnings during Project Execution
- Detailed learning of 5G architecture.
- Hands-on practice gained with Open5GS and UERANSIM.
- Hands-on practice gained with Linux.
- Gained knowledge about network slicing.
- Gained knowledge about virtualization and creating virtual machines in Oracle VM VirtualBox.
- Gained knowledge about Kamailio SIP server, OBS, Nextcloud, Zoiper SIP Client, iPerf, and Owncast.

## 8. Conclusion
- The end-to-end implementation of the emulated 5G standalone environment using Open5GS and UERANSIM has been successfully completed for both case setups.
- The implementation of network slicing has been successfully accomplished to run different services on UEs, utilizing parameters such as SST and SD, as well as 5QI values.
- Handover testing has been successfully conducted on UE2, utilizing the TAC parameter.
- Real-world services including VOIP calling, file transferring, and video streaming have been successfully tested on both case setups.
- Downlink throughput testing and internet connectivity have been verified for multiple UEs across both test setups.
  
 ## References
- 5QI table: https://www.sharetechnote.com/html/5G/5G_5QI.html
- DL throughput testing using iperf: https://hackmd.io/@JenRungHuang/By4zdoBdB
- Kamailio: https://www.kamailio.org/w/
- Nextcloud: https://nextcloud.com/.
- OBS: https://obsproject.com/
- Open5gs: https://open5gs.org/open5gs/docs/
- Owncast: https://owncast.online/quickstart/installer/.
- Trick, U. (2021): An Introduction to the 5th Generation Mobile Networks, ISBN 978-3-11-072437-0, Frankfurt am Main: De Gruyter Oldenbourg. 
- UERANSIM: https://github.com/aligungr/UERANSIM
- Zoiper: https://www.zoiper.com/en/voip-softphone/download/current
