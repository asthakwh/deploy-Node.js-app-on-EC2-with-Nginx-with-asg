AWS Console → EC2 → Target Groups → Create
add instance
add security group

Load Balancers → Create → Application LB

 ALB Security Group — Inbound

        HTTP  port 80   from 0.0.0.0/0
        HTTPS port 443  from 0.0.0.0/0

 ALB Security Group — Outbound

All traffic → 0.0.0.0/0

listen: http 80
Target group: name

now
Create AMI from your running instance then Create Launch Template

Paste User Data script -according to instance


            #!/bin/bash
            set -e
        # Update system

            sudo apt update -y
            sudo apt upgrade -y
        # Install Node.js (LTS)

            curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
            sudo apt install -y nodejs git nginx
        # Install PM2 globally

            sudo npm install -g pm2
        # Go to home directory

            cd /home/ubuntu
        # Clone your repo

                git clone https://github.com/asthakwh/deploy-Node.js-app-on-EC2-with-Nginx-with-asg.git app
            cd app
        # Install dependencies

            npm install --production
        # Start app using PM2

            pm2 start app.js
            pm2 startup systemd
        pm2 save
        # Configure Nginx reverse proxy

            sudo bash -c 'cat > /etc/nginx/sites-available/default <<EOF
            server {
                listen 80;
                server_name _;

                location / {
                   proxy_pass http://localhost:3000;
                    proxy_http_version 1.1;
                   proxy_set_header Upgrade \$http_upgrade;
                    proxy_set_header Connection "upgrade";
                    proxy_set_header Host \$host;
                    proxy_cache_bypass \$http_upgrade;
                }
            }
            EOF'

# Restart Nginx

            sudo systemctl restart nginx
            sudo systemctl enable nginx
############################################################

to verify 
1=paste on browser http://alb-dns-id
2=Target group shows instances as Healthy
3=Simulate scale-out to test(check with delete instances)
4=
