---
layout: default
title: Tailscale
parent: + Raspberry Pi
grand_parent: Bonus Section
nav_exclude: true
has_toc: false
---

# Bonus guide: Tailscale
{: .no_toc }

Difficulty: Intermediate
{: .label .label-yellow }

Status: Tested v3 
{: .label .label-green }

We set up [Tailscale](https://tailscale.com/download/linux/rpi-bullseye){:target="blank"}, tailscale lets you easily manage access to private resources, quickly SSH into devices on your network, and work securely from anywhere in the world.

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### Introduction
[Tailscale](https://tailscale.com) tailscale lets you easily manage access to private resources, quickly SSH into devices on your network, and work securely from anywhere in the world.

---

## Install Tailscale
* Add Tailscale’s package signing key and repository

  ```sh
  $ curl -fsSL https://pkgs.tailscale.com/stable/raspbian/bullseye.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg > /dev/null
  $ curl -fsSL https://pkgs.tailscale.com/stable/raspbian/bullseye.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
  ```

* Install Tailscale

  ```sh
  $ sudo apt-get update
  $ sudo apt-get install tailscale
  ```

* Connect your machine to your Tailscale network and authenticate in your browser

  ```sh
  $ sudo tailscale up
  ```

* You’re connected! You can find your Tailscale IPv4 address by running

  ```sh
  $ tailscale ip -4
  ```

### Access your node

* To access the services installed in your machine you normally use your local ip (https://192.168.1.10) or your machine hostname (https://raspibolt.local)

* With Tailscale you can use the external ip from the tailscale vpn or the hostname of your machine

  ```sh
  $ tailscale ip -4
  ```

## Ride The Lightning

* First install [Ride The Lightning](https://raspibolt.org/guide/lightning/web-app.html) if it's not installed

* To access RTL instead of using your local ip (eg https://raspibolt.local or https://192.168.1.10) you would use the ip you got from the command above (eg https://100.100.100.100:4001 or https://raspibolt:4001)

## Zeus

* Follow the [Mobile App Guide](https://raspibolt.org/guide/lightning/mobile-app.html) until the "Create a lndconnect QR code" section

* Add this line in your lnd.conf in the Application options section to allow rest access from anywhere

  ```sh
  $ restlisten=0.0.0.0:8080
  ``` 

* Configure the firewall

  ```sh
  $ sudo ufw allow 8080/tcp comment "allow LND REST from anywhere"
  ``` 

* Create a lndconnect QR code. Make sure to replace the .onion address with the ip from tailscale.
  
  ```sh
  $ lndconnect --host=abcdefg..............xyz.onion --port=8080
  ``` 
 
* Open Zeus and tap on “GET STARTED”

* Tap on “Connect a node” and then tap on the “+” at the top right to add your node

* Enter a Nickname for your node (e.g., “RaspiBolt”)

* Click on “SCAN LNDCONNECT CONFIG” and, if prompted, allow Zeus to use the camera

* Scan the QR code generated earlier

* Click on “SAVE NODE CONFIG”. Zeus is now connecting to your node, and it might take a while the first time.

### For the future: Tailscale upgrade

* As “admin” user, run:

  ```sh
  $ sudo apt-get update
  $ sudo apt-get install tailscale
  ```

### Uninstall Tailscale

* Uninstall tailscale package

  ```sh
  $ sudo apt-get remove tailscale
  ```

* Rename the certificate in nginx.conf

  ```sh
  $ sudo nano /etc/nginx/nginx.conf
  ```

  ```sh
  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
  ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
  ```

---

## Extras

### Configure dns and certificate

* Sign in on tailscale.com and go to the DNS settings in tailscale admin section [panel](https://login.tailscale.com/admin/dns){:target="_blank"}

* Enable Magic DNS and HTTPS Certificates
  
  ![tailscale_magic_dns](../../../images/tailscale_magic_dns.png)

  ![tailscale_https](../../../images/tailscale_https.png)

* Generate your ssl certificate
  
  ```sh
  $ sudo tailscale cert machine-name.your-tailscale-domain.ts.net
  ```

* Move the certificates to /etc/ssl folder
  
  ```sh
  $ sudo mv machine-name.your-tailscale-domain.ts.net.key /etc/ssl/private/machine-name.your-tailscale-domain.ts.net.key
  $ sudo mv machine-name.your-tailscale-domain.ts.net.crt /etc/ssl/certs/machine-name.your-tailscale-domain.ts.net.crt
  ```

* Edit your nginx config
  
  ```sh
  $ sudo nano /etc/nginx/nginx.conf
  ```

* Replace nginx-selfsigned .crt and .key with machine-name.your-tailscale-domain.ts.net

  ```sh
  ssl_certificate /etc/ssl/certs/machine-name.your-tailscale-domain.ts.net.crt;
  ssl_certificate_key /etc/ssl/private/machine-name.your-tailscale-domain.ts.net.key;
  ```