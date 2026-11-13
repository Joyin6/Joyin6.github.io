+++ 
draft = false
date = 2026-11-13T20:08:32+01:00
title = "I Put WordPress on AWS Lightsail with a Free Domain"
description = ""
slug = ""
authors = []
tags = ['aws', 'lightsail', 'wordpress', 'https', 'lets-encrypt', 'cloud-security']
categories = []
externalLink = ""
series = []
+++

I wanted a quick win in AWS: launch a WordPress site on Lightsail, give it a free domain, and make it secure. It was a quick win with several detours, one of which involved my own VPN gaslighting me.

## What I wanted to build

A working WordPress site on a Lightsail instance, reachable on a real-looking domain instead of a bare IP address, and served over HTTPS so the browser stops calling it "Not secure". Lightsail is AWS's simpler, flat-price corner, so it is a gentler place to learn than a full EC2 setup.

## What I actually did

1. **Created the instance.** I picked the Stockholm region, chose Linux, and selected the WordPress blueprint. I ended up with an instance called WordPress-Production, which is a very grand name for a throwaway site.
2. **Attached a static IP.** A Lightsail instance gets a new public IP address if it is stopped and started, so a static IP keeps the address from changing underneath my domain.

   ![Lightsail instance page showing the Stockholm region, the attached static IP, and a running instance that passed its status checks](/images/ssh.png)
   

3. **Connected to the server.** My first attempt followed a guide that connects from your own terminal as the `bitnami` user, and the server answered with "Permission denied (publickey)". The browser-based SSH terminal in Lightsail let me straight in, as `admin`.

   ![My own terminal attempt as bitnami ending in Permission denied (publickey)](/images/connection.png)

4. **Found the default credentials and changed them straight away.** Lightsail shows the default username (`user`) and points you to a password on the server. I went looking in the Bitnami location, and it was not there.

   ![Terminal as admin showing that the bitnami_application_password file does not exist](/images/mess.png)

   Once I had the default password and logged in, I changed it immediately. Default credentials are public knowledge in the way that "password123" is public knowledge. I will not show either password here, because credentials never belong in a blog post, even from a lab.

   ![Lightsail's default credentials panel, with its advice to change the password after signing in](/images/default_credentials.png)


5. **Got a free domain from DuckDNS.** I created a subdomain there, pointed it at my static IP, and watched my site load on a proper name.

   ![DuckDNS dashboard showing the subdomain pointing at the server's IP address](/images/lightsail_wp.png)

6. **Noticed the site was not secure.** It loaded, but over plain HTTP, which means everything between the visitor and the server, including the admin login, travels unencrypted. Lightsail even says so on the same page, with a warning that the URL is unencrypted.
7. **Installed Certbot and requested a free Let's Encrypt certificate:**

   ```bash
   sudo apt update && sudo apt install -y certbot python3-certbot-apache
   sudo certbot --apache -d my-lightsail-wp.duckdns.org
   ```

8. **Checked the certificate.** The "Issued By" field says Let's Encrypt, which is exactly what I wanted to see.

## Everything that went wrong

| Problem | What was really going on | How I fixed it |
| --- | --- | --- |
| SSH as `bitnami` failed with "Permission denied (publickey)", and the Bitnami password file did not exist | The guide I followed was written for the older Bitnami WordPress setup. The newer native Lightsail blueprint uses the `admin` user and a different layout | I used the browser-based terminal, logged in as `admin`, and stopped assuming where things live |
| Certbot said it could not find a virtual host for my domain | Apache only had two default virtual hosts, and neither was named after my domain, so Certbot had to ask me which to use | I picked the default SSL one, and Certbot deployed the certificate there |
| Mixed-content warnings and a "Not secure" label, even though the certificate was installed | Certbot did its job, but WordPress was still building its own URLs with `http://` at the front | I changed the setting in `wp-config.php` from `'http://'` to `'https://'`, then restarted Apache |
| My browser showed a certificate for a completely different name (`ip-172-26-3-104`) | My VPN's threat protection was intercepting HTTPS traffic and handing my browser its own certificate, which hid the real Let's Encrypt one | I turned the VPN protection off and on again, and the correct certificate appeared |

## The confusing parts, cleared up

**Why the certificate was "wrong" when nothing was wrong with the server.** The server was serving the right certificate the whole time. Something on my own laptop was sitting in the middle and answering instead. That is exactly what a man-in-the-middle looks like, which is a funny thing to discover when the culprit is software you installed on purpose.

**How to tell the difference in ten seconds.** Open the certificate and look at "Issued By". If it says Let's Encrypt, the server is fine. If it names something else, like a VPN or antivirus, local software is interfering. 

**Why Certbot asked me to pick a virtual host.** Certbot looks for an Apache virtual host with your domain as its `ServerName`. I had not set one up, so it showed me the two defaults and let me choose.

**The red warning at the end.** Certbot added a redirect from HTTP to HTTPS and warned me to check it against the rewrite rules that were already there. It is easy to skim past, and worth reading.

**Why a free DuckDNS name still counts.** Let's Encrypt does not care whether the domain was expensive, only that you can prove you control it. For a learning project, a free subdomain is perfectly good practice.

**Why the static IP matters for HTTPS.** Certbot proves control of the domain by reaching your server through it. If the IP changes underneath the DNS record, that check breaks, and so does your site.

**Why renewal is part of the deal.** My certificate runs from November 13 to February 11, so it lasts about three months. Free certificates are short-lived on purpose.

## What I'm remembering for next time

- Check the blueprint's defaults before copying commands from a guide. Native Lightsail WordPress uses the `admin` user, keeps the credentials in the `admin` home folder, and does not store things where older Bitnami guides expect them.
- When HTTPS looks broken, open the certificate and read "Issued By" before touching the server.
- Read Certbot's closing output, including the warning about rewrite rules.
- After Certbot, check `wp-config.php` for `http://` and switch it to `https://`.
- Change default credentials immediately, and keep them out of notes that might end up in a blog post.
- Delete Lightsail instances instead of stopping them. A stopped instance is still billed at the full rate, so delete the instance and its detached static IP too, and take a snapshot first if you want to resume later.

## The takeaway

The padlock in the browser only tells you the connection is encrypted, not that the server at the other end is set up safely. Getting HTTPS working was satisfying, but the bigger lessons were smaller: check your assumptions about defaults, read the certificate before blaming the server, and remember that a stopped instance still sends a bill.

*This was a throwaway learning site in my own AWS account, with no real data on it.*
