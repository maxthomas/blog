+++
date = "2017-03-26T16:05:23-05:00"
title = "Moving the blog host to DigitalOcean, FreeBSD, and nginx"
Description = "Describing the process of moving from GitHub pages to DigitalOcean, FreeBSD, and nginx inside a jail."
Categories = ["digital-ocean", "freebsd", "ssl"]
Tags = [ "digital-ocean", "freebsd", "jails", "ssl", "lets-encrypt", "nginx", "gandi" ]

+++

**Important: This blog post contains an affiliate link for
DigitalOcean.**  *To try to prove I'm not fishing for your dollars,
you'll have to find it yourself!*

*While I'm not in it for the money, I stand to gain a small credit (as
do you) if you click the affiliate link located on this page. If you
choose to get started with DigitalOcean through that link, I'd certainly
be very appreciative.*

[TL;DR link to how-to](post/moving-digital-ocean/#action)

## Little Green Locks

Today I decided to investigate what it would take to add an SSL
certificate to the blog.

I had been using [GitHub Pages][gh-pages] to host the blog. This is a
great service offered by GitHub that allows an individual to commit
files to a `git` repository and serve these out to the web. Because
this blog is powered by [Hugo][hugo], which generates `.html` files,
essentially the preferred file format of a web browser, it is
naturally very easy to generate these files and have GitHub Pages
serve them up.

Normally, when hosted in the `github.io` domain (the default), GitHub
Pages provides an `https` connection automagically -
[here's an example](https://michael-myers.github.io/blog/).
Unfortunately, this blog is hosted under my custom domain, which
prevents me from using GitHub's SSL certificate. Obviously, this is
completely unacceptable for a security-conscious web site. More
on why in a later post.

As far as I could tell, this could be remedied in at least two ways:

1. Purchase an SSL certificate through my registrar, [Gandi][gandi]
   (or via many other companies)
2. Move to a server under my (remote) control, e.g. a VPS on
   [DigitalOcean][do-referral],
   allowing me to set up [Let's Encrypt][lets-encrypt] for free

Option 1                                 | Option 2
---------------------------------------- | ----------------------------------------
Can be easily done through Gandi's site  | Requires set up of Let's Encrypt
No control/management of server          | Full control of/management of/responsibility for server
DNS still handled through Gandi          | Need to switch DNS to DigitalOcean, including domain mail
$18 USD / year (Gandi)               | $60 USD / year (DigitalOcean)
Not that much work                       | Way more work
Nothing new to learn                     | Way more new things to learn

I chose option 2 because it would allow me to something I've always
wanted an excuse to work on: learning how to use FreeBSD, jails, and
nginx. Plus, it'll be way more *fun*.

## FreeBSD and its philosophy

I've always been intrigued by the [BSD family of operating systems][bsd].
For the less technical audience, you can think of BSD as yet another
operating system, similar to Windows, OS X, or GNU/Linux. In fact,
[Apple's OS X shares a lot of code with BSD](https://en.wikipedia.org/wiki/MacOS#Development).

More technical readers might appreciate a core difference between BSD
and GNU/Linux, which are often lumped together in the *nix-like
domain.  As far as I can tell, there are two main idealistic
differences between GNU/Linux and BSD, and several quasi-myths that
I could find little or no evidence for:

1. BSD seeks to provide both the kernel and the initial application
   layer that actually makes the kernel useful. Thus, GNU/Linux:
   one cannot really have "just linux" or "just GNU", but one can
   clearly have, say, just FreeBSD or just OpenBSD.
2. Licensing. There's a lot of e-ink spilled on this topic,
   and there may be more from me about this in future posts; for now,
   the TL;DR is that GNU/Linux has chosen the GPL,
   and FreeBSD has generally [not](https://en.wikipedia.org/wiki/FreeBSD#License).

Less clear differences include:

1. BSD is faster, more performant, etc.: Seems impossible to test
   this claim on its own merits - how would one set up such a test? -
   but many anecdotes of less memory usage have
   cropped up around the tubes.
2. BSD does releases better: given that BSD is both the kernel and
   a core set of tools, it behooves them to have a bit more polished
   releases - the viability of the OS depends on it. Again, though,
   these broad claims require a bit of careful analysis and thought.
3. Security: Many will claim BSD is more secure. Again, a somewhat wild claim,
   not really empirically testable apples to apples without some serious thought.
   What is clear, however, is that
   [OpenBSD really prioritizes security](https://en.wikipedia.org/wiki/OpenBSD#Security_and_code_auditing).
   It's less clear that FreeBSD does the same, though they may benefit
   from OpenBSD's attitude (and thus, security updates).

### Throw your apps in jail

One major **similarity** between the two exists in their approach
to virtualization. FreeBSD uses a concept of [jails][jails] to provide
virtualization to applications. GNU/Linux has a similar feature,
[LXC](https://en.wikipedia.org/wiki/LXC), utilized by tools like Docker.
Many think these two are distinct, but in reality, the mechanisms
are quite similar. More on this in a future post as well.

The FreeBSD approach to this problem is the use of jails. You
can think of jails as Docker-containers built inside of the FreeBSD
operating system itself. It's a bit more nuanced, obviously, but
good enough for now.

In any event, software/systems engineers at large
have started to appreciate things like virtualization
at the level of applications. Docker's popularity has
[absolutely exploded](https://deis.com/blog/2015/containers-future-docker/),
now containing a massive ecosystem of images, applications, and
support from higher-level applications like Marathon. And for good
reason: isolated environments prevent entire servers from going down,
provide an additional layer of security between application exploits
and root access on the OS, and provide modular ecosystems for
applications.

The awesome benefit of added security is one of the main selling
points for jails. If, say, there's a security hole in the program
you're using to serve up web pages to browsers, the attacker will also
have to penetrate the virtual environment that you've (hopefully) set
up. While this is not impossible, it is far superior to the
non-partitioned alternative, where compromising the web server would
result in an attacker owning the box.

## nginx, the new King of the Hill?

In a bygone era - about 10 years ago - the Apache web server
was the silent workhorse of the internet. Serving up LAMP stacks
(and even WAMP stacks!!) all over the internet, the Apache
web server project was arguably one of a few flagship projects
that helped power some of the world's most popular web pages.

Nowadays, there's a new kid on the block: [nginx](http://nginx.org/).
One of the project's main goals was to beat up on the Apache
web server's apparent poor performance numbers, especially
with regards to memory usage.

As someone interested in new technologies - especially more performant
ones - I decided trying out `nginx` would be something worth doing.
Due to its popularity, `nginx` is now becoming the target of many
security researchers and hackers alike. It'll be interesting and
useful to learn a bit more about this super-popular web server.

## Free SSL from Let's Encrypt

To obtain the actual SSL certificate for the server, I'll be
using [Let's Encrypt][lets-encrypt]. It has emerged as an organization
with great interest in promoting `https`, aka "the green lock", across
much of the internet: essentially anyone with their own server can
register and get an SSL certificate for their web site - for free!
They are also making openness a priority. As someone interested in
security and privacy, that is something I'm absolutely on board with.
Interested readers can get more information
[here](https://letsencrypt.org/how-it-works/).

## Action

In this section I'll document the process I used to go from
GitHub Pages to a FreeBSD Digital Ocean droplet, running nginx
inside of a jail.

I'll treat this as a dumb server serving up `.html` files.  Therefore,
the server won't have `hugo`, `go`, or anything else used to actually
generate the site. Think of this droplet/server as a Docker container
that serves up images from a directory via `nginx`. Less software on
the server means less opportunity for exploits.  Simple and clean.

I'm doing this all from Arch Linux. I'll assume you've got
SSH keys and are comfortable with the command line. If you
aren't, just read through the docs on DigitalOcean - which,
by the way, has *excellent* documentation (so far, anyway).

### Start the FreeBSD droplet

First, you need to sign up for DigitalOcean. If you want to support
me, you can use [this link][do-referral], or you can use their
web site directly.

Start a droplet with the FreeBSD operating system. You'll need to
add your SSH public key, as the `root` user will not have
password authentication/SSH authentication enabled. All options
are optional; the only one I checked was `Enable IPv6`. Put
`blog` in the name and/or tag it appropriately if you expect
to use DigitalOcean a lot.

Once running, you can test your setup like so:

``` shell
ssh freebsd@your-droplet-ip
```

For now, close the connection after you ensure it works.

### Handling DNS

Next up is transferring DNS from your current provider to D.O. If
you're using Gandi, or a host of other registrars, you'll certainly find
[this link](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars)
useful. Gandi allows you to postpone this to a date in the future,
which is probably wise - otherwise, you might have a brief outage in
email delivery, web site/blog being unavailable, and things like
that.I recommend saving this until **after** completing the
DigitalOcean setup, but because the next document suggests doing it
now, I thought I'd cover it here.

In any event, take a look at
[this document](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)
to map the proper DNS entries to DigitalOcean. Make sure you've gotten
all of the ones from your current DNS provider into the droplet.

Do not forget to add your mail records if you use a custom mail server
for your domain's email address! Forgetting this can seriously hose
your email, and you may have delayed/bounced emails from others
sending mail to your domain.  For anyone running a business from their
domain and relying on email, this is **crucial** to get right. Double
check the `mx` records, especially before making the switch.

### Setup FreeBSD with a new user, jails, and a firewall

We'll now switch gears again and head back to the command line.
`ssh` into your droplet.

#### Optional: Install modern shell

One interesting note is that FreeBSD defaults to the use
of `csh`. That is absolutely insane! I also installed
`zsh` (yes, I'm a hypocrite about installing stuff):

``` shell
sudo pkg install zsh
```

I then made this the default shell of my own user (see below).

#### Optional: non-default user

Using the `freebsd` user is OK, but it's a small security issue as it
gives an attacker a (perhaps) guaranteed user-name to target. I
recommend creating a new user that you'll use to interact with your
droplet. Whether or not you decide to do this is up to you.

Copy-pasta for doing this:

``` shell
sudo adduser
# follow prompts
# when prompted, add user to wheel group, OR:
# wait until after and run:
sudo pw groupmod wheel -m max      # replace 'max' with your user in next commands
sudo mkdir /home/max/.ssh
sudo cp ~/.ssh/authorized_keys /home/max/.ssh/
sudo chmod 700 /home/max/.ssh
sudo chmod 600 /home/max/.ssh/authorized_keys
sudo chown -R max:max /home/max
sudo vim /usr/local/etc/sudoers    # ugh, vim
```

After opening that file, uncomment this line towards the bottom:

``` shell
%wheel ALL=(ALL) NOPASSWD: ALL
```

If your `vim` is rusty, remember: `ctrl-p`/`ctrl-n` to navigate,
hit `i` to start to edit (??!?!?), then `esc-wq!` to save-exit.

### Set up firewall, jail, nginx

Follow
[this excellent post](https://www.kirkg.us/posts/how-to-configure-a-freebsd-jail-on-a-digital-ocean-droplet/)
to set up this stack. Huge props to that author for that post.

### Get the SSL cert set up

[Let's Encrypt recommends](https://letsencrypt.org/getting-started/)
[this tool](https://certbot.eff.org/) for getting up and running.
I gave it a shot as of course I have shell access.

[This site](https://certbot.eff.org/#freebsd-nginx) is the version
we'll want to use. There's no plug and play for FreeBSD.
Bummer. For some reason I'm not surprised.

I chose to install the python stuff via `pkg` as the ports thing
didn't work for me. I think it was because I didn't have `ports`
configured - or something like that - or maybe it's straight up
broken. Unfortunately, this installed a bunch of `python2.7` gunk.
I'll be looking into how to do this better soon.

Note: I'm running these commands **outside** the jail. The firewall
prevents running these commands inside the jail, I think; the software will
time out, leaving you wondering WTF is going on (it did for me, anyway).

Here we go:

``` shell
pkg install py27-certbot
sudo certbot certonly --webroot \
    -w /usr/jails/YOUR_JAIL/usr/local/www/blog \     # or whatever
    -d maxthomas.io \
    -m 'your@email.com' \
    --staging \                                      # you can easily get rate limited on 'prod'
    --config-dir /usr/jails/YOUR_JAIL/usr/local/etc/letsencrypt
```

If that works, run without the `--staging` flag.  Note the locations
of your stuff; you'll need them later when editing the `nginx` config.

### Editing nginx config

After heading back into your jail:

``` shell
vim /usr/local/etc/nginx/nginx.conf
```

[This post](http://serverfault.com/questions/67316/in-nginx-how-can-i-rewrite-all-http-requests-to-https-while-maintaining-sub-dom#337893)
claims that the best way to do redirects is to have two
`server` blocks. I followed along and create the two blocks,
one for the redirect and the other for my actual site.

Add the following lines for your ssh cert in the second block; obviously,
your domain path will differ, so update it accordingly:

```
  ssl_certificate /usr/local/etc/letsencrypt/live/my.domain.com/fullchain.pem;
  ssl_certificate_key /usr/local/etc/letsencrypt/live/my.domain.com/privkey.pem;
```

I also changed the location/root from `/usr/local/www/nginx`
to something else as I was not sure if the former was potentially
going to go away with package updates. There is, hilariously, a
file called `EXAMPLE_DIRECTORY-DONT_ADD_OR_TOUCH_ANYTHING`, so I
followed its instructions and didn't do anything in there.

Back to the shell to copy error pages and the index.

``` shell
mkdir /usr/local/www/blog
cp /usr/local/www/nginx/*.html /usr/local/www/blog/
service nginx restart          # if all went well, no config errors
```

You should now be able to hit `http://yourdomain` and get auto-redirected
to the `https` version. Slick! Unfortunately, this only got my website a
`B` grade on the [SSL report](https://www.ssllabs.com/index.html).
I have no idea if that grade is worth the disk it's stored on; more on
that in the future as I try to improve the grade.

I didn't yet try to automate the renewal with `cron`. I'll likely update this
post with information about how to do that, but at present, I'm ready
to get this thing over with.

### Move, Move, Move

Well, we've now got the server up and running, but our `.html`
files are conspicuously absent.

As stated earlier, I want this to be a dumb server that just
serves up `.html` files. As a result I'll be `rsync`ing the contents
of the generated page over to my node, then moving those into the jail.
I'll probably automate this in the future, but for now:

```
# on the computer with hugo installed
hugo new post/blahfooblah.md
# write something half-decent for all of our sakes
hugo
# now have a bunch of .html files and so forth
rsync -raz --progress doc/ server:blog/                                   # pick your own directories
ssh server
sudo rsync -raz ~/blog /usr/jails/YOURS/usr/local/www/yours/blog/blah     # fix paths yourself
```

Assuming all your paths line up, you should now be able to hit the
link to your blog and have it served up. Now mix yourself ~~a Black
Russian~~ a cup of ginger tea and enjoy your new droplet, SSL
certificate, and blog host.

Well, there you have it: about a day's work learning about
DigitalOcean, DNS, `nginx`, FreeBSD, jails, firewalls, Lets Encrypt,
and a bunch of other stuff. I'm pretty satisfied with the setup, in
all honesty: it's exactly what I want, a jailed `nginx` that serves
up content I'm producing on another machine. Look out for future
updates about automating the certificate renewal as well as automating
content movement.

[do-referral]: https://m.do.co/c/85c974583b0b
[gh-pages]: https://pages.github.com/
[gandi]: https://gandi.net
[lets-encrypt]: https://letsencrypt.org/
[hugo]: https://gohugo.io
[bsd]: https://en.wikipedia.org/wiki/Berkeley_Software_Distribution
[jails]: https://en.wikipedia.org/wiki/FreeBSD_jail
