### How to deploy my blog in 3 easy steps!

It's now time to provision a server using a tool called Ansible. Once we've completed that, we're going to use another tool called Capistrano to then deploy your new blog app onto that server.

#### Step 1. What is Ansible
Ben's IT Lessons
youtube.com/watch?v=icR-df2Olm8

The video tutorial above is great for walking you through the basics of how to use Ansible and understand it's framework.

#### Step 2. Make a dog house and then put the dog in it.
Chris Herring
youtube.com/watch?v=3QTD6nHKVYI

This tutorial shows you how to provision a rails server with Ansible and then deploy your rails app with Capistrano. It's rather daunting at first. It goes over two big concepts in one short video but doesn't really hold your hand. It took me a few days to make it to the end of this 16 minute video due to all the errors I had to research and overcome. Not to mention that the playbook he uses in this tutorial is hidden behind a mailing list. Don't worry, I had already jumped through the hoops for you and was able to grab his playbook. It's on my GitHub located at the link below.

https://github.com/eKioga/blog_playbook.git

You may have already noticed that there are actually two playbooks here, server_pre_setup.yml and server_setup.yml. Follow the logic of these playbooks and make sure that you understand what all the roles do. 

The server_pre_setup playbook primarily handles the Python installation, user creation, SSH key transfer and hardening. Make sure that your pub key path is added to the pre_setup playbook. The last few steps allow your Ansible Control Machine to no longer require a password to connect to the server and hardening means that the root account can no longer log in via SSH. The Python installation step alone is extremely important because Ansible requires Python to function. Older OS versions, like Ubuntu 14.04, already have the required version preinstalled. However, I recommend on running this task just to rule it out as a cause for errors.

The server_setup playbook handles the software provision side of the environment.  Be sure to fill out the variables listed within the playbook file itself. After that, be sure to read through all the roles and understand what's happening. The tutorial video above gives you a brief idea but it's worth it to be 100% sure what the apps are and why they're being installed.

### Troubleshooting your playbook

At some point, probably as your working through errors, you're going to want a second opinion in terms of playbooks or even consider making your own. Having a decent understanding of what sort of apps you need to install and why, will take you pretty far in figuring things out. For example, this playbook wants you to install Passenger to use as your app server. Say that your now getting errors from Ansible as it's trying to install or run Passenger. If troubleshooting/Googling the Ansible errors aren't getting you anywhere. My recommendation would probably go to the Ansible Galaxy website (link below) and look up some Passenger roles and see how other devs have done it. Sometimes I'll discover something and refactor my yaml a bit to fix my problem. If that doesn't help, then I'd look into roles for the Passenger alternatives and give that a shot instead. It's common to find that some playbooks out there are tailored for a specific environment and may not always instantly work on yours without putting any work into it. If your only deploying a small app like this to a small audience, it's mostly not a big deal what you use as long as it works. However, once you find yourself scaling up, it's time to hit the documentation and weigh the pros and cons of each alternative and go from there.

### Ansible Galaxy
https://galaxy.ansible.com/list#/roles?page=1&page_size=10

I did mention the Ansible Galaxy website above but I wanted add a huge caveat to it, especially as a Ansible newbie. At first it feels like the holy land, almost as if everything is done for you. For me, it's been more of a roller coaster ride when trying to find much use out of it. If you're like me, you may quickly find that perfect premade role and suddenly you feel as if all your prayers have been answered. Only then to discover after adding it to your playbook that it's over bloated and mostly overlaps with what you already have in place that works fine already. Deleting what you've poured sweat and tears into that you know works elegantly, only to then butcher it to make room for this strange code can put you into a tailspin quickly. 

I've found that the most popular roles, simple stuff like 'install sqlite3', often have dozens of additional unrelated stuff that overlap with your other roles. Some even lead you down rabbit holes to larger frameworks that absorb Anisble, like DebOps. To sum it up, if your looking to download roles to simplify your playbook development, it often ends up becoming more complicated as your time is spent untangling duplicate or unnecessary code. The trick that I've found is to instead spend that time looking for as many examples as possible. Not only does it expose you to more ways of developing roles, but it will also help you find the role that fits best with the playbook structure that your already developing. You'll eventually find something that either fits what your currently working on well enough to get your playbook to 100%, or you'll find a less popular playbook that lets you start at 95% but allows you to quickly tailor it to your environment.

