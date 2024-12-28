---
title: "Managing Multiple GitHub Accounts on One Computer: A Step-by-Step Guide"
seoTitle: "Easily Manage Multiple GitHub Accounts"
seoDescription: "Easily manage multiple GitHub accounts on one computer by setting up SSH keys and configuring your system with this step-by-step guide"
datePublished: Sat Dec 28 2024 16:18:15 GMT+0000 (Coordinated Universal Time)
cuid: cm58dyhgv000509lebgpidico
slug: managing-multiple-github-accounts-on-one-computer-a-step-by-step-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735402376940/51ee5491-090b-423f-84ba-008acfaab569.png
tags: github, bash, git, ssh, configuration

---

## Introduction

Managing multiple GitHub accounts on a single computer can seem daunting, but with a structured approach, it becomes straightforward and efficient. Let’s suppose you have two GitHub accounts, [https://github.com/rohit-office](https://github.com/rohit-office) and [https://github.com/rohit-personal](https://github.com/rohit-personal), and you want to set up your machine to easily communicate with both.

By setting up unique SSH keys, adding them to the SSH agent, and configuring the **~/.ssh/config** file, you can seamlessly switch between personal and professional accounts without manual intervention.

> NOTE: This logic can be extended to more than two accounts also. :)

The setup can be done in 5 easy steps:

## Step 1 - Create SSH keys for all accounts

First make sure your current directory is your **.ssh** folder.

```bash
$ cd ~/.ssh
```

Syntax for generating unique ssh key for an account is:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f "desired-key-name"
```

here,

**\-C** stands for comment to help identify your ssh key

**\-f** stands for the file name where your ssh key get saved

#### Now generating SSH keys for my two accounts

```bash
ssh-keygen -t ed25519 -C "my_office_email@gmail.com" -f "github-rohit-office"
ssh-keygen -t ed25519 -C "my_personal_email@gmail.com" -f "github-rohit-personal"
```

Notice here **rohit-office** and **rohit-personal** are the username of my GitHub accounts corresponding to **my\_office\_email@gmail.com** and **my\_personal\_email@gmail.com** email ids respectively.

After entering the command the terminal will ask for passphrase, leave it empty and proceed.

```bash
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

> Now after adding keys , in your `.ssh` folder, a public key and a private will get generated.

> The public key will have an extension `.pub` and private key will be there without any extension both having same name which you have passed after **\-f** option in the above command. (in my case **github-rohit-office** and **github-rohit-personal**)

## Step 2 - Ensure the SSH Agent is running

Start it manually:

```bash
# start the ssh-agent in the background
$ eval "$(ssh-agent -s)"
> Agent pid 59566
```

## Step 3 - Add SSH keys to SSH Agent

Now we have the keys but it cannot be used until we add them to the SSH Agent.

```bash
$ ssh-add -K ~/.ssh/github-rohit-office
$ ssh-add -K ~/.ssh/github-rohit-personal
```

You can read more about adding keys to SSH Agent [here.](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## Step 4 - Add SSH public key to the GitHub

For the next step we need to add our public key (that we have generated in our previous step) and add it to corresponding GitHub accounts.

For doing this we need to:

**1\. Copy the public key**

We can view the public key either by viewing content of the github-rohit-office.pub file using:

```bash
# View file content without opening
$ cat ~/.ssh/github-rohit-office.pub
$ cat ~/.ssh/github-rohit-personal.pub
```

Then copying the content of it.

***OR***

We can directly copy the content of the public key file in the clipboard.

```bash
# Copy file content to clipboard
$ cat ~/.ssh/github-rohit-office.pub > /dev/clipboard
$ cat ~/.ssh/github-rohit-personal.pub > /dev/clipboard
```

**2\. Paste the public key on GitHub**

* Sign in to GitHub Account
    
* Go to **Settings** &gt; **SSH and GPG keys** &gt; **New SSH Key**
    
* Paste your copied public key and give it a Title of your choice.
    

***OR***

* Sign in to GitHub
    
* Paste this link in your browser (https://github.com/settings/keys) or click [here](https://github.com/settings/keys)
    
* Click on **New SSH Key** and paste your copied key.
    

## Step 5 - Create a Config File and Make Host Entries

The `~/.ssh/config` file allows us specify many config options for SSH.

If **config** file not already exists then create one (make sure you are in `~/.ssh` directory)

```bash
$ touch config
```

open **config** file in any editor of your choice or run the command below to open config in `nano` editor.

```sh
$ nano config
```

Now we need to add these lines to the file, each block corresponding to each account we created earlier.

```bash
# Office account
Host github.com-rohit-office
  HostName github.com
  User git
  IdentityFile ~/.ssh/github-rohit-office

# Personal account
Host github.com-rohit-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/github-rohit-personal
```

## Step 6 - Cloning GitHub repositories using different accounts

So we are done with our setups and now its time to see it in action. We will clone a repository using one of the account we have added.

```bash
# Syntax to clone using diffrent accounts
$ git clone git@github.com-{your-username}:{owner-user-name}/{the-repo-name}.git
```

For Example: I am making a repository on my personal GitHub account and naming it **TestRepo**

Now for cloning the repo use the below command:

```bash
$ git clone git@github.com-rohit-personal:rohit-personal/TestRepo.git
```

## Finally

From now on, to ensure that our commits and pushes from each repository on the system uses the correct GitHub user — we will have to configure **user.email** and **user.name** in every repository freshly cloned or existing before.

To do this use the following commands.

```bash
# Inside the git repo where office project located
$ git config --local user.email "my_office_email@gmail.com"
$ git config --local user.name "Rohit Tewari"
     
# Inside the git repo where personal project located
$ git config --local user.email "my-personal-email@gmail.com"
$ git config --local user.name "Rohit Tewari"
```

Pick the correct pair for your repository accordingly.

To push or pull to the correct account we need to add the remote origin to the project (Optional: if remote not already added).

```bash
# Syntax
$ git remote add origin git@github.com-{your-username}:{owner-user-name}/{the-repo-name}.git

# Inside the git repo where office project located
$ git remote add origin git@github.com-rohit-office:rohit-office/TestRepo.git

# Inside the git repo where personal project located
$ git remote add origin git@github.com-rohit-personal:rohit-personal/TestRepo.git
```

> NOTE: `git@github.com-{your-username}` is necessary to clone using right GitHub account

Now you can use:

```bash
$ git pull
$ git push
```

## Conclusion

Handling multiple GitHub accounts just got easier! By setting up SSH keys and configuring your machine, you can switch between personal and work accounts like a pro. No more mix-ups, no more hassle — just smooth sailing on your coding journey. Let’s keep building great things! 🚀✨