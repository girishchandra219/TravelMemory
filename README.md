Travel Memory app on AWS:
I deployed the Travel Memory app on aws ec2 with 2 backend and 2 frontend servers, each pair behind a load balancer. The database is mongodb atlas.
Link: http://travelmemory-frontend-alb-123265019.us-east-1.elb.amazonaws.com

Architecture:
Region is us-east-1. Master ec2 are in us-east-1d and the replicas are in us-east-1b.

Step 1 - Mongodb atlas:
Created a cluster on gui, a user, and allowed 0.0.0.0/0 in network access.

Step 2 - ec2 instances:
Launched backend-master and frontend-master. Security group allows ports 22, 80 and 3000.

Step 3 - Backend:
sudo apt update
sudo apt install -y nodejs npm git nginx
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
npm install
-----------------------------------------------
Created .env:
MONGO_URI=mongodb+srv://<user>:<password>@cluster0.xxxxx.mongodb.net/travelmemory
PORT=3000
-----------------------------------------------
sudo systemctl daemon-reload
sudo systemctl enable --now travelmemory-backend
-----------------------------------------------
Set up nginx as a reverse proxy port 80 to 3000. File: deployment/nginx-backend.conf

Step 4 - Frontend: 
Installed the same packages, added 2GB swap (to make sure memory shouldn't max out) and cloned the repo and set the backend IP in src/url.js.
-----------------------------------------------
npm install
npm run build
sudo rm -rf /var/www/html/*
sudo cp -r build/* /var/www/html/
-----------------------------------------------
Nginx config: deployment/nginx-frontend.conf

Step 5 - Testing before ami: 
Added a trip from the form and checked it in the app and in Atlas.

Step 6 and 7 - amis and second instances: 
Created amis from both masters and launched backend-2 and frontend-2 from them in a different zone. backend-2 worked without logging in because of systemd.

Step 8 - backend load balancer
Target group with health check /hello, both backend servers registered, alb listening on port 80.

Step 9 - frontend load balancer
Same thing for the frontend, health check /.

Step 10 - frontend pointing to backend alb
Changed src/url.js on both frontend servers to the backend ALB address, rebuilt and copied again.
-----------------------------------------------
export const baseUrl = "http://travelmemory-backend-alb-362367303.us-east-1.elb.amazonaws.com";
-----------------------------------------------

Notes:
- I did not set up Cloudflare. The app works on the frontend alb link.
- SSH was blocked on my office network, so I used EC2 Instance Connect.
- .env is not pushed to gitHub because it has the DB password.

  ## Screenshots

### Architecture
![](Images/architecture-diagram.png)

### MongoDB Atlas
![](Images/01-atlas-cluster.png)
![](Images/02-atlas-db-user.png)
![](Images/03-atlas-ip-access.png)

### EC2 and security group
![](Images/04-ec2-instances.png)
![](Images/05-security-group.png)
![](Images/06-key-pair.png)

### Backend setup
![](Images/07-os-version.png)
![](Images/08-node-version.png)
![](Images/09-backend-files.png)
![](Images/12-systemd-status.png)
![](Images/13-curl-hello-systemd.png)
![](Images/15-browser-hello.png)
![](Images/16-after-reboot.png)

### Frontend setup
![](Images/17-frontend-os.png)
![](Images/18-frontend-swap.png)
![](Images/19-frontend-url-js.png)
![](Images/20-frontend-build.png)

### Testing
![](Images/22-app-homepage.png)
![](Images/23-add-experience-form.jpeg)
![](Images/24-trip-listed.png)
![](Images/25-atlas-data.png)

### AMIs and second instances
![](Images/26-amis-available.png)
![](Images/27-four-instances.png)
![](Images/28-backend-2-hello.png)
![](Images/29-frontend-2-app.png)

### Backend load balancer
![](Images/30-backend-tg.png)
![](Images/31-backend-targets-healthy.png.png)
![](Images/32-backend-alb-hello.png)

### Frontend load balancer
![](Images/33-frontend-tg.png)
![](Images/34-frontend-targets-healthy.png)
![](Images/35-frontend-alb-app.png)

### Frontend pointing to backend ALB
![](Images/36-url-js-alb.png)
![](Images/37-final-app-via-alb.png)
![](Images/38-network-tab-backend-alb.png)
