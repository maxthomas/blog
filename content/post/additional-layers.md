+++
Categories = [ "ssl", "ssh", "encryption" ]
Description = "Adding more security to both https and ssh connections to a server."
Tags = [ "ssl", "nginx", "encryption", "ssh", "https" ]
date = "2017-04-03T23:31:19-05:00"
title = "Additional layers of https and ssh security"

+++

After [moving to the new blog host](moving-digital-ocean) and scoring
only a `B` on the [SSL report](https://www.ssllabs.com/index.html), I
wanted to make sure I was doing everything I could do secure the
server from exploits. Part of the reason for moving was exactly that -
increased security! On my journey to improve the score, I learned a fair
bit about SSL, server and browser based attacks, and how to mitigate them.

I also did a bit of digging into how to secure my personal connection
to the VPS over `ssh`. I'll cover some information on that at the end of this
post.

## Weakness is unacceptable

Turns out that by default, `nginx` allows weaker versions of cryptographic
technology that can be more easily penetrated by nefarious users. Specifically,
an attacker can eventually intercept traffic between a browser and server.
More reading, including a technical paper and mitigation, can be found
[here](https://weakdh.org/).

For now, we'll focus on mitigation. Luckily enough, the same site
has some [information for sysadmins](https://weakdh.org/sysadmin.html).
Let's get started.

### Generating a better DH group

The above group is convinced that state-level actors are capable of
penetrating 1024-bit groups. It wasn't clear whether that is by sheer computing
power or exploits. Nevertheless, the group recommends using at least a
2048-bit length group.

I figured that doubling wouldn't hurt anything, so chose 4096 bits
as the length. Running this command generates the `.pem` file we need
to give to `nginx`.

``` shell
openssl dhparam -out dhparams.pem 4096
sudo mv dhparams.pem /path/inside/your/jail
```

While we're editing `nginx` configuration, I'll also suggest that you
enable Strict Transport Security (HSTS). It is helpful in preventing
the same encryption downgrade attacks.

I added both HSTS and `dhparams` info to the `nginx` configuration
file in the jail I created before. Mostly taken straight from their
sysadmin suggestion page:

``` conf
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
add_header Strict-Transport-Security "max-age=31536000";

ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';

ssl_prefer_server_ciphers on;
ssl_dhparam /path/inside/your/jail/dhparams.pem;
```

Afterwards, restart nginx:

``` shell
sudo service nginx reload
```

This was enough to get me to an `A+` on the SSL test. Skeptical
readers can
[run it themselves](https://www.ssllabs.com/ssltest/analyze.html?d=maxthomas.io&hideResults=on).
Again, I'm not entirely sure how well qualified they are to give that
grade - but as long as the grade is improving, things can't be getting
worse...  right?

## Hardening ssh

Now that the `https` part of the server was mitigated against most
attacks, I wanted to see if there was anything I was missing on the
`ssh` side of things.  Knowing that OpenSSL is basically full of
vulnerabilities and odd nuggets (here's one of the
[latest ones](https://guidovranken.wordpress.com/2017/01/28/can-you-spot-the-vulnerability/)
I've seen), I figured the defaults weren't good enough.  And,
according to
[at least one guy on the 'net](https://stribika.github.io/2015/01/04/secure-secure-shell.html),
I was right!

The information there is great. There's also a very brief
justification for most of the changes, enough to satisfy a sysadmin
without needing a ton of security background. Those paying attention
will notice the same information about weaknesses in DH that was
covered earlier, in the `https` part of the post.

A small note: in the section about enabling `Protocol 2`,
you'll be asked to accept the new fingerprint (as the blog says).
Even if you skip this step, however, you'll likely be asked
to accept the new fingerprint. The scary warning that pops up
might make you think your server has been compromised,
but it's just using the `ED25519` keys instead of the old,
normally `RSA` keys.

That's not to guaranteed your server *wasn't* compromised, of course,
but you're probably fine if it happened in between updating your
configuration.

## Other avenues

Right now I can't add too much to the linked resources,
due to time constraints. I think a few things left unsaid,
such as [port knocking](https://en.wikipedia.org/wiki/Port_knocking), are
also worth considering. I'm considering adding that to my setup -
expect a future blog post on its pros and cons to come.
