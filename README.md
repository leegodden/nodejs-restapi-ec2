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
4. push project to repo

UPTO: `https://youtu.be/cgWXQqL-ZU8?t=222`
5. Signin in to AWS account and create an EC2 instance