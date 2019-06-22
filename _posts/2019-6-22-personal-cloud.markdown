---
layout: post
title: Private Cloud using Nextcloud
date: 2019-6-22 15:00:20 +0300
description: A personal, open source cloud using AWS, Wasabi, and Nextcloud
img: private_cloud/cloud.jpg
tags: [aws, opensource, nextcloud, cloud]
author: Aaron Peterson
---
# Overview
This post will explain how I setup my completely cloud based nextcloud installation as another step forward in securing my data privacy.

Here is what I used:
1. Ubuntu 18.04 server in an [AWS Lightsail](https://aws.amazon.com/lightsail/) VM
2. [Wasabi Cloud Storage](https://wasabi.com/) as the primary object storage
3. DDNS via No-Ip for a free domain
4. HTTPS encryption via [Let's Encrypt](https://letsencrypt.org/)
5. Nextcloud's Snap package (includes all depencies and helper scripts)

# Setup AWS Lightsail instance
1. Log into [AWS Lightsail](https://aws.amazon.com/lightsail/)
2. 
2. Select Ubuntu 18.04 as the instance OS.
3. Select an appropriately sized instance. I used the smallest with 1 vCPU, 512mb ram, and 20gb storage.
4. (optional) setup a ssh key for secure remote ssh access. Run in a terminal:
    ```
    ssh-keygen -t rsa -C "your_email@example.com"
    ```
5. upload the .pub key file to lightsail via the Upload key button.
5. Once the VM is up, Select the VM and select the Networking tab.
6. Find the firewall section and add port 443 for https requests.
7. Click the 'Create static IP' button and setup a static ip for this VM. AWS lets you have 5 free static ip's.
8. Note the static IP, we'll need it later.

# Setup Wasabi
Wasabi is super easy to setup, I didn't even need a guide. It's 100% bit compliant with S3 so configuration is similar to that of a normal S3 bucket:
1. Go to your storage on the [wasabi console](console.wasabisys.com)
2. Choose 'Create Bucket'
3. give the bucket a unique name (I used: myemail-nextcloud)
3. Choose a region closest to where you live. This should match the region of your Lightsail VM.
4. I chose no versioning (read up on it to see if you need it)
5. I opted for bucket logging, and I setup a second bucket just for logs (I think this is necessary for nextcloud; it doesn't like having something else putting things in its bucket)
6. Click the side menu button and go to Access Keys
7. Click 'Create new Access Key' and save the access and secret key for later use.
8. That's it!

# Setup DDNS via No-Ip
1. Create an account on [No-Ip](https://www.noip.com/)
2. Under Quick Add, click '+ More Records'
3. Choose a domain and subdomain
4. Enter the static IP address from the lightsail VM under the IPv4 address box.
5. Click 'Create Hostname'
6. Done!


# Connect to the Lightsail Virtual Machine
You can either access the VM's console via the Lightsail web dashboard or via ssh with the private key mentioned above.

I like to use SSH so I opted for that:
```
ssh ubuntu@<static ip address>
```

# Install Nextcloud
We'll be using the Snap package for Nextcloud which includes all its dependencies and helper scripts to make installation a..... snap!

1. install Snap via terminal:
    ```
    sudo apt udpate
    sudo apt install snap
    ```
2. install Nextcloud
    ```
    sudo snap install nextcloud
    ```
###  Before going any further, we need to configure Nextcloud to use Wasabi as the primary storage. This requires some text file configuration.

[Here is the documentation](https://docs.nextcloud.com/server/stable/admin_manual/configuration_files/primary_storage.html?highlight=primary%20storage) on configuring primary storage

3. Edit the Nextcloud config file at /var/snap/nextcloud/current/nextcloud/config/config.php
    ```
    sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
    ```
4. inside the $CONFIG array, add a comma to the end of the previous array element and add this code block. Edit the fields with '' to include your own info from wasabi:
    ```
    'objectstore' => 
    array (
        'class' => '\\OC\\Files\\ObjectStore\\S3',
        'arguments' => 
        array (
        'bucket' => '',
        'autocreate' => false,
        'key' => '',
        'secret' => '',
        'hostname' => 's3.wasabisys.com',
        'use_ssl' => true,
        'region' => '',
        'use_path_style' => false,
        )
    ),

    ```
### It's apparently important to make this change before we setup the admin user. It appears that changing the primary storage once the admin account is setup can break the user profiles in Nextcloud.

5. Setup the Nextcoud admin account
    ```
    sudo nextcloud.manual-install admin <admin password of your choosing>
    ```
6. Update the whitelist of domains allowed to access the Nextcloud server:
    ```
    sudo nano /var/snap/nextcloud/current/nextcloud/config/config.php
    ```
    In config.php, find the section for *trusted_domains* and add the IP address of your instance and the domain (whatever you setup in No-Ip)
    ```
    'trusted_domains' => array (0 => 'localhost', 1 => '111.122.133.144', 2 => 'mycloud.example.com',),
    ```
7. Restart the Nextcloud snap with:
    ```
    sudo snap restart nextcloud
    ```
8. Open a web browser and visit your ddns domain. You should see the login page for Nextcloud.

# Setup HTTPS
Setting up HTTPS inolves two steps: configure Nextcloud to listen on port 443 (https port) and creating a CA-signed certificat for your domain. This is all handled for us in one of Nextclouds handy helper scripts!

1. Log into your lightsail VM via ssh (or web console), type the following command and follow the instructions:
    ```
    sudo nextcloud.enable-https lets-encrypt
    ```
    for the domain, I put in the full subdomain.domain from No-IP
2. To confirm that HTTPS works on our server, go to "https://\<your no-ip domain> and look for the green padlock to the left of your URL.

# Keeping your VM updated
Thanks to the technology chosen, we don't need to worry about keeping the server updated, it will do it all by itself

* Ubuntu server now comes with a package called [unattended-upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html) which will keep the server updated automatically. It's enabled by default on Ubuntu.

* Snapd [checks a couple times a day](https://docs.snapcraft.io/keeping-snaps-up-to-date) for updates to the installed snap packages and updates them as necessary.