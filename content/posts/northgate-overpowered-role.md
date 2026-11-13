+++ 
draft = false
date = 2026-11-13T08:54:28+01:00
title = "The Role That Could Do Everything: How One Overpowered IAM Role Handed Over the Keys"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'aws', 'iam', 'least-privilege', 'cloud-security']
categories = []
externalLink = ""
series = []
+++

In my last post, a stranger needed one link to read my manifest bucket, and the finance bucket stayed locked. I felt a little smug about that locked bucket for roughly one afternoon. Then I tried another attack, and it taught me that a locked door means nothing if the person already inside has a master key. This post is about an overpowered role, and about how quickly one broad permission turned a single compromised server into a compromised account.

## The starting assumption

This attack begins after the break-in. I assumed the attacker had already managed to SSH into the app server, perhaps through a stolen key or a flaw in the app, and I wanted to know the answer to one question: what happens next? Security teams call this "assume breach", and it is a healthier way to think than "nobody will ever get in", because sooner or later somebody does.

Everything below happened in my own lab account, with fictional data only.

## Step 1: Ask the server who it is

The first thing the attacker can do is ask AWS who they are. From the server, a single identity check (`aws sts get-caller-identity`) answers with the name of the role the server is running as. It sounds harmless, and for a defender it is the first clue worth watching, because an attacker who has just landed always wants to learn what the machine they are on is allowed to do.

The AWS command line was not installed on the server, so the attacker simply installed it. A fresh install on a production server is a very loud thing to do, and nobody was listening in my lab.

## Step 2: Find out what the role can do

A few more commands later, I had the answer, and it was alarming. The role attached to my server had been given a broad S3 policy, far more than a manifest app needs. Because of that policy, the attacker could now do things that the public attack could never do.

<!-- ADD: the exact policy you used for the overpowered role, for example whether it was all S3 actions on all buckets, so readers can see how broad "broad" was -->

| What the attacker did | Why it worked | What a real company would lose |
| --- | --- | --- |
| Read the payroll data they could not touch before | The role had access to every bucket, not only the manifest bucket | Employee salaries and personal details, which is a privacy breach and a morale disaster |
| Edited the manifests | The role could write to the bucket, not just read from it | Customers and shipments with the wrong details, which means misdelivered goods and records nobody can trust |
| Deleted the manifests | The role could delete as well as write | Lost shipment records, a stalled business, and a very bad week |
| Changed bucket policies and public access settings | An S3 policy that broad includes permission to change the bucket's own rules | The attacker can make any bucket public, so the breach becomes everyone's problem |
| Obtained the role's temporary credentials | The server receives credentials for its role, and anyone on the server can ask for them | A way to log in as the role from the attacker's own computer, long after leaving my server |

## The part that changed how I see roles

The finance bucket was denied to the anonymous visitor in my last attack, so I assumed it was protected. It was protected from strangers, but it was not protected from the role. That is an important difference, because a bucket's permissions answer the question "who is allowed in?", and my role was on the guest list for every bucket in the account.

The last row of the table is the one that worries me most. The credentials for the role come as three pieces: an access key ID, a secret access key, and a session token. Together they work like a username and password for the role, and they can be used from anywhere, not only from the server. I used my lab to see how easy it is to reach them, and I will not publish any of them, because credentials never belong in a blog post, even from a lab. Earlier in this project, I found that the instance metadata service answered without asking for a token, and this attack shows why that mattered, because that service is where a server's role credentials live.

<!-- ADD: a screenshot of the identity check output, with the account number and role details covered -->

## How this maps to real attacks

This chain is not exotic, and it lines up with well-known attacker techniques in the MITRE ATT&CK framework: stealing credentials from the cloud instance metadata API (T1552.005), reading data from cloud storage (T1530), and destroying data (T1485). Real incidents have followed the same pattern of one compromised server and one overpowered role leading to a bigger breach.

## What would have stopped it

Every row in that table has a fix, and none of them is clever. They are all the same idea, which is to give the role only what the app needs.

- **Least privilege.** The manifest app should be allowed to read and write objects in the manifest bucket, and nothing else. No payroll, no other buckets, and no permission to delete.
- **No permission to change the rules.** The role should never be allowed to edit bucket policies or public access settings, because those actions decide who else gets access.
- **Block Public Access at the account level.** Even if a role is compromised, a bucket cannot be made public while this setting is on.
- **IMDSv2.** Requiring a token for the metadata service makes it much harder to pull a role's credentials from the server.
- **Protect the data itself.** Turning on versioning for the manifest bucket means an edited or deleted manifest can be recovered.
- **Watch for the strange moments.** Logging and alerts should notice a role's credentials being used from somewhere that is not the server, or a bucket policy suddenly changing.

## What I learned

- A compromised server inherits whatever power its role has, so the role is the real blast radius.
- A locked bucket protects against strangers, but it does not protect against an overpowered identity.
- Permission to change policies is permission to change the whole security model.
- The credentials a server receives can leave the server, so the server is only as safe as its role is small.
- Assume breach, then ask what the attacker could do with the foothold.

## What's next

Next, I will start fixing my "before" environment one weakness at a time, beginning with the role, and I will show the same attack failing afterwards.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*