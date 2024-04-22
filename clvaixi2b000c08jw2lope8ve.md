---
title: "Automating MERN App Deployment on DigitalOcean with GitHub Actions"
seoTitle: "Automating MERN Deployment on DigitalOcean"
seoDescription: "Automate MERN app deployment on DigitalOcean using GitHub Actions for seamless continuous deployment"
datePublished: Mon Apr 22 2024 05:35:56 GMT+0000 (Coordinated Universal Time)
cuid: clvaixi2b000c08jw2lope8ve
slug: automating-mern-app-deployment-on-digitalocean-with-github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713633787966/fb22225a-0a47-4968-a289-bbb3290dd755.png
tags: digitalocean, github, bash, git, mern, cicd, github-actions

---

### Introduction

Deploying a MERN stack application can be challenging, but it becomes manageable with the right approach. In this guide, we'll walk through the steps to deploy your MERN app on a DigitalOcean server using GitHub Actions for continuous deployment. By automating the deployment process, you can ensure a seamless transition from development to production. Let's dive in!

### Step 1 - Create a new repository that will host your application on GitHub

Begin by creating a new repository on GitHub to host your application. Follow the steps outlined in [GitHub's documentation](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/create-a-repo).

In this case, I created one as shown below :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713627649854/451f39b7-a655-41bf-a34e-44042b704073.png align="center")

### Step 2 - Create & Configure A New DigitalOcean Server

Before proceeding further, set up a Virtual Private Server (VPS) on DigitalOcean to host your application.

To create a new droplet on DigitalOcean check out [this](https://github.com/rtewari056/digitalocean-deployment#step-1---create--configure-a-new-digitalocean-server) tutorial.

### Step 3 - Set up the Nginx, Node.js and PM2

Configure Nginx, Node.js, and PM2 for your MERN project using the following guides:

1. [Install & Configure Nginx](https://github.com/rtewari056/digitalocean-deployment#step-3---install--configure-nginx)
    
2. [Install Node.js](https://github.com/rtewari056/digitalocean-deployment#install-nodejs)
    
3. [Install PM2](https://github.com/rtewari056/digitalocean-deployment#install-pm2)
    

### Step 4 - Configure Continuous Deployment to DigitalOcean

With the deployment server set up and running, let's Implement a deployment workflow to automate the process of pushing changes to the server.

### Configure Self-hosted Runner

We will use the self-hosted runner option in the deployment workflow for deployment. For this example, the runner will be at a repository level hosted on the DigitalOcean Droplet that was set up in the previous section.

Go to the `Settings` &gt; `Actions` &gt; `Runners` section of the GitHub repository, then click the `New self-hosted runner` button.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713627802033/a7ed5130-c9b8-4d00-8bbc-fe9f04c84627.png align="center")

Since we have `Ubuntu` installed on the Droplet, select `Linux` as the runner image and `x64` as the architecture.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713627852111/687f0523-b62c-4abd-adaf-18d4de5f5c86.png align="center")

To integrate the `self-hosted` runner into your `Ubuntu` server, log in to your VPS via the terminal, and follow the `Download` and `Configure` instructions provided in your `Settings` &gt; `Actions` &gt; `Runners` page.

On your local machine, SSH into your server. Use the following command to do so (substitute your username and IP address):

```bash
$ ssh USERNAME@SERVER_IP_ADDRESS
```

If you have too many keys on your local Machine, then you can try specifying which key you want to use:

```bash
$ ssh -i ~/.ssh/PRIVATE_KEY_FILE_NAME USERNAME@SERVER_IP_ADDRESS
```

Let’s start by creating a folder for our application:

```bash
rohit@hostname:~$ mkdir your_project_name && cd your_project_name
```

Download the latest runner package:

```bash
rohit@hostname:~$ sudo curl -o actions-runner-linux-x64-2.298.2.tar.gz -L https://github.com/actions/runner/releases/download/v2.298.2/actions-runner-linux-x64-2.298.2.tar.gz
```

Extract the installer:

```bash
rohit@hostname:~$ sudo tar xzf ./actions-runner-linux-x64-2.298.2.tar.gz
```

The `actions-runner-linux-x64-2.298.2.tar.gz` package won't be needed again, so let's delete it:

```bash
rohit@hostname:~$ sudo rm actions-runner-linux-x64-2.298.2.tar.gz
```

Before going forward, we need to give the user full access permission to install and configure the runner inside the current directory:

```bash
rohit@hostname:~$ sudo chmod -R 777 <PATH_TO_YOUR_PRESENT_WORK_DIRECTORY>
```

Configure the runner:

```bash
rohit@hostname:~$ ./config.sh --url https://github.com/GITHUB_USERNAME/GITHUB_REPO_NAME --token YOUR_TOKEN
```

You will be asked to enter some information to register your `self-hosted` runner with GitHub Actions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713627995805/8cde7afd-9aeb-45eb-aa0a-abdd63a9d8e8.png align="left")

Now go to the `Settings` &gt; `Actions` &gt; `Runners` section of your project repository.

If your `self-hosted` runner successfully registered, you will see your runner is currently offline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713628027436/89c60717-96c3-4387-a4c4-d3dc5ab3197b.png align="left")

