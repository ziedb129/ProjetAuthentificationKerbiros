# ProjetAuthentificationKerbiros
Enabling GSSAPI / Kerberos authentication in SSH will allow single-sign-on – i.e. authentication without using the standard username and password for SSH clients.
#  Steps To Setup Kerberos On UBUNTU
Kerberos is a network authentication protocol used to verify the identity of two or more trusted hosts across an untrusted network. It uses secret-key cryptography and a trusted third party (Kerberos Key Distribution Center) for authenticating client-server applications. Key Distribution Cente (KDC) gives clients tickets representing their network credentials. The Kerberos ticket is presented to the servers after the connection has been established.
   # Principe de Kerberos
   ![kerberos](https://user-images.githubusercontent.com/113895553/235969169-f22bd5a9-a23e-40d2-b3fb-af8af9870f59.jpg)
# Hostname and IP Addresses
We will need three machines. In my case I'm using three ubuntu machines : my physical machine and two virtual machines inside of VMware Workstation. My physical machine will be the client and the two other machines will be the Service Server and the KDC.
We can check the IP addresses of all three machines by running hostname -I 
in each one.
In my case :

Client machine ip address is 192.168.75.131\
Service server machine ip address is 192.168.75.134\
KDC machine ip address is 192.168.75.133\
Now that we added ip addresses to the virtual machines, we will start by setting hostnames for each machine :
* KDC machine\
 hostnamectl --static set-hostname kdc.centrale.tn
 




![cap1](https://user-images.githubusercontent.com/113895553/235993494-203e4490-99e0-41f8-a148-c83763d74f79.PNG)

And open a new terminal for changes to take effect.

We can check the hostname of a machine by running the command : hostname

Next, we will be mapping these hostnames to their corresponding IP addresses on all three machines using /etc/hosts file.\
sudo vi /etc/hosts


