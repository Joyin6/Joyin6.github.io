+++ 
draft = false
date = 2026-11-10T10:10:41+01:00
title = "I Used My Own App, Proved Where Its Data Lives, and Found a Weakness on Purpose"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'aws', 's3', 'devsecops']
categories = []
externalLink = ""
series = []
+++

With the server finally running, I did something I had been looking forward to for weeks, which was to use my own app like an ordinary user and then poke at it like a suspicious one. This post covers what I tested, how I proved where the app really stores its data, and the first weakness I found, which I found on purpose and which my guide fixes in Part 4.

## Using the app like a customer, then like staff

From my browser, I opened the public tracking page, logged in through the staff login, and uploaded and downloaded a manifest. Everything worked, which is a deeply suspicious feeling after the week I'd had.

![Logging In as Staff](/images/staff-login.png)

## Proving the upload is not on the server

The upload is meant to be stored in an S3 bucket and not on the server's own disk, and in security you do not take "meant to be" for an answer. First I checked in the AWS console, where I could see my file sitting in the bucket. Then I checked from the command line, because the console is a screenshot and the terminal is evidence.

The listing command needs the real bucket name, which has a random ending on it. I was not sure how to run it at first, and the fix was to save the name in a variable so I never had to type it:

`$bucket = terraform output -raw bucket_name`

After that, this worked straight away:

`aws s3 ls s3://$bucket/ --recursive`

There was my manifest, in S3, exactly where it should be. It matters because a server can be destroyed and replaced at any moment, so anything that lives only on its disk disappears with it.

![Manifest Uploaded On s3](/images/s3-manifests.png)

## Reading the app's security log

I also read the app's security log, which records things like logins and what happened around them. A log is only useful if someone reads it, so I am trying to make that a habit and not a decoration. 

## The weakness I found on purpose

From the server, I queried the **instance metadata service**, a service that every AWS server can talk to for information about itself. It answered me without asking for a token. That is the old setting, called IMDSv1, and it is weak because it makes it easier for an attacker who tricks an app into making requests to reach that information. 

## When my own IP address betrayed me

I also looked again at how a changed IP address breaks access, and how to fix it. My firewall only trusts one address, so when my public IP changed, my connection stopped working. I compared the address in the firewall rule with my current public address, updated `my_ip_cidr`, and applied again. I had first run into this during the deployment, which is why I recognised it immediately this time.

## Mistakes and how I figured them out

- **I did not know how to run the bucket listing,** because the command needs the real bucket name with its random ending. Saving it in a variable with `terraform output -raw bucket_name` fixed that.
- **My changed IP address stopped the connection.** I compared the address in the firewall rule with my current public address, updated `my_ip_cidr`, and applied again.

## What I learned

- Prove where data lives, with the console first and the command line second.
- Saving a long name in a variable is kinder to my fingers than typing it.
- A weakness I have seen myself is much easier to explain than one I have only read about.
- A firewall locking me out is a sign that it works, as long as I know how to update it.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*