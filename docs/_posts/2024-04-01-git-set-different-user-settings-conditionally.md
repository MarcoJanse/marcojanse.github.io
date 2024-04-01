---
layout: post
title:  Set different user settings conditionally in Git
categories: git
---

![Git logo](../assets/images/post_2024-04-01_git-logo2.png){:width="50%"}

## Introduction

As you probably know, when using git you have to set your user name and email address before you can do any commits.

- You can use `git config user.name "John Doe"` and `git config user.email "john.doe@example.com"` to set these for a specific repository
- You can use `git config --global user.name "John Doe"` and `git config --global user.email "john.doe@example.com"` to set these values for all repositories.

But what if you want to use different values depending on the customer/folder?

I use git both personally, using my own private Github email address, but also for work, where I mostly use my work e-mail address for commits to Azure DevOps. On top of that, I work for different customers where I usually end up with a dedicated user account in the customer's tenant.

I want an easy way to sign my private commits using my private email address, work-related with my UPN and customer-specific with the user principal name of my account from that customer.
Here's how I did that:

## Use an easy folder structure

These steps work best if you have different root directories for work related git repositories and private/public git repositories.
In my case, I only use GitHub privately and Azure DevOps for work-related repo's, so that's easy.

Therefore, my Git folder structure looks like this on a Windows laptop:

- C:\Git
  - AzDevOps
    - CustomerA
      - Repo1
      - Repo2
    - CostomerB
      - Repo1
  - GitHub
    - MarcoJanse
      - Repo1
      - Repo2
      - Repo3

## Set-up

Because I primarily use non-work related git repositories, I use my GitHub user name and email address in my global config.

### Basic commands

Some useful commands for viewing and setting config

- To view all config settings from a git repository
  - `git config --list`
- To view settings defined in your global config
  - `git config --global --list`
- To show where all the configuration is coming from in a git repository:
  - `git config --list --show-origin`
- To set your global user name
  - `git config --global user.name John Doe`
- To set your global email address
  - `git config --global user-email john.doe@privatemail.com`
- To view your configured email adres from within a git repository
  - `git config --get user.email`

### Creating the conditional includes file

First, I create a file called `.gitconfig_work` and store this in my documents folder for safe keeping.
You can name it whatever you want, but make sure it starts with a dot `(.)`.
I created a Git folder in my OneDrive folder for this.

In there, put the settings that apply only for your work-related repos.

```bash
[user]
	name = John Doe
	email = john.doe@company.com
```

### Add the conditional include to your global git configuration

Edit your global git config file via a code editor like VSCode. Mine is stored in `C:\Users\<username>\.gitconfig`

Add the include at the end, to make sure the settings don't get overwritten again later in the same file.

```bash
[user]
    name = John Doe
    email = john.doe@company.com
    signingkey = FAKESIGNINGKEY001
[gpg]
    program = c:/Program Files (x86)/GnuPG/bin/gpg.exe
[commit]
    gpgsign = true
[includeIf "gitdir/i:C:/Git/AzDevOps/"]
    path = ~/OneDrive - CompanyName/Documents/Git/.gitconfig_work
```

### Test the settings

- Open a new shell and test your settings from within a git repository in AzDevOps folder:
  - `git config --get user.email`

Depending on the folder and the level your're in, you should see different value.

## Closing notes

Hopefully this article helped you make git a little easier.