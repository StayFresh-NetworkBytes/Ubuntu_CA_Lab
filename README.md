# Cisco Modeling Labs - Ubuntu CA Lab

This lab allows you to build your own Ubuntu CA server to run int Cisco Modeling Labs.  As part of the Ubuntu Cloud-Init process, a couple things occur:

- /etc/hosts is updated to update the hostname and provide a fully qualified domain-name.  You can use this, or update it with the domain suffix you intended to you in your lab.

  path: /etc/hosts
  permissions: '0644'
  content: |
    # Enter Content below
    127.0.0.1 localhost
    127.0.1.1 ubuntu-ca ubuntu-ca.cml.lab

- A bash scripted called /root/create_ca.sh is used to create a CA directory and additional directories to organize certificates, write files to track and assign serial numbers, create an aes256 enrypted private key, and finally create a CA in the Country (US), State (MD), Locality (Baltimore), Organization (CML), with a Common Name (ubuntu-ca.cml.lab **Note this matches what we see in /etc/hosts**).  The CA is created for 3650 days or 10 years.

  path: /root/create_ca.sh
  permissions: '0700'
  content: |
    #!/bin/bash
    echo "Create root CA directory"
    echo ""
    mkdir -p /root/ca
    echo ""
    echo "cd /root/ca and create directories"
    echo ""
    cd /root/ca
    mkdir newcerts certs crl private requests
    echo ""
    echo "Create index.txt to track signed certificates"
    echo ""
    touch index.txt
    echo ""
    echo "Create serial file to applied SN to sign certificates"
    echo ""
    echo '5000' > serial
    echo ""
    echo "Create CA Private Key w/ Password using AES256 encryption, and a key size of 4096-bits"
    openssl genrsa -aes256 -passout pass:s3cr3tpw1 -out /root/ca/private/cakey.pem 4096
    echo ""
    echo "Create Root CA Cert"
    echo ""
    openssl req -new -x509 -subj "/C=US/ST=MD/L=Baltimore/O=CML/CN=ubuntu-ca.cml.lab" -key /root/ca/private/cakey.pem -passin pass:s3cr3tpw1 -out cacert.pem -days 3650 -set_serial 0

- The file above is written, and executed below.  By doing it this way, we can run the bash script and automatically run through the steps above.

runcmd:
- netplan generate
- netplan apply
- apt update ; apt full-upgrade -y
- apt install openssl
- sed -i 's|./demoCA|/root/ca|' /etc/ssl/openssl.cnf
- sed -i 's/^stateOrProvinceName.*= match/stateOrProvinceName     = optional/' /etc/ssl/openssl.cnf
- sed -i 's/^organizationName.*= match/organizationName        = optional/' /etc/ssl/openssl.cnf
- /root/create_ca.sh

# How to run

This CML Lab was created with CML 2.9.1.  You can only import the YAML file if you are running 2.9 or greater.  If you are not running CML 2.9.1, you can copy and paste the cloud-config into a new Ubuntu node and let it boot.  Keep in mind, you'll need to update the IP addresses either way to have SSH access through the external connector.