Start the runner as a service:

```bash
rohit@hostname:~$ sudo ./svc.sh install
rohit@hostname:~$ sudo ./svc.sh start
```

If you go to the `Settings` &gt; `Actions` &gt; `Runners` section of your project repository, you will see that `self-hosted` runner is now successfully started.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713628054604/2e0f3f77-c650-4705-8db9-24c060a39054.png align="left")

### Configuring Nginx to display our project

The Nginx configuration is kept in the `/etc/nginx/sites-available` directory. To create a new configuration, let’s navigate to this directory and create a configuration file pointing to the server block of our Node.js application.

```bash
rohit@hostname:~$ cd /etc/nginx/sites-available
rohit@hostname:~$ sudo nano myserver.config
```

Paste in the following configuration:

```bash
# The Nginx server instance
server {
    listen 80;
    server_name your_domain www.your_domain;

    location / {
        # Replace 8080 with the port number where your node js server is running
        proxy_pass http://localhost:8080;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_cache_bypass $http_upgrade;

        # Security Patches (Optional)
        server_tokens off;
        proxy_hide_header X-powered-by;
        proxy_hide_header X-Runtime;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Frame-Options "deny";
        add_header X-Content-Type-Options "nosniff";
    }
}
```

Save the file and exit the editor (`CTRL-X`+`Y`+`ENTER`).

For the next step, let’s enable the above file by creating a symbolic from it to the `sites-enabled` directory, which Nginx reads from during startup:

```bash
rohit@hostname:~$ sudo ln -s /etc/nginx/sites-available/myserver.config /etc/nginx/sites-enabled/
```

The server block is now enabled and configured to return responses to requests based on the `listen` port and `location` path.

let’s check the status of Nginx to confirm that the configuration is working properly:

```bash
rohit@hostname:~$ sudo nginx -t
```

The output upon running the above command would look like this:

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

The above output confirms that our configuration was successful. Next, restart Nginx to enable your changes:

```bash
rohit@hostname:~$ sudo systemctl restart nginx
```

### Create a GitHub Actions workflow file

Clone your project from GitHub on your local computer. Then create a directory named `.github` in your project's root directory and create a subdirectory named `workflows` inside the `.github` directory.

Now go to `.github` &gt; `workflows` and create a file with any name you want but make sure it ends with `.js.yml` so `GitHub Actions` can recognize it. In my case, I will name the file as `deployment.js.yml`.

Paste in the following configuration inside `deployment.js.yml`:

```yml
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and deploy the application
# For more information see: https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

name: Build & Deploy

# Controls when the workflow will run
on:
  push:
    branches: [ "main" ] # Will run on push to the "main" branch

jobs:
  build:

    runs-on: self-hosted # The type of runner that the job will run on

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js 18 # We can make it more understandable by adding names to the steps
      uses: actions/setup-node@v3.5.0
      with:
        node-version: '18.x' # Change according to the installed version
        cache: 'npm'
    
    - name: Install dependencies
      run: | 
        npm install
        cd frontend
        npm install

    - name: Creating a Production Build of React App
      run: | 
        npm run build
        cd ..
      
  create-envfile:
 
    runs-on: self-hosted
    needs: build # Run only after the 'build' job is completed
 
    steps:
    - name: Create .env file
      # Creates an '.env' file with environment variables
      run: |
        touch .env
        echo JWT_SECRET_KEY=${{ secrets.JWT_SECRET_KEY }} >> .env
        echo MONGO_URI=${{ secrets.MONGO_URI }} >> .env
        echo PORT=${{ secrets.PORT }} >> .env
        echo NODE_ENV=${{ secrets.NODE_ENV }} >> .env

  deploy:
 
    runs-on: self-hosted
    needs: [build, create-envfile] # Run only after the 'build' and 'create-envfile' job is completed
 
    steps:
    - name: Deploy to production 
      # Starts your node js app in 'PM2'
      run: |
        pm2 stop ecosystem.config.js
        pm2 start ecosystem.config.js
        pm2 save
```

If you are using `.env` file to use environment variables inside your project, you have to define `actions secrets` in your GitHub repository.

To define secrets, go to `Settings` &gt; `Secrets` &gt; `Actions` and click on `New repository secret` button. Then add your environment variable name and value inside the textarea.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713628133204/a1ca63a8-c391-4a14-b766-3423e9b886b0.png align="left")

After adding the secrets it will look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713628148302/618ebdf8-3f36-4158-b09f-da1fe5d4ef20.png align="left")

Along with that, we will also create a configuration file named `ecosystem.config.js` in our project root directory for `PM2` so we can easily `start/stop/restart` our `node js` application whenever we push changes to GitHub. You can name it anything but make sure it ends with `.config.js` so `PM2` can recognize it as a configuration file.

Paste in the following configuration inside `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [
    {
      name: "chat-app",
      script: "./backend/server.js",
    },
  ],
};
```

Replace `chat-app` with your application name and `./backend/server.js` with the relative path of your node js entry point file.

Finally, push the changes to GitHub to trigger the workflow.

### Conclusion

Upon completion, check the `Actions` tab in your GitHub repository to monitor the workflow's progress. A successful deployment will be indicated by a green checkmark.