Personally, I developed several rails playbooks and took them to what I felt was about 70% – 80% complete. I've taken others and broke them down and reorganized them to how I liked it and I've also started from scratch many times. After many days of research and frustrations trying to push them to 100%, I luckily found this playbook and this tutorial. I don't think it's perfect, but it allowed me to start at 98% of what I personally needed for this specific blog. Tailoring the rest to my environment was as simple as commenting out a few lines and editing a few variables. I also liked how the author organized the roles into largely separate sections. Unfortunately, it's been way too common for me to find playbooks that have everything dumped into one confusingly long role that has dozens of separate functions. This playbook is such a gold mine for beginners like me that I'll probably use this as my main rails template going forward and just add/edit whenever needed.

You may have notice that my Git repo currently has the Letsencrypt and nginx roles commented out for the server_setup playbook. This is because I'm self hosting my blog at the moment and using No-IP's service (dynamic DNS) to handle connections through my firewall. And since No-IP does not allow any SSL or CA attached to it's domain, I had to comment out those steps. However, I kept the code in there because I do plan on purchasing a proper domain at some point soon and Letsencrypt is probably what I'll end up using. 

Too be honest, I'm not entirely sure why this was split up into two separate playbooks. My guess is to aid in troubleshooting. The majority of issues you'd have into would be from running the server_setup playbook. Not having to run the presetup portion everytime as well could probably speed up troubleshooting a bit. Either way, if I had these playbook tailored down to exactly what I needed and worked every time, I'd probably consider stitching them together. 

#### Step 3. Capistrano

Once your Ansible playbook completes without any errors, it's time to move onto the second half of Chris's tutorial. This is where he goes over the rails app deployment tool by the name of Capistrano. This tool picks up where Ansible left off. It takes your app and plops it on your server and runs it. Following the tutorial seems pretty straightforward but I will go over some the issues that I ran into while working my way through the errors it gave me.

### The beauty of the Dotenv gem and .gitignore

One area of the tutorial that took me awhile to understand was the following task under namespace within the deploy.rb file. When I copied it down from the video, I gave it no thought nor did I know what it was all about. Turns out it was the key to solving a bunch of errors I was about to run into while teaching me a big lesson in security at same time.

``` Ruby
desc 'Upload .env'
task :upload_env do
  on roles(:app) do |host|
  upload! '.env', '/home/deploy/blog/.env'
  end
end
```
Learning the purpose of the above code took me down a very important path I am embarrassed to say that I didn't see coming before hand. You may notice that the above code is slightly different than what he has in the tutorial. When I used his code, I kept getting errors when running the Capistrano deployment. The first error was Devise complaining that it was missing it's secret key. In Googling how to solve that error, I found the solution is to copy the secret key right into your devise.rb file. Which is stored in plain text... in your GitHub public repo... This didn't feel like a great idea. And when I did so anyways and finished the Capistrano deployment, the deployment worked great but I received a generic error when trying to view my website within a browser. When I Googled that error, turns out the number one cause is not have your apps secret key base available for your app to read. Again, the solution was to store it in plain text within your app itself. The sudden requirement to publish sensitive data to solve security errors became very troubling to me. I decided to stop what I was doing and come up with an actual solution. 

This is when my research brought me to environment variables. There are a few ways to tackle this issue but my favorite is by using the dotenv gem. It makes the whole process very simple. After adding the gem to your gem list and running bundle, head to one of your files that has a secret key stored. I'll use the Devise example I mentioned above. Within the devise.rb file look for 'config.secret_key = fj29jf3j45rl3k4u892kd...' and replace it with 'config.secret_key = ENV['DEVISE_TOKEN_AUTH_SECRET_KEY']. Then create a file in the root of your blog repo named '.env'. Then place "DEVISE_TOKEN_AUTH_SECRET_KEY=fj29jf3j45rl3k4u892kd...' and save the file. The dontenv gem directs any text that starts with ENV to seek out the variable name within the following square brackets and matches it with the value stored in the '.env' file automatically. I found this to be a great way to store values you want to keep off a public GitHub profile. Stuff like your database credentials, API keys and whatever else you don't want openly published. Plus, in the Ansible/Capistrano video tutorial I linked above, if you skim around and pause it when he is looking at his Gemfile, you'll notice dotenv sitting in there. Things are starting to make sense!

