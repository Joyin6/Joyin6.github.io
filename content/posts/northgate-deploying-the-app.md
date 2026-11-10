+++ 
draft = false
date = 2026-11-10T09:58:02+01:00
title = "I Deployed My App to a Server I Built from Code (and My Own Firewall Locked Me Out)"
description = ""
slug = ""
authors = []
tags = ['northgate-freight', 'terraform', 'aws', 'deployment', 'devsecops']
categories = []
externalLink = ""
series = []
+++

After a break from Northgate Freight, I came back to the project the way most people return to the gym: confident, slightly rusty, and immediately humbled by the equipment. 

## Where I left off

The estate was gone. That is not a bug, it is the rule I set for myself, because I destroy everything at the end of every sitting so nothing sits there quietly charging me. It did mean that my first `terraform output` printed absolutely nothing, which is the infrastructure equivalent of knocking on a door and discovering the house has been demolished.

## What I actually did

Once I stopped panicking, the day went like this:

1. **Signed in to AWS** as my IAM user, with MFA, and confirmed who I was with `aws sts get-caller-identity`.
2. **Rebuilt the estate** with `terraform plan` and `terraform apply`, which brought back all twelve resources.
3. **Packaged the app** with `git archive`, which bundles the code into a single zip file without dragging along any junk.
4. **Copied it to the server** with `scp`.
5. **Logged in over SSH** and ran `sudo northgate-deploy`, which sets the app up on the machine.
6. **Created a staff account** and **loaded the demo shipments**, so the tracker had something to track.
7. **Checked who the app runs as.** I confirmed that gunicorn, the program that serves the app, runs as the `northgate` user and not as root. This matters, because if a web server runs as root, then any bug in it becomes a root-level problem. A small account means a small blast radius.
8. **Logged in to the live app from my browser**, which was the moment I felt like a real engineer for about ninety seconds.

## Everything that went wrong

| Problem | What was really going on | How I fixed it |
| --- | --- | --- |
| A "reauthenticate" message from `aws sts get-caller-identity` | My AWS login session had expired during the break | I signed in again with `aws login --profile northfreight` |
| I was signing in as root | Root is the all-powerful owner account and should not be used for daily work (I know, I know, I wrote a whole post about this) | I signed in as my IAM user `northfreight_admin` with MFA instead |
| `terraform output` showed nothing | The estate had been destroyed previously, exactly as planned | I ran `terraform plan` and `terraform apply` to rebuild all twelve resources |
| `scp` and the port test timed out | The firewall only allowed one IP address, and it no longer matched my laptop's address | I wrote my current address into `terraform.tfvars` and applied again, and the test then showed `TcpTestSucceeded : True` |
| `northgate-app.zip` was not found | I had never created the zip, which is a difficult file to copy when it does not exist | I ran `git archive` from inside the app folder |
| `scp` printed its usage text and "Connection closed" | My `$server` variable was empty, so the command had no address in it | I typed the server's IP address directly |
| The passphrase seemed impossible to paste | Passphrase prompts show nothing at all while you type or paste, which feels exactly like nothing is happening | I pasted it anyway and pressed Enter, and the copy went through at 100% |

## The confusing parts, cleared up

**Two names, two meanings.** `northfreight` is my AWS account and command-line profile, so it goes in `aws` commands. `northgate-lab` is the lab itself, so it shows up in my SSH key's file name and my folder names. My guide used `northgate-lab` as the profile name, which is what caused the mix-up, and I now keep the two names separate on purpose.

**Why ping fails, and why that is fine.** The firewall only opens ports 22 and 8000, so a ping is always blocked. The line that really matters is `TcpTestSucceeded`, which tells me that the port I care about is reachable.

**What "current address" means.** It is my public IP, the number the internet sees me coming from. My provider can change it between sittings, and the firewall rule has to match it, written with `/32` on the end to mean exactly one address. Honestly, being locked out by my own firewall was the control working perfectly, just against the wrong person.

**Why the plan said "destroy" for the firewall rules.** Changing an address replaces the rules, so that is expected. The real warning sign would have been a plan that wanted to replace the server itself (`aws_instance.app`), and I would have stopped immediately if I had seen one.

**What the `ps` output means.** `northga+` is just `northgate` shortened by `ps` so it fits in its column. The three gunicorn lines are one parent process and two workers, which is exactly what I wanted to see.

## What I'm remembering for next time

- Start every sitting with the sign-in check, using the profile `northfreight`.
- Refresh the firewall address in `terraform.tfvars` before I apply, because my IP can change between sittings.
- Read the plan every time, and stop if the server itself is going to be replaced or destroyed.
- Run each command from the right folder: `git` in the app folder, and `terraform` in the infrastructure folder.
- Keep my passphrase somewhere safe and never inside a guide, because a lost passphrase cannot be recovered.
- Destroy the estate at the end of every sitting.

## What's next

The app is live, the estate is built from code, and everything works, which means it is finally time for the fun part: deliberately weakening the setup, finding the holes, and fixing them step by step with evidence.

*Northgate Freight is a fictional company, and everything in this project runs only in my own lab account.*