+++ 
draft = false
date = 2026-10-24T12:42:17+01:00
title = "Northgate Freight: Meet the Company I Built to Break"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'devsecops', 'cloud-security']
categories = []
externalLink = ""
series = []
+++

Every security person eventually hits the same wall, which is that you cannot practice on real companies without a lawyer getting involved. So I did the sensible thing and built my own. Meet **Northgate Freight**, a fictional freight company that exists for one purpose, which is to be attacked, defended, hardened, and written about in public by me.

## Why a fictional company?

Tutorials teach tools, but jobs teach context. A real security team does not wonder what a vulnerability scanner does. It wonders what to fix first in a messy environment with real users, real deadlines, and a CEO who wants everything yesterday. Northgate gives me that context, with a place where I can make mistakes safely and then document what they taught me.

<!-- ADD: one or two lines about what Northgate does as a company, such as the routes it covers or its customers -->

## Northgate at a glance

| Area | What it is |
| --- | --- |
| The company | A fictional freight and shipping business |
| The application | A small shipping app that I built and launched, and which works |
| The cloud estate | A small environment I plan to host on the AWS free tier and define with Terraform |
| The goal | Build it, break it on purpose, harden it, and write down every step |

<!-- ADD: your departments or teams, the network layout, and anything else about the company's structure -->

## The infrastructure plan

The first build is a **cloud security hardening lab**. The plan is to deploy a small estate on AWS, then break it on purpose with the kind of mistakes real companies make, such as public storage buckets and identities with far more permissions than they need. After that, I will fix it by applying least privilege, removing standing admin access, enforcing multi-factor authentication, and adding logging and encryption.

I'm also considering a second lab, a **detection lab** with Windows and Linux machines, a SIEM, and log shipping, where I generate activity, write detections, and document the investigations.

## What the series will cover

Over time, Northgate will grow to cover areas such as identity and access management, governance and compliance, security monitoring, vulnerability management, networking, remote workers, incident response, data security, and AI security. I am not promising a perfect order, because plans change the moment real work starts, and I will explain those changes as they happen.

## How to follow along

Each step I take will be its own post on the [Blog](/posts/), and all of them carry the `northgate-freight` tag, so you can find the whole story at once on the [Northgate Freight series](/tags/northgate-freight/) page. The next post covers the first real thing I did, whether that turns out to be something I built or something I broke.

## A promise about honesty

I am at the beginning of this, and some things will go wrong. I will write about those moments as well as the wins, because a portfolio that only shows success teaches nobody anything.