But, your not quite done yet. The last step here is arguably the most important because without it, your defeating the whole purpose of that '.env' file. If your like me when I got to this point, you never really thought much about that '.gitignore' file. Turns out it's now your best friend. All you need to do is open that file up and add '.env' on it's own line, anywhere, and then save it. Doing this is what prevents you’re your beloved '.env' file, along with its secrets, from heading to GitHub. I'd also recommend making a copy of your '.env' file and backing it up somewhere since GitHub won't be able to save you if you lose it.

Now circling back to that ruby code I referenced above. Note that it's a task named "upload_env". It's whole purpose is to copy your '.env' file from your dev computer and then upload it into a directory on your app server. All I had to do was enter the pdw or the working directory to where I wanted the ".env" file to be placed. For some reason, the variables he has in the tutorial didn't work for me, but the pwd did. Running this task before the Capistrano app deployment command (cap production deploy) ensures that your app has access to your secret information beforehand, which then clears any security related errors during the deployment itself. You still get to enjoy the automation without having to make any secret stuff public.
  
#### Weird permission errors using Capistrano?

I was very confused when I first ran into this. Capistrano works by pulling your app down from your GitHub account. By default GitHub allows any machine who has ever pushed updates to a repo the access rights to use Capistrano to pull from that repo. You can see this info within your account personal settings under SSH and GPG keys. However, Capistrano will every once in awhile tell you that GitHub is blocking your computer. To fix this, copy and paste the following two lines into your shell.

```
eval "$(ssh-agent -s)" 
ssh-add ~/.ssh/id_rsa
```

### Capistrano deploying from the wrong Git branch?

If your like me, then you probably don't consider your master branch as production. Perhaps you prefer better organization and like to have a branch named 'production' and then use that branch for production. If so, your going to run into an issue while using Capistrano. By default, it always pulls from your master branch, even if you have another branch listed as default within your GitHub repo settings. Capistrano doesn't care. Only way I've found to solve this issue is by adding the following line of code into your deploy.rb

```
set :branch, ENV['BRANCH'] if ENV['BRANCH']
```

Doing this allows you to specify what branch you want to use when your running your Capistrano deployment. Since the branch I use to deploy to my server is called 'production', I use the command below to deploy to my server. 

```
cap production deploy BRANCH=production
```

Whatever branch you want Capistrano to deploy, just type the name of that branch after 'BRANCH=' and your good to go.

### Database exploding everytime you run a Capistrano deployment?

Capistrano makes it very easy to rapidly deploy updates to your web app. It was working great for me until I realized it was erasing my database on every deploy. Remember that few seconds in the Ansible/Capistrano tutorial I referenced at the top where he mentions linked directories? He quickly stated that he also uses a shared directory for his database but didn't go into much of an explanation as to why. At the time I heard that, I figured it was more an extra thing and I wasn't going to worry about adding things I didn't fully understand anyways. Well, I did some research on why all my content disappears whenever I update even the smallest parts of my web app and found out that Capistrano overwrites the DB directory on each deploy. This means that the database gets deleted and recreated each time it runs a rake db:migrate at the end of each Capistrano deploy. After more research, I discovered that a common solution is to tell your rails app, through the database.yml file, to look for your DB in directory outside your repo. Since the tutorial referenced the creation a linked directory in the shared folder outside my apps root, I figured that was my key to solving this issue. Once I made the same change that the guy did in the tutorial and told rails to store the DB there, all my problems disappeared. The only issue I have left is figuring out a way to automatically backup my DB file. I'll probably look into setting up a cron job at some point.
