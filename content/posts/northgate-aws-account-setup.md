+++ 
draft = false
date = 2026-10-31T16:07:50+01:00
title = "Northgate Freight: Setting Up AWS Without Setting Myself on Fire"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'aws', 'cloud-security', 'iam']
categories = []
externalLink = ""
series = []
+++

Before I built anything for my Northgate Freight cloud security project, I wanted to get the foundation right. In cloud security, the account itself is the first thing an attacker would go after, so I treated setup as part of the project instead of a chore I could rush through with snacks in hand. This post covers what I did, the mistake I made, and why each step matters.

## The mistake that cost me a restart

My first attempt did not go the way I thought it would. I signed up through the AWS Builder ID and managed developer flow, and I assumed I was getting a normal, standalone account of my own. I was not. AWS quietly provisioned a **member account** inside a pre-configured parent AWS Organization, which meant my account lived under someone else's umbrella instead of being the independent account I wanted.

That matters in cloud security, because an organization sits above your account and can apply its own rules and controls to it. I wanted to build, break, and harden an environment that I fully controlled, and that was not possible from inside a managed one. So I created a proper standalone account of my own and redid every setting from the beginning. It was a very educational way to spend an afternoon.

The lesson is simple: **check where a sign-up path leads before you commit to it**. I didn't look closely at what the flow would create, and a few minutes of reading would have saved me the whole restart. In security work, a link, an invitation, or a sign-up page deserves the same suspicion as an email attachment from a "prince". I now confirm the address, the source, and the kind of account I'm getting before I type anything into anything.

## Step 1: Lock down the root user with MFA

When you create an AWS account, you start with a **root user**. The root user can do everything, including closing the account, and it cannot be restricted the way normal users can. That makes it the most valuable target in the whole account, which is basically a golden crown with a "please steal me" sign.

As soon as my new account existed, I turned on **multi-factor authentication (MFA)** for it. I chose an authenticator app, because it generates a short code on my phone that changes every thirty seconds, so a stolen password alone is not enough to get in. A thief would need my password and my phone, and my phone has never once left my hand since I started this project.

![MFA SetUp](/images/mfa_redacted.png)

## Step 2: Set a budget alert

Next, I created a budget alert of five dollars. Cloud costs can grow quietly, and a forgotten resource can keep charging you long after you stopped caring about it. The alert emails me when my spending approaches the limit, so I find out early and not when the bill arrives with its tiny, terrible invoice.

It covers everything in the account, including EC2 instances, which is simply the AWS name for virtual servers. For a student lab, a budget alert is one of the most important safety nets there is.

![Budget Limit Set](/images/budget.png)

## Step 3: Create a daily-use admin and stop using root

The root user should be used as rarely as possible. So I used **IAM**, AWS's identity service, to create a separate user called `northfreight_admin` for everyday work, and I gave it administrator access for now.

That is deliberately broad. This project begins with a "before" state, an environment with realistic weaknesses that I will find and fix later. In the later stages, I will replace this standing administrator with a least-privilege setup, an MFA-protected admin role, and short-lived sessions, and I will prove the change with evidence, because "trust me" is not a security control.

![Permissions Policy Set](/images/permissions_policy.png)

After that, I signed out of the root user, and I now use `northfreight_admin` for everything. The root user is back in its vault.

![IAM User](/images/iam_user.png)

## What this taught me

- Verify links and sources before using them.
- Know whether an account is standalone or part of an organization, because the parent organization can control what you do.
- Protect the root user with MFA, then avoid using it.
- Set a budget alert before building anything.
- Use a separate daily-use identity, so the most powerful account stays locked away.
- Broad permissions are acceptable only as a conscious, temporary choice, with a plan to reduce them.

## What's next

I've started working with **Terraform**, a tool that builds cloud environments from code, and my next post covers my first run. I promise to tell you honestly how it went, including the parts where it did not go as planned.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*