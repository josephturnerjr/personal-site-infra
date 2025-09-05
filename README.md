# VPS Setup

* Log in as root via LISH
* apt update
* apt upgrade
* shutdown -r now
* adduser jturner
* usermod -aG sudo jturner
* Check the new user’s sudo privs:
  * Sudo su jturner
  * Sudo whoami
* Set up jturner access
  * Copy your local pubkey over
    * ssh-copy-id -i ~/.ssh/id_ed25519_personal.pub jturner@ip_of_vps
  * Check that it works
    * ssh jturner@ip_of_vps
* Update your local ssh config ~/.ssh/config
  * Host friendlyname
    Hostname ip_of_vps
    User jturner
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519_personal
* Update ssh config 
  * Vi /etc/ssh/sshd_config
  * PermitRootLogin no
  * PasswordAuthentication no
  * Port 1022
  * Sudo service ssh restart
* Good idea to reboot at this point
  * Sudo shutdown -r now
* Verify the new ssh settings work (root should fail, 22 should connection refused
* Update your local ssh config to use the new port
* Set up the firewall
  * Verify it is disabled
    * Sudo ufw status
  * Set defaults
    * Sudo ufw default deny incoming
    * Sudo ufw default allow outgoing
  * Allow ssh (this is important lol)
    * Sudo ufw allow 1022
    * Allow http/https
    * Sudo ufw allow http
    * Sudo ufw allow https
    * Optiona: QUIC
      * Sudo ufw allow 443/udp
  * Enable it
    * Sudo ufw enable
  * Check your configs
    * Sudo ufw status verbose
* Set up fail2ban
  * sudo apt install fail2ban
  * sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
  * Edit the jail.local file
    * [ssh]
      enabled = true
      mode = aggressive
  * sudo systemctl restart fail2ban
  * sudo systemctl enable fail2ban
  * sudo systemctl status fail2ban
* Set up unattended upgrades
  * sudo apt install unattended-upgrades
  * Sudo systemctl status unattended-upgrades
  * sudo systemctl start unattended-upgrades
  * sudo systemctl enable unattended-upgrades
* Set up for Caddy
  * Sudo mkdir /etc/caddy
  * Sudo mkdir /etc/caddy/caddy_data
  * Sudo mkdir /etc/caddy/caddy_config
* Install docker and docker compose and set up
  * Sudo apt install docker.io docker-compose-v2
  * sudo usermod -aG docker jturner
  * Newgrp docker (or new session)
* Set up the infra repo on the VPS
  * Cd ~
  * Mkdir infra.git
  * Cd infra.git
  * Git init --bare
  * Git branch -m main
  * Copy the content from infra/hooks/post-receive into infra.git/hooks/post-receive
  * Chmod 755 post-receive
  * Locally, add the infra remote
    * Cd <wherever you have the infra project checked out>
    * git remote add deploy friendlyname:~/infra.git
  * Push the repo
    * Git push deploy main
  * Check that the service started
    * On the VPS, docker ps
* At this point, you have Caddy running with the Caddyfile in the infra repo. You need to deploy the actual sites’ docker containers to get them to show up. For each Dockerized site:
  * Make sure your port exposure matches the reverse proxy rule in the Caddyfile for each site
  * Set up a bare repo on the VPS and a remote locally
  * Use the same post-receive hook from the infra project to build and (re)deploy the site
  * Push locally to the VPS remote
  * Check your site

# Running the site for dev
  
docker run -p 4000:4000 -v $(pwd):/site bretfisher/jekyll-serve

# Set up a new project 

docker run -v $(pwd):/site bretfisher/jekyll new .

# Build the current project

docker run -v $(pwd):/site bretfisher/jekyll build


# How to deploy a new static site

  * Create the server block in /etc/nginx/sites-available
  * Link the server block in /etc/nginx/sites-enabled
  * Test the config with `sudo nginx -t`
  * Restart nginx with `sudo service restart nginx`
  * Create the content folder in /var/www/mydomain.com
  * Create a subfolder in that folder called site
  * Chown user+group of the site folder to your deploy user


# How to set up push-to-deploy

  * Create folder myrepo.git in home directory on server
  * `git init --bare`
  * `cp hooks/post-receive.sample hooks/post-receive`
  * `mkdir /var/www/myrepo`
  * Add your build logic to hooks/post-receive (see [this
    site](https://jekyllrb.com/docs/deployment/automated/) for example jekyll
    build)
  * Make sure that hooks/post-receive is executable
  * Then on the dev system, `git remote add deploy deployer@example.com:~/myrepo.git`


# How to push to deploy

  * `git push deploy main`

