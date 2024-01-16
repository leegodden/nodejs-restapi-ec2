## Getting Started

-   STEP1 - Login to AWS console and create EC2 instance
-   STEP2 - Setup GitHub Repo and Push your project
-   STEP3 - Login to EC2 instance
-   STEP4 - Setup GitHub Action runner on EC2 instance
-   STEP5 - Create GitHub Secrets for managing environment variables
-   STEP6 - Create CI/CD Workflow using GitHub Action
-   STEP7 - Install nodejs and nginx on EC2 instance

```bash
sudo apt update

curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

sudo apt-get install -y nodejs

sudo apt-get install -y nginx
```

-   STEP8 - Install pm2

```bash
sudo npm i -g pm2
```

-   STEP9 - Config nginx and restart it

```bash
cd /etc/nginx/sites-available

sudo nano default

location /api {
	rewrite ^\/api\/(.*)$ /api/$1 break;
	proxy_pass  http://localhost:5000;
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}

sudo systemctl restart nginx.
```

-   STEP10 - Run backend api in the background as a service using pm2

```bash
pm2 start server.js --name=BackendAPI
```

-   STEP11 - Add the command in yml script of project to restart the nodejs api server after every push to the repo

```bash
run: pm2 restart BackendAPI
```

////////////////////////////////////////////////////////////////////////////////////////////////////

# Auto Deploy Nodejs Rest API on AWS EC2 | CI/CD pipeline using GitHub Actions

This app Sets up an automatic deployment of Nodejs Rest API on AWS EC2 instance using GitHub Actions

YT Source:			`https://www.youtube.com/watch?v=cgWXQqL-ZU8`
Repo: 				`https://github.com/leegodden/nodejs-restapi-ec2`
AWS IAM account: 	`zerotohero`

1. create project with mongo Atlas project 
2. test routes 
3. browse collectons in the project in Atlas to see that routes have added to db
4. push project to github repo

UPTO: `https://youtu.be/cgWXQqL-ZU8?t=222`
5. Signin in to AWS account and create an EC2 instance

# Setup AWS instance

1. Login to AWS & go to Ec2 and click Launch Instance
2. give instance a name and choose ubuntu OS
3. create a key pair using RSA
4. Launch instance

5. create a security group for our created AWSinstance by clicking `Edit inbound rules` on the 
	default security group

6. Click `Add rule` and choose html with `0.0.0.0/0` for access anywhere

# Conneting to AWS Instance

1. Now select our instance in the list and click `Connect` button
2. Select the SSH client tab and copy the the example connection string:
	 This will be like: `ssh -i "your-key-name.pem" ubuntu@ec2-35-179-97-97.eu-west-2.compute.amazonaws.com`

3. Open terminal and navigate to the directory where the key.pem is
4. Add chmod permission to the key as example: `chmod 400 key-name.pem`
5. Now copy the example connection we copied ealier add change `your-key-name.pem` to you actual key name
6. Hit enter and you should no be in the `ubuntu` server Ec2 instance

# Setup Github Actions

create `node.js yml` file in `github/workflows` directory and add code

////////////////////////////////////////////////////////////////////////////////////////////////

# NOTES FOR node.yml

`name: Node.js CI/CD`
sets the name of the GitHub Actions workflow to "Node.js CI/CD."

------------------------------------------------------------------------------------------------

```yaml

on:
  push:
    branches: [ "main" ]
```

The above code defines the `trigger` for the workflow. In this case, the workflow will run when there is a 
push to the `main` branch.

------------------------------------------------------------------------------------------------

