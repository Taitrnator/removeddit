# Removeddit
[Removeddit](https://removeddit.com) is a site for viewing removed stuff from [Reddit](https://www.reddit.com).
Just go to a reddit thread and change the `reddit` in the URL to `removeddit` to see all removed comments.

This is a done by comparing the comments found from Reddit API and comments being stored in [Jason Baumgartners](https://pushshift.io/) [Pushshift Reddit API](https://github.com/pushshift/api). The frontend is written in react with redux for state management. There's also a seperate [backend](https://github.com/JubbeArt/removeddit-api) used for storing removed threads and banned subreddits.

# Development
```
sudo git clone https://github.com/JubbeArt/removeddit.git && cd removeddit
sudo apt install npm
npm install
npm start
```

Visit http://localhost:8080 and make sure the site is running. Subreddits and the banned subreddit list will not work unless you also set up a backend. This might change in the future if I decide to open up my backend on removeddit.com/api. Contact me if you want this feature and we can discuss it. 

# Production
**THIS WILL BE UPDATED LATER WHEN THE PROJECT IN REWRITTEN**

Using [Ubuntu 16.04](http://releases.ubuntu.com/16.04/) and [nginx](https://www.nginx.com/resources/wiki/)

```
sudo git clone https://github.com/JubbeArt/removeddit.git /var/www/removeddit
sudo apt install -y nginx
sudo cp /var/www/removeddit/server-config/basic /etc/nginx/sites-available/default
```

Create a reddit app [here](https://www.reddit.com/prefs/apps/), select **installed app**. For "redirect url" it doesn't really matter in this case, you can pick `http://localhost`.

Copy the **client ID** for your app set it as a variable in `id.js`, e.g. with 
```
sudo nano /var/www/removeddit/static/js/id.js
# Insert with ctrl-shift-v
# Save with ctrl-o, exit with ctrl-x
```

Restart nginx and visit "localhost"
```
sudo service nginx restart
```

# The "I care about HTTPS and security" guide
In this part we'll set up [nginx](https://www.nginx.com/resources/wiki/) with SSL and set up a free renewing SSL certificates with [Let's Encrypt](https://letsencrypt.org/). I assume you've already done the guide above.

## Nginx.conf
Add the following to `/etc/nginx/nginx.conf` in the **http-block** (you can read about them [here](https://gist.github.com/plentz/6737338))
```
server_tokens off;
add_header X-Frame-Options SAMEORIGIN;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

In the same file you also want to change `gzip on` to `gzip off` (read more [here](https://github.com/h5bp/server-configs-nginx/issues/72)).

## Nginx server config
Copy the ssl config and create a soft link. Create folder for logs and also remove the default config
```
sudo cp /var/www/removeddit/server-config/ssl /etc/nginx/sites-available/removeddit.com
sudo ln -s /etc/nginx/sites-available/removeddit.com /etc/nginx/sites-enabled/removeddit.com
sudo mkdir /var/log/nginx/removeddit
sudo rm /etc/nginx/sites-enabled/default
```

Change the "server_name" for both server blocks in the config to your own domain name.

## SSL with Let's encrypt
Read the full guide [here](https://certbot.eff.org/#ubuntutyakkety-nginx). Start of by installing the Let's Encrypt client [certbot](https://certbot.eff.org/)
```
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install -y python-certbot-nginx 
```

Copy the Let's Encrypt config file for our site 
```
sudo mkdir /etc/letsencrypt/configs
cp /var/www/removeddit/server-config/letsencrypt.conf /etc/letsencrypt/configs/removeddit.com.conf
```

In this config file change the domains and the email address for your own. The emails tells you when the certificates are close to expiring.

This is when the webmasters start praying to God, for only He can deside the fate of the certbot. 
Forgive me Father, for I have committed.
```
sudo certbot --config /etc/letsencrypt/configs/removeddit.com.conf certonly
```

You'll now have a valid SSL certificate (hopefully)! You now have to uncomment the `ssl_certificate` and `ssl_certificate_key` in `/etc/nginx/sites-available/removeddit.com` and maybe change the path depending on where the certs are located (check in `/etc/letsencrypt/live/`).

## Automated renewal of certs
The certificate expires after 90 days so we want a way to atomatically update the certs.
There are multiple ways of doing this but I find the easiest to be [cron jobs](https://en.wikipedia.org/wiki/Cron).

First we'll test if renewing atcutally works with

```
sudo certbot renew --dry-run
```

If everything works fine we can create a cron with ```sudo crontab -e``` and select an editor your comfortable with (I like *nano*).
Add the following lines at the bottom

```
# Let's Encrypt cert renewal for all sites (runs every day at 04:30)
30 4 * * * certbot renew --post-hook "systemctl reload nginx"
```

Then just restart nginx and that should do it! 

This guide was mostly written for myself, you learn a shitton writing guides, highly recommended. Hopefully you learned something too.