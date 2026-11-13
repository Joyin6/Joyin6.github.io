+++ 
draft = false
date = 2026-11-13T08:54:21+01:00
title = "I Left a Door Open on Purpose, and It Took One Link to Walk Through"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'aws', 's3', 'cloud-security', 'information-disclosure']
categories = []
externalLink = ""
series = []
+++

Northgate stores the shipping manifests that staff upload in an S3 bucket, and the estate also has a second bucket for finance data. I made the manifest bucket public, which is one of the most common cloud mistakes there is, and I left the finance bucket alone. Then I stopped thinking like the person who built it and started thinking like a stranger on the internet who had found the address.

Everything in this lab runs in my own account, with fictional data only, and I tested only my own buckets.

## What I found

The manifest bucket opened with no problem at all. I did not log in, I did not guess a password, and I did not need an AWS account. The only thing I needed was the public URL of the bucket, and the bucket handed over its contents to anyone who asked.

The finance bucket was a different story. Access was explicitly denied, which is exactly what I wanted to see. It is a good reminder that AWS denies access by default, so every open door in an account is a door that somebody opened.

<!-- ADD: what you actually saw in the manifest bucket, for example whether you could list every file or only open files you already knew the names of, and the kinds of files and details inside -->

<!-- ADD: a screenshot of the browser showing the bucket contents, with the bucket name and any real-looking details covered -->

## The exposure in one table

| Question | The answer from my lab |
| --- | --- |
| What was exposed | The shipment manifests in the manifest bucket, meaning customer and shipment details |
| Who could see it | Anyone on the internet who had the public bucket URL |
| How hard was it | Barely any effort: one URL, with no login and no tools |
| What stayed safe | The finance bucket, where access was explicitly denied |
| What stopped the attacker there | Nothing about the manifest bucket, and a deliberate access rule on the finance bucket |

## Why one link is enough

A public bucket does not need to be "hacked", because it is simply open. There is no password to crack and no vulnerability to exploit, so the attacker's whole job is finding the address and asking nicely. For a real company that is the scariest part, because the people who would notice the problem are usually not the people looking at the bucket, and the attacker will find it first.

This is also why defenders should compare the two sides of the effort. The attacker needed one link and a few seconds. Fixing it takes a few lines of configuration, such as turning on S3 Block Public Access and replacing the open permissions with a rule that allows only the app's own identity. The difference is that the fix has to be remembered, while the attack only has to be noticed once.

## What a real company would lose

My manifests are fictional, but imagine they were real, with customer names, addresses, and shipment details inside.

- **Customer privacy.** An attacker would hold a list of real people and where they live, which is exactly what scammers and stalkers look for.
- **Control of deliveries.** Someone who knows what is being shipped, to whom, and when could try to intercept a delivery, redirect it, or impersonate the company to the customer.
- **Customer service.** Customers who find out that strangers had their details would flood the support team with questions, complaints, and refund requests.
- **Trust.** Freight companies sell reliability above everything else, and a leak says that the company cannot keep its own paperwork private. That is very hard to win back.
- **Legal trouble.** Depending on the country and the data, a leak of personal information can mean investigations, fines, and mandatory notices to customers.

None of these losses needed a clever attacker, which is the most uncomfortable part of the whole exercise.

## The lesson

The finance bucket stayed locked and the manifest bucket did not, and the only difference between them was a decision somebody made. Cloud security is mostly that: not exotic attacks, but small decisions about who is allowed to ask for what.

## What I learned

- A public bucket has no lock to pick, so the attacker needs only the address.
- Default deny works, and the finance bucket proved it.
- The attacker's effort and the defender's effort are very uneven, so the defender has to build the safeguards before the mistake happens.
- Customer data in the wrong bucket is not a technical problem for the business, it is a trust problem.

## What's next

Next, I will keep going through the weaknesses in my "before" environment, and then I will fix them one at a time, with evidence of each fix.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*