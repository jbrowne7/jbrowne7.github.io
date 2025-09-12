---
title: "Secure Remote Access Solution with BYOD Policy"
date: 2024-11-04
media_subpath: /assets/posts/2024-11-04-secure-remote-access-solution-with-byod-policy
published: true
layout: post
categories:
- home lab
tags:
- openvpn
- azure
- cloud
- byod
---

Scenario, a small-sized start-up with remote employees wants to ensure they meet certain security standards while enabling seamless access to company resources. They need a solution to allow this.


## Demo


{% include embed/youtube.html id="bAP_b_181Jg" %}





<!-- The objective for this project is to implement a VPN solution for secure remote access and create a BYOD policy that ensures employee devices meet security standards.

Security objectives:
 - Data Protection: Implement measures to protect sensitive data from unauthorized access during transmission and on personal devices.
 - Access Control: Ensure that only authorized personnel can access critical company resources, regardless of their location.
 - Compliance: Adhere to industry standards and regulations related to data protection and privacy
 - User Awareness: Educate employees about security best practices, including how to securely connect to the VPN and the importance of adhering to the BYOD policy.
 - Incident Response: Establish a clear protocol for responding to security incidents related to remote access or BYOD, including reporting lost devices or suspected breaches.

How we will implement the solution to meet our security objectives:
 - OpenVPN for remote access
 - MFA to improve security for VPN login
 - BYOD policy that outlines security requirements for personal devices, including encryption, password protection, and regular updates.
 - Define employee responsibilities regarding device security and acceptable use of company resources.
 - Training sessions for employees to familiarize them with the VPN connection process, BYOD policy, and security best practices.
 - Resources (e.g., documentation, FAQs) to support employees in maintaining device security.
 - Monitoring tools to track VPN usage and ensure compliance with the BYOD policy.

## OpenVPN connection and MFA

Here's the workflow of how the client will connect to the VPN

1. Client initiates openVPN connection with openVPN server using their openVPN configuration file, digital certificate, private key, and username and password, sending this information securely (maybe via HTTPS?)
2. openVPN server verifies certificate and username and password using PAM, then responds with a request to verify via google authenticator
3. Client responds with their TOTP
4. openVPN server verifies the TOTP with a google authenticator PAM
5. Connection is established

We'll be deploying the solution entirely on-premises so for the VM setup I'm using:
- Ubuntu VM for CA
- Ubuntu VM for openVPN server and authentication system
- Windows 10 VM as the client
- Ubuntu VM to act as a private resource for the organisation which hosts a git repo which should only be accessed by employees

### CA setup

We'll be using easy-rsa to setup the CA, it can be done with the openSSL package but easy-rsa simplifies it alot.

`sudo apt install easy-rsa`  Install easy-rsa package  
`mkdir ~/ca` create dir to hold ca files  
`cp -r /usr/share/easy-rsa/* ~/ca` copy easy-rsa files to ca dir  
`./easyrsa init-pki` initialise pki  
`./easyrsa build-ca` create ca

Now we have our CA keypair and certificate


### Generate server CSR

we'll use openVPN to generate the CSR need to install the following packages:  

`sudo apt install openvpn openssl`  

Now we can generate a key-pair, we'll use RSA for our key pair:  

`openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048`  

`openssl rsa -pubout -in private_key.pem -out public_key.pem`  

In the first command we generate our 2048 bit RSA private key and in the second command we extract our public key from the private key.  

Next we navigate to the cd where our easyrsa binary is (in my case /usr/share/easy-rsa/ ) and execute:

`openssl req -new -key private_key.pem -out server1.req`  

This generates our CSR in the file server1.csr

### Upload CSR and Sign CSR

In a real world environment we may have a web interface our some API to send our CSR to the CA and receive the result but in this project we will just use sftp for communication

`sftp <hostname>@<ip>`  

`put server1.csr ca/pki/reqs/server1.req`


