+++ 
draft = false
date = 2026-11-02T09:46:30+01:00
title = "I Built My First Cloud Server with Terraform, and Everything That Went Wrong First"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'terraform', 'aws', 'cloud-security']
categories = []
externalLink = ""
series = []
+++

This week I moved from reading Terraform code to running it for real, which is a bit like going from reading about swimming to being thrown into the deep end with a very helpful lifeguard called `terraform plan`. For my Northgate Freight cloud security project, I used Terraform to build a small AWS environment: a server, a firewall, a storage bucket, and an identity for the app. By the end I was logged in to a server in Ireland that I had created from a text file. It did not go smoothly, and the mistakes taught me more than the successes did.

## The workflow I followed

Every build followed the same sequence. I checked that I was signed in to AWS, ran `terraform init` to download what Terraform needs, ran `terraform fmt` and `terraform validate` to tidy and check the code, and ran `terraform plan` to preview exactly what would be created. Only then did I run `terraform apply` to build it. The plan step is the most important habit I've learned so far, because it shows what will happen before anything is changed.

![Terraform init downloading the AWS provider and finishing successfully](/images/terraform-aws-init.png)

![Terraform fmt and validate confirming the configuration is valid](/images/terraform-aws-validate.png)

## Mistake 1: Terraform asked me for a value I never gave it

My first `terraform plan` stopped politely and asked me to enter a value for `my_ip_cidr`. I had skipped creating my settings file. That value is my own IP address, and the firewall uses it to let only me reach the server, while everyone else on the internet is blocked. Once I created the `terraform.tfvars` file, copied in my address, and added `/32` (which means exactly one address), the prompt disappeared. I also liked that Terraform checks the value is a single address, so a typing mistake can't accidentally open the server to the entire internet.

![Terraform plan stopping to ask for the my_ip_cidr value](/images/terraform-aws-plan-prompt.png)

## Mistake 2: The SSH key didn't exist

The next plan failed with an error saying that no file existed at the path where Terraform expected my public key. I had skipped the step of creating an SSH key pair. SSH keys work like a lock and key: the server holds the public half, and only my private half, which stays on my laptop, can open it. I created the pair with `ssh-keygen` (and typed my passphrase wrong the first time, which the tool noticed with admirable calm), and the error went away. The most important rule is that the file without `.pub` is private and must never be shared or uploaded anywhere.

![Generating an SSH key pair with ssh-keygen](/images/terraform-aws-sshkey.png)

## Mistake 3: Expired sessions and a shaky connection

My login session expired partway through, and later my connection to the AWS console wouldn't load. That taught me something useful: `terraform state list` worked even without internet, because it only reads a local file. Seeing what Terraform remembers is not the same as proving that my connection to AWS works, so I checked that separately with `aws sts get-caller-identity`.

## The result

After fixing those problems, I ran `terraform apply`, typed `yes`, and Terraform built the environment in about a minute: twelve resources added, none changed, none destroyed. I then logged in to the new server over SSH and was greeted by the Ubuntu welcome screen. Seeing a prompt on a computer I had created from a text file, thousands of kilometres away, made the whole idea click.

![Terraform apply finishing with twelve resources added](/images/terraform-aws-apply.png)

![The Ubuntu welcome screen after logging in to the new server over SSH](/images/terraform-aws-welcome.png)

Once I was in, I checked that the server's startup script had finished, and it had. The Northgate service is installed but not running yet, which is exactly what I expected, because I have not deployed the app.

![Checking the startup script and the not-yet-running Northgate service](/images/terraform-aws-cloudinit.png)

## What I learned

- Read the plan every time, and never apply something I don't understand.
- Error messages are clues, so I read the file name and line number first.
- Keep private keys private, and keep my IP address and account details out of anything public, including screenshots.
- A tool working locally doesn't prove my connection to the cloud works.
- Set up the small things first, such as a settings file, keys, and the right profile, because most failures came from missing prerequisites.

## What's next

Next, I'll deploy the Northgate Freight app to this server, then deliberately weaken the setup and secure it again. I destroy the environment at the end of each session, so I'm not paying for forgotten resources.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*