```yaml

jobs:
  build:
    runs-on: self-hosted
    strategy:
      matrix:
        node-version: [18.x]
```
The above code declares a single job named `build` The job will run on a `self-hosted runner` (indicated by 
`runs-on: self-hosted)`. The strategy section defines a `matrix` that specifies different versions of Node.js 
to be used. In this case, it's using `Node.js version 18.x`.

A `self-hosted runner` in the context of GitHub Actions is a machine (physical or virtual) that you set up 
and manage on your own infrastructure. Unlike GitHub-hosted runners, which are provided and maintained 
by GitHub, self-hosted runners run on your own hardware or cloud infrastructure.

-------------------------------------------------------------------------------------------------------------

```yaml

steps:
    - uses: actions/checkout@v3
```
This step checks out the source code from the repository using the `actions/checkou` action at version 3.

-------------------------------------------------------------------------------------------------------------

```yaml

- name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
```
This step sets up the Node.js environment using the `actions/setup-node` action at version 3. It uses 
the Node.js version specified in the `matrix` and also caches the `npm packages` to speed up 
subsequent builds.

-------------------------------------------------------------------------------------------------------------

`- run: npm ci`
This step installs Node.js dependencies using npm ci, which is a more strict and faster way to install 
dependencies than npm install for CI/CD environments.


-------------------------------------------------------------------------------------------------------------

```yaml

- run: |
        touch .env
        echo "${{ secrets.PROD_ENV_FILE }}" > .env
```
This step creates a new .env file and writes the value of the secret named `PROD_ENV_FILE` into it.

-------------------------------------------------------------------------------------------------------------

`- run: pm2 restart BackendAPI`
This step restarts the process managed by `PM2` named `BackendAPI`. This assumes that PM2 is installed and 
configured in the environment. This command is often used to restart a `Node.js` application in a 
production environment.

In summary, this GitHub Actions workflow is triggered on pushes to the "main" branch, sets up a self-hosted runner 
with Node.js version 18.x, installs dependencies, creates an environment file from a secret, and restarts a 
Node.js application using PM2.

////////////////////////////////////////////////////////////////////////////////////////////////

# Setup Github actions/runner in our Ec2 instance

1. in our Github repo select `Settings` > `Actions` > `Runners`
2. click on `New self-hosted runner` button, and choose `linux` (as we are using `ubuntu `or our EC2 instance)

	Here you'll see some instructions & examples for creating a self-hosted runner. First we need to create a 
	folder for our runner in the EC2 instance

3. in the terminal create a new folder for our Ec2 instance  as below example and cd into it.
	`mkdir folder-name && cd folder-name`

4. press `Enter

5. back in Github, copy the `curl` command to download the `lastest runner package` into our Ec2 instance, paste into 
	the terminal, and hit enter

6. Next copy the `Validate the hash` command from Github, paste into the terminal and hit Enter.
7. Now copy the  `Extract the installer` command, past into the terminal and hit enter

8. Now in Github, under `Configure` copy the `Create the runner and stsrt the configuration experience` command,
	paste into the terminal and hit enter.

9. Now once you see the `Github Actions` logo in the termianla press Enter to accept the default Runner name
10. press enter again and then once more.

You should now  see the `Runner Successfully added` and `Runner connection is good`. 

11. At the `Enter name of work folder:` prompt press Enter again to  go with the default to see the 
	 `Settings saved` message

12. run `ls` to view the files and folder added to our `/actions-runner-backendserver` folder

Now we need to install the `svc.sh` script file to run the runner as a background service .
When you run `sudo ./svc.sh install` as below, it configures the necessary settings to ensure that the runner 
starts automatically as a service when the machine boots up.

13. in the termianl run `sudo ./svc.sh install`
14. we now have to run the `svc.sh` script file. run: `sudo ./svc.sh start`

After running the script You should now see the below in the returned output
Active: `active (running) since...`

15. Now go to `Guthub` > `Runners` page and refresh browser to see the state of the Runner which should 
	at this stage it be `idle`

# Create Github Secrets

1. in Github go to `Settings` > `Secrets & Varaibles` > `Actions`
2. click the `New Repository Secret` button
3. In the `Name*` field add a Secret name, then copy all our .env variables in the the `Secret*` section

4. Now click `Add secret button`
5. go to: `https://github.com/leegodden/nodejs-restapi-ec2/actions/new`
6. under `Continuous Integration`, choose `Node.js` by clicking the `Configure` button
7. Ammend the code to our `Node.js.yaml` file
