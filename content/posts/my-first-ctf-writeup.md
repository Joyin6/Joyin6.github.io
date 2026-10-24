+++ 
draft = true
date = 2026-10-24T11:33:58+01:00
title = "Traverse - TryHackMe CTF"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

I would love to show you screenshots of this one, but I was too busy being fascinated to take any, so you are getting a play-by-play instead. This is how I worked through the **Traverse** room on TryHackMe, and how one developer's pile of breadcrumbs took me from a plain landing page to an admin login.

## The attack chain at a glance

| Stage | What I did | Weakness behind it | What would have stopped it |
| --- | --- | --- | --- |
| 1. Recon | Inspected the page source and read the developer's comments, which revealed two endpoints | Sensitive information left in source code comments | Strip comments and run secret scanning before deploying |
| 2. Exposed files | Followed the endpoint to a log, then found an email dump file inside it | Security misconfiguration, with files that should never be public | Deny public access to logs and dumps in the server configuration |
| 3. Leaked secret | Pulled a key out of the dump, plus a hint about where the real API lived | Secret stored in plain text in a reachable file | Keep secrets in a vault, never in files on the web server |
| 4. API enumeration | Used the key to log in, found the real API, then tested ID after ID | Broken access control (an IDOR, where changing an ID returns someone else's data) | Check on the server, for every request, that the user may see that record |
| 5. Admin access | Logged in to the admin page with the email and password I had collected | Credentials exposed through an API response | Never return credentials, and add multi-factor authentication |

## Stage 1: Reading what the developer left behind

The room gave me a link to open in my browser, and a page loaded immediately. I did not start throwing tools at it. I opened the page source and read it like a story, and the developer had left comments in the code that mentioned two different endpoints. A browser delivers every comment to every visitor, so "hidden" notes in front-end code are only hidden from people who never right-click. I tested both endpoints.

## Stage 2: Following the chattiest one

One endpoint led to a log, and logs are usually the gossips of a system, so I went there first. Inside, I found a file containing an email dump, and I opened it. That is information disclosure in its purest form, because a file nobody meant for visitors was sitting there waiting for one.

## Stage 3: A key, and a very polite hint

The dump contained a key. It also contained a hint about where the real API lived, and the endpoint was named after the first phase of the secure software development lifecycle. I will say only that the developers were thinking about security in the wrong order, because naming the door after "planning" did not plan for me finding it. I followed the hint, found a login page, and used the key.

## Stage 4: One ID at a time

The login gave me the endpoint for the actual API. I ran it, and then I tried one ID after another and watched how the responses changed. When I reached the right ones, I had John's email, the admin's email, and the number attached to John's endpoint. The API handed over records to anyone who asked for them by number, which is the classic IDOR pattern.

<!-- ADD: what the responses looked like, or how many IDs you tried before it made sense -->

## Stage 5: Walking in through the front door

I then went to the admin login, which asked for an email and a password. I used what I had collected from the API, and I was in. No exploit, no clever payload, only a chain of small mistakes, each one making the next one easier.

## The DevSecOps takeaway

None of these weaknesses needed a sophisticated attack, and every one of them could have been caught earlier in a pipeline. Secret-scanning tools such as gitleaks can flag keys and credentials in code before it ships, automated checks can reject builds that expose comments or sensitive files, and access control can be covered by tests that confirm user A can never read user B's data. That is the part of security I want to work on, which is stopping the chain before the first link.

## What I'd do differently next time

I would take screenshots as I go, because notes written in the excitement of the moment are great, but pictures are better. Consider this my promise that the next write-up will have them.