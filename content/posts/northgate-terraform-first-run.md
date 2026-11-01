+++ 
draft = false
date = 2026-11-01T10:22:26+01:00
title = "Northgate Freight: I Taught a Computer to Name a Turtle (My First Terraform Run)"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'terraform', 'infrastructure-as-code', 'cloud-security']
categories = []
externalLink = ""
series = []
+++

Until this week, I had only ever built things by clicking through a console, which is a bit like learning to cook by pointing at the fridge. For my Northgate Freight cloud security project, I needed to learn **Terraform**, a tool that lets you describe infrastructure in text files and have a program build it for you. This post is about my first run, and about the moment it finally made sense.

## What Terraform is, in plain words

Terraform is like a recipe that a cloud provider follows. I write down what I want, and Terraform works out how to create it. Because the recipe is a file, I can rebuild the same thing again, or remove all of it, with a single command, and the kitchen never complains.

That repeatability is the reason security teams like it. If an environment is code, it can be reviewed, version-controlled, and scanned for mistakes **before** anything is built, which is a much nicer time to discover a mistake than after it is live.

## The practice project

Before building anything in AWS, I started with a tiny practice project that creates nothing in the cloud. It generates a random two-word name, something like "heroic-anteater". That sounds silly, and it is, but it let me learn the whole workflow with no risk and no cost, and now I am on first-name terms with an anteater.

## The four commands

1. **`terraform init`** prepares the folder and downloads the plugin Terraform needs. Think of it as setting up your kitchen before you cook. Mine ended with a cheerful green message saying Terraform had been successfully initialized.

![Terminal output of terraform init finishing with a green success message](/images/terraform-init.png)

2. **`terraform plan`** shows exactly what Terraform intends to do, without doing it. My plan said that one resource would be added. This is the most important habit I picked up: you read the plan **before** you apply anything.

3. **`terraform apply`** carries out the plan, after you type "yes". Mine finished with "Apply complete! Resources: 1 added", and printed the random name.

![Terminal output of terraform apply showing one resource added and the generated name](/images/terraform-apply.png)

4. **`terraform destroy`** removes everything Terraform created. This matters in cloud security, because forgotten resources cost money and can quietly become security risks.

![Terminal output of terraform destroy confirming one resource was destroyed](/images/terraform-destroy.png)

## The plan that taught me to read plans

Later, I changed the name length from two words to three, just to see what would happen. I expected a tiny edit, and Terraform politely informed me that the change "forces replacement", which means it would destroy the old resource and create a new one. Nothing was harmed, because it was only a pet name, but it was a very clear lesson. A small change in a file can mean a big action in the real world, and the plan is where you find out.

![Terraform plan output showing a change that forces replacement of the resource](/images/terraform-plan-replace.png)

## What confused me

When I moved on to the real Northgate project, I opened a folder with several `.tf` files and felt overwhelmed, like walking into a library where every book was called "main".

What helped was realizing that Terraform reads **every `.tf` file in a folder as one big recipe**, and that the files are only split up to keep things tidy. Each file is built from the same few blocks:

- **Resources** are the things to create.
- **Variables** are settings I can change.
- **Data sources** are things that already exist.
- **Outputs** are facts printed at the end.

Once I could recognize those four, no file looked scary anymore.

## What I learned

- Always read the plan before applying it.
- Always destroy lab resources when I finish, so nothing is left running.
- Infrastructure as code turns a confusing console into something I can read, review, and repeat.

## What's next

Next, I'll use Terraform to build the Northgate Freight estate in AWS, which means a server, a firewall, a storage bucket, and an identity for the app. Then I will deliberately break it and secure it, step by step, and I will tell you honestly about every wrong turn.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*