+++ 
draft = false
date = 2026-10-26T12:56:11+01:00
title = "Meet the Northgate Tracker: The App I'm About to Put in Danger"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'cloud-security']
categories = []
externalLink = ""
series = []
+++

Before you can break something, you should meet it. It's only polite.

In the intro post I promised that the next post would cover the first real thing I did. It turns out the first real thing was designing a small web app for a freight company that doesn't exist. Nobody asked for it, and I'm still rather attached to it. This is its introduction, because every later post (cloud, detection, pentesting, compliance) leans on it.

## A quick word on how it was built

I'm a security person, not a Flask developer, so let's get this out of the way: **I designed the Northgate tracker and built it with AI assistance**. I decided what it should do, what it should and shouldn't protect against, and which gaps to leave open on purpose. The AI wrote most of the code. I reviewed it, tested it, and made sure I could explain every security decision in it.

That last part is the point. The app is the target, not the showpiece. And since AI-generated code can ship plausible-looking flaws, reviewing it is a security exercise in its own right. It may well earn its own post later.

## The app at a glance

The tracker is a small Flask app in Python with two halves. The public half lets anyone look up a shipment, and the staff half sits behind a login. Underneath, it quietly carries a considerable amount of security thinking: parameterised database queries, a CSRF token on every form, hashed passwords, a login that gives nothing away, strict rules for uploads, automatic output escaping in the templates, security headers on every response, careful logging, and a refusal to start in production with its placeholder secret key. Each of those exists to make a specific attack harder, and I'll point out which as we go.

![Northgate Homepage](/joyce-sec/static/images/northgate_homepage.png)

## Part 1: The public half

Anyone can type in a tracking code and see a shipment's route and status. It shows no customer names and no documents. The query behind it selects only five columns and never touches the customer's name, because a page can't leak what it was never given. Nosy visitors get nothing.

## Part 2: The staff half

Behind a login, dispatchers can register shipments, update their status, upload a manifest for each one, and download it again.

![Northgate Staff Page](/joyce-sec/static/images/northgate_staffpage.png)

## How a request travels through it

Every request takes the same short trip: the browser asks, some automatic checks run, a route decides what to do, the database or file storage gets involved, and a template turns the result into a page. The checks happen before the route, which matters more than it sounds. A bouncer who checks IDs after you're inside isn't doing much.

## The defences I'm quietly proud of

The app isn't a blank target. I want to know exactly which defences it ships with, because later I'll have to tell an app weakness from a cloud weakness. Four highlights:

### 1. Parameterised queries

The app never glues user input into a database command. It sends the command and the value separately, so a "tracking code" like ' OR '1'='1 is treated as a very strange code, not as instructions. This is what stops SQL injection.

### 2. A CSRF token on every form

Imagine a dispatcher logged in to Northgate who opens a sketchy tab that quietly submits a form to the site. The browser helpfully attaches their login cookie, so the request looks genuine. The defence is a hidden random token that a malicious page can't read, and a check that rejects any form without it.

### 3. The dummy hash on login

If a username doesn't exist, the app checks the password against a fake hash anyway. Otherwise, real usernames would answer slightly slower than invented ones, because only they trigger the slow password check, and an attacker could time the difference to build a list of valid accounts. The app also gives the same vague error either way, and starts a fresh session at login to prevent session fixation.

![Failed Login](/joyce-sec/static/images/northgate_incorrect_login.png)

### 4. Random names for uploaded files

Uploads are one of the riskiest features on any site. The manifest route checks the extension against an allowlist (.pdf, .txt, .csv), caps the size at 2 MB, and saves the file under a random name the app generates. The filename the user chose lives only in the database, for display, so a hostile filename never gets to decide where anything lands on disk.

## What it deliberately doesn't do

I left gaps in on purpose, because they're realistic, and because the cloud setup around the app is supposed to answer for them. There is no limit on login attempts, so an attacker can guess passwords all day without being slowed or locked out. There is no MFA for staff, so a stolen password is enough to get in. There is no encryption in transit, because the app speaks plain HTTP, which means anyone on the network path can read the traffic, passwords included. There are no backups, so losing the disk means losing the database. And the logs stay on the server, so anyone who takes it over can tidy up after themselves.

Some of these the cloud can fix, and some it can't. Working out which is which is the whole story I'm going to tell.

## The takeaway

A secure app can still live in an insecure setup, and an insecure app can sometimes be rescued by a good one. Knowing which problems belong to the code and which belong to the cloud around it is the skill this series is built to practise. Step one was knowing what the app already gets right, and what it was designed to get wrong.

## What's next

The app works on my machine, which, as every engineer knows, is the least reassuring sentence in computing. Next up is getting it into AWS, where I'll find out what happens when a well-behaved app meets a messy cloud estate.