Now on the CA server

`./easyrsa sign-req server server1`

And back on the openVPN server

`get ca/pki/issued/server1.crt server1.crt`

Now our openVPN server has a valid certificate

### Client certificate

On the CA server

`openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048`   

`openssl rsa -pubout -in private_key.pem -out public_key.pem`   

`openssl req -new -key private_key.pem -out client1.req`  

`mv client1.req pki/reqs/client1.red`  
 
`./easyrsa sign-req client client1`  

Then use filezilla to download the server certificate, CA certificate and key pair, then we can delete the key pair from the CA server

Also download openVPN on the windows machine

### VPN connection

Before starting the openVPN server there are a few more things we need to do, we need to generate diffe hellman parameters which are used when establishing the VPN connection, on the CA server  

`openssl dhparam -out dh2048.pem 2048`  

We also need to edit the openVPN conf file, we can copy the example and change a few parameters

`cp /usr/share/doc/openvpn/example/sample-config-files/server-conf ~/server.conf`  

And change the following values

`ca` to CA certificate path
`cert` to server certificate path
`key` to server private key path
`dh` to dh parameters path

We must also generate a secret shared-key for tls-auth, we can do this using 

`openvpn --genkey --secret ta.key`  

And copying this key to our server and client machines, in my case I generated it on the server and then used sftp to transfer it onto the client

Now on the client we get the sample client conf from the openvpn github and change the same values values except the dh parameter to the client paths, set the remote parameter to the openVPN servers IP and then we can run

`sudo openvpn server.conf`  on the server

and on the client we can run the openVPN gui and we're connected

### Privately hosted gitlab setup

Now on the third VM which is running ubuntu we need to install gitlab, we can simply do this by executing the following commands  

`curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash`  

`sudo apt install gitlab-ce`  

`cat /etc/gitlab/initial_root_password`  

Navigate to localhost and log in using root and the password obtained in the last command

Since we want to ensure secure access and considering the scenario first we disable sign-up enabled

We want to setup a digital certificate on this server as well so it can connect to our VPN. I wont go into detail on this since its pretty much the same as the previous steps for generating the certificate for the openVPN server and the windows machine.

Now we have our gitlab setup, windows machine, and openVPN server its time to configure the gitlab server to only receive connections from our VPN subnet, to do this we'll use `ufw`.

Currently if we go to the ip of our gitlab machine from our openVPN server we should be able to access the gitlab page, which isn't what we want, so to only allow connections via the VPN we can add
the following firewall rule  

`sudo ufw allow from <VPN subnet> to any port 80`  

`sudo ufw enable`  

in my case my VPN subnet (defined in the server.conf file) was 10.8.0.0/24

(also note we'll just be going with HTTP for now for simplicity, although in a real world scenario it would be very important to use HTTPS for security)

Now if we go to the gitlab server via its VPN subent IP we should be directed to the gitlab site but if we go to the server via its public IP it wont be reachable  

(also note in a real world scenario we may set static IPs and use DNS to make this process more seamless, i.e we dont need to check what IPs been assigned to the machine when it connects to the VPN)

### Securing VPN connections with MFA

To strengthen the security we want to enable MFA for clients connecting to the VPN, we'll be using Google Authenticator for the TOTP

We'll be using the google authenticator PAM

first we need to install the PAM on our openVPN server

`sudo apt install libpam-google-authenticator`  

Then add the following line to enable the PAM

`auth required pam_google_authenticator.so`  

Each user needs to setup their own google authenticator token, switch to this user (in my case `evan`) and run  

`google-authenticator`  

And setup the TOTP  

Now to edit the server.conf to ensure that google authenticator is used for connecting to the server, we need to add the following lines to our server.conf  

`plugin /usr/lib/openvpn/openvpn-plugin-auth-pam.so openvpn`  
`username-as-common-name`

Now we can start openVPN with this conf -->


