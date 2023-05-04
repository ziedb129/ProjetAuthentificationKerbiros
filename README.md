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

Once the setup is done, we can check if everything is working fine by using the nslookup command to query the DNS to obtain the mapping we just did and the ping command to ensure that all three machines are reachable.\

This an example in the kdc machine :


![ping2 ](https://user-images.githubusercontent.com/113895553/236020811-7c0af267-cf4e-4954-81f9-db0fd9999358.PNG)\
![ping ](https://user-images.githubusercontent.com/113895553/236020937-14fa0cbf-9a54-40cd-bd16-e870f39eca3f.PNG)\
Key Distribution Center Machine Configuration

Following are the packages that need to installed on the KDC machine

   $ sudo apt-get update
   $ sudo apt-get install krb5-kdc krb5-admin-server krb5-config
   
   During the installation, we will be asked for configuration of :

the realm : 'CENTRALE.TN' (must be all uppercase)


   ![config kerbiros kdc machine](https://user-images.githubusercontent.com/113895553/236022592-40788761-ff3f-4ea4-bc79-46b707af0a42.PNG)\
the Kerberos server : 'kdc.centrale.tn'\
   ![config hostname kerberos machine kdc](https://user-images.githubusercontent.com/113895553/236023036-78e6832e-d0c7-4442-b08c-8675575917b1.PNG)\
the administrative server : 'kdc.centrale.tn\
![interface config kerberos authentification ](https://user-images.githubusercontent.com/113895553/236023424-88b86935-1622-4d25-8a07-1e897cbd0b4b.PNG)\
Realm is a logical network, similar to a domain, that all the users and servers sharing the same Kerberos database belong to.

The master key for this KDC database needs to be set once the installation is complete :\
sudo krb5_newrealm\
![capture database master key](https://user-images.githubusercontent.com/113895553/236030135-e05c968d-d4e4-4e9c-bb2e-8ef7909d195e.PNG)\

The users and services in a realm are defined as a principal in Kerberos. These principals are managed by an admin user that we need to create manually :\
    $ sudo kadmin.local\
    kadmin.local:  add_principal root/admin\
    

![create principal admin root](https://user-images.githubusercontent.com/113895553/236046868-621b9bf8-f8e7-4847-9195-09e07bbb7685.PNG)\

kadmin.local is a KDC database administration program. We used this tool to create a new principal in the CENTRALE.TN realm (add_principal).\

We can check if the user root/admin was successfully created by running the command : kadmin.local: list_principals. We should see the 'root/admin@CENTRALE.TN' principal listed along with other default principals.\

![create root admin](https://user-images.githubusercontent.com/113895553/236050361-4261a08f-a470-4e2a-8277-9f20bce35325.PNG)\

Next, we need to grant all access rights to the Kerberos database to admin principal root/admin using the configuration file /etc/krb5kdc/kadm5.acl .\
sudo vi /etc/krb5kdc/kadm5.acl\

In this file, we need to add the following line :\

*/admin@CENTRALE.TN    *\



![config admin](https://user-images.githubusercontent.com/113895553/236051030-bb46912f-7624-473e-bd3b-4637749f194e.PNG)\
For changes to take effect, we need to restart the following service :\
sudo service krb5-admin-server restart\

Once the admin user who manages principals is created, we need to create the principals. We will to create principals for both the client machine and the service server machine.

![create un utilisateur client](https://user-images.githubusercontent.com/113895553/236077369-8f200604-422c-45e7-83da-fb46d574ff1f.PNG)


# Service server Machine Configuration

Configuration of Kerberos\
Installation of Packages\
Following are the packages that need to be installed on the Service server machine :

$ sudo apt-get update\
$ sudo apt-get install krb5-user libpam-krb5 libpam-ccreds\
During the installation, we will be asked for the configuration of :

the realm : 'CENTRALE.TN' (must be all uppercase)\
the Kerberos server : 'kdc.centrale.tn'\
the administrative server : 'kdc.centrale.tn\'\
PS : We need to enter the same information used for KDC Server.

PS: ! We need to have openssh-server package installed on the service server : 

~$ sudo su
~$ apt-get install openssh-server.

# Edit ssh config file

We need first to modify the ssh configuration files (sshd_config & ssh_config) by running the following commands:

$ nano /etc/ssh/sshd_config
$ nano /etc/ssh/ssh_config

![config service ssh](https://user-images.githubusercontent.com/113895553/236073342-9662a783-3b2e-43a5-839f-a79e174900c8.PNG)

To take effect, we need to restart the ssh service using the command:

$ systemctl restart sshd

# Preparation of the keytab file

Let's verify if the KEYTAB file is created by using the 'cat' command. we find it in the path '/etc/krb5kdc/kadm5.keytab'

![ajouter un utilisateur](https://user-images.githubusercontent.com/113895553/236081333-558ba367-f20d-4217-86e9-4aa7b4fc88e7.PNG)

Then we will be positioned in the same folder to manipulate the file using 'ktutil' 

We need to extract the service principal from KDC principal database to a keytab file.

In the KDC machine we run the following command to generate the keytab file in the current folder :
   $ ktutil 

![kadm5 keytab list](https://user-images.githubusercontent.com/113895553/236082819-53215ef4-f3ee-4a08-a8f8-4e81ecae4a20.PNG)


![radnkey](https://user-images.githubusercontent.com/113895553/236084402-60c41017-62ec-4e92-b029-b9996ed05e42.PNG)



![adduser utilisateur](https://user-images.githubusercontent.com/113895553/236084993-0baa350a-cad3-492f-b72d-67e667ff79f9.PNG)


![acces service ssh avec kdc centrale  tn](https://user-images.githubusercontent.com/113895553/236085087-08ba636c-3bf8-4b3a-a7fa-bcddd931b1d5.PNG)

![kinit et klist](https://user-images.githubusercontent.com/113895553/236085146-54e9acab-d93b-4264-a808-0c769f81c0b7.PNG)


