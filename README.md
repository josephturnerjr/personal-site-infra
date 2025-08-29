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
