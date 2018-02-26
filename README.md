### AWS Console

#### 1. Launch Instance

[Go to console and click EC2](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2), then on the button to "Launch Instance"

#### 2. Choose AMI and Instance Type

Select Ubuntu and General Purpose Instance Type (free tier eligible)

#### 3. Next a few times

Leave default settings for: Configure Instance, Add Storage, Add Tags

#### 4. Configure Security Groups

Choose the servers-group-1, or create new security group with rules:
- SSH         | TCP | Port 22   | Source: 0.0.0.0/0
- HTTP        | TCP | Port 80   | Source: 0.0.0.0/0
- Custom Rule | TCP | Port 3000 | Source: 0.0.0.0/0, ::/0

#### 5. Click on Review and Launch

And then on Launch again after reviewing

#### 6. Choose an SSH key

When prompted, select the servers-key-1 (or create a new one) -- download file and store in safe location

#### 7. Click on Launch (again!)

Should be [new instance on the list of Instances](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#Instances:sort=instanceId). Could take a few mins to initialize.

----

### SSH into Server

#### 8. Set permission on SSH key

To use the key (if new key was generated), permissions need to be set using chmod command

```
$ chmod 400 ~/.ssh/whatever-your-key-name-is.pem
```

#### 9. Create script to SSH into EC2

To get the line of code needed, right click on the Instance and select 'connect'. The file (in Ruby) will look something like this:

```ruby
# file: lib/tasks/ssh.rake
namespace :ssh do
	task :login => :environment do
		system 'ssh -i ~/.ssh/whatever-your-key-name-is.pem ubuntu@ec2-11-22-33-44.us-west-2.compute.amazonaws.com'
	end
end
```

#### 10. Install dependencies

If using Rails, install ruby and rails on the server. Or any other required libraries.
- [Installing Rails 5 on Ubuntu](http://blog.teamtreehouse.com/installing-rails-5-linux)
- [Installing Node](https://hackernoon.com/tutorial-creating-and-managing-a-node-js-server-on-aws-part-1-d67367ac5171)

#### 11. Clone repo from source

Choose a file where the repo should live and clone the repo from Github

----

### Using NGINX as a Proxy Server

#### 12. Install Nginx

Run the following command `sudo apt-get install nginx` on the EC2 Instance
Visit the URL and if Nginx is not working, might need to run: `sudo /etc/init.d/nginx start`

#### 13. Remove the default from site-enabled

Remove the default file by running `sudo rm /etc/nginx/sites-enabled/default`

#### 14. Create a new config file on sites-available

File will look like this and be placed in /etc/nginx/sites-available/server-name
```
server {
  listen 80;
  server_name server-name;
  location / {
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  Host       $http_host;
    proxy_pass        http://127.0.0.1:3000;
  }
}
```

#### 15. Link config file in the sites-eneabled folder

Run the following command: `$ sudo ln -s /etc/nginx/sites-available/server-name /etc/nginx/sites-enabled/server-name`

and then restart Nginx: `$ sudo service nginx restart`

----

### Setting up PM2

#### 16. Install NPM (if not installed)

[Insalling NPM](https://www.rosehosting.com/blog/install-npm-on-ubuntu-16-04/)

#### 17. Install pm2 globally

Run `$ npm i -g pm2`

#### 18. Create a file to run your server

File could be located in ~/server/start-rails.rb and look like this:
```ruby
system `cd ~/server/rails-app/ && bundle install && rails s -p 3000`
```

#### 19. Start server process

Run `$ pm2 start start-rails.rb --name "rails-server"`

#### 20. Run startup and save

Run `$ pm2 startup` and then copy/paste the command that's returned
Run `$ pm2 save`
[More info](http://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/)

----

#### NOTES:

- To run a process in the Background without a process manager:
	- Hit Ctrl+z
	- Run `$ bg %1`
	- To view, run `$ jobs`
	- To stop, run `$ kill %1`

