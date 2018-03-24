---
title: "How to get a free SSL certificate for your GitHub Pages site"
description: "In the first article on this site, I described a way to host your
    site or blog on GitHub Pages while having it accessible via your own domain.
    I left the article with one thing left unsolved&#58; the site was not
    accessible via HTTPS, and something like that would not fly in <em>current
    year</em>. Let us see how one can fix that – for free."
tags: [web]
---

While GitHub Pages issues a free SSL certificate for you if you access your site
via the `github.io` domain, it unfortunately cannot (yet?) generate a
certificate for your custom domain. Well, worry not, this is easily solvable by
using [Cloudflare](https://www.cloudflare.com) (also known as Buttflare to the
users of the extremely useful
[Cloud to Butt](https://chrome.google.com/webstore/detail/cloud-to-butt-plus/apmlngnhgbnjpajelfkmabhkfapgnoai)
Chrome extension :peach:).

Cloudflare is many things: it's a reverse cache proxy, a content delivery
network, a DNS provider, it saves you traffic by compressing and minifying your
site, it protects your site from DDoS attacks and content scraping, modifies
your site on the fly with Cloudflare applications and it even prepares the
coffee in the morning for you :coffee:. Also, I might be lying about the last
thing.

Anyhow, without further ado, here's how I did it for my site.

## Setting up Cloudflare

Like any proper IT fairytale, it all started with registering a new account with
[Cloudflare](https://www.cloudflare.com). After you're done with that, you'll be
taken to a configuration wizard that will lead you through the steps necessary
for Cloudflare to start taking care of your domain.

Once you fill in your domain name, your existing DNS records will be analysed
and transferred over to Cloudflare. I'd recommend paying careful attention
during this step as Cloudflare's auto-detection does fairly good, yet not
perfect job.

In my case, Cloudflare was able to find apex domain's `A` record, apex domain's
and `mail` & `www` subdomain's `CNAME` records, all `MX` records and apex
domain's `SRV` and `TXT` records. Everything else you'll have to configure
manually; I recommend going through your old DNS records, which you can likely
find in your domain registrar's administration interface, and transferring them
one by one over. [Vivaldi](https://vivaldi.com)'s split screen view is
absolutely amazing for this.

After you're done, Cloudflare will assign you two name-servers (these are
different for each domain so I can't just write the addresses of my name-servers
here for you) and with the knowledge of these addresses, you'll have to go back
to your domain's registrar's administration interface and configure Cloudflare's
name-servers as your domain's primary and secondary name-servers. As far as I
can tell, Cloudflare (or for that matter, anyone else) doesn't really care about
which one of them is primary and which one is secondary.

Once Cloudflare is successfully set up for your domain, you can head to their
administration panel, into the *Crypto* section, and verify that the *SSL*
option is set to *Full*. You may also want to scroll down and turn *Always use
HTTPS* to *On*, *Automatic HTTPS Rewrites* to *On* and if you care about your
site's [SSL Labs rating](https://www.ssllabs.com/ssltest/)[^1], you may want to
go through the configuration of the *HTTP Strict Transport Security (HSTS)*
option; however, be **incredibly careful** with the last one, read through all
the warnings Cloudflare shows you and verify that Cloudflare already issued a
SSL certificate for your domain (issuing may take a few minutes, and after all,
you *just* signed up for the service -- so if the certificate is not ready yet,
just be patient and return in a while :crocodile:) and that HTTPS is working
correctly for your domain. I recommend first verifying that everything truly
works, waiting a few days during which you test just HTTPS alone and if
everything seems fine and dandy, go for the HSTS option. In order to score an
A+ grade on the test, you need to set HSTS's *Max-Age* attribute to at least
6 months.

[^1]: If you want to improve your SSL Labs rating even further, you should add
    `CAA` DNS records for `comodoca.com`, `digicert.com` and `globalsign.com`
    issuers by following [this short tutorial](https://support.cloudflare.com/hc/en-us/articles/115000310792-Configuring-CAA-Records-).

And that should do the trick! I truly recommend going through the entire
administration panel at this point -- there is a ton of hidden and useful
features. In fact, let me briefly mention my favourite ones.

## Additional perks

- The entire *Analytics* section is sweet, and complements Google Analytics
    nicely;
- *DNS* &#8605; *DNSSEC* allows you to further secure your domain, but support
    on your domain's registrar's side will also be needed; which is why I
    transferred my domain recently away from IGNUM to
    [Namecheap](https://www.namecheap.com), which so far seems to be an amazing
    (and cheap!) registrar that I can wholeheartedly recommend;
- *DNS* &#8605; *CNAME Flattening* is an always-on feature that allows you to
    *not* specify an `A` record for your apex domain and instead use a `CNAME`
    record[^2];
- *Speed* &#8605; *Auto Minify* reduces the size of your HTML, CSS and
    JavaScript files by getting rid of whitespace;
- *Speed* &#8605; *Brotli* further compresses your site's traffic;
- *Speed* &#8605; *Rocket Loader™* takes all your JavaScript files, bundles them
    into a single file and still somehow doesn't break your site in the process;
- and finally, *Scrape Shield* &#8605; *Email Address Obfuscation* encodes all
    e-mail addresses found on your site and injects a JavaScript to decode them
    back before displaying them to your visitors.

[^2]: For example, I have a `CNAME` record on this apex domain that points to
    `cellane.github.io`, and it works just fine and apparently resolves even
    faster than an `A` record -- how is that possible? Personally, I suspect
    black magic!

Pretty amazing for a free service, isn't it? There's just one last thing to
mention...

## On the topic of cache

By employing Cloudflare to cache your site, you may see some delay before newly
published articles or pages appear to your visitors. Or better said, you would
be able to see some delay if I didn't have a handy solution ready.

In my
[previous article on this topic]({{ site.baseurl }}{% link _posts/2018-03-06-how-to-host-your-site-for-free-using-github-pages.md %}),
I mentioned something about cache purging -- and cache purging is exactly what
you need to do to indicate to Cloudflare that new content is available on your
site. You can do so by calling
[appropriate API method](https://api.cloudflare.com/#zone-purge-all-files),
preferably as a part of your site's build process, exactly as I mentioned in the
previous article. You can find your domain's zone ID in the *Overview* section
while your API key hides in *My Profile* which you can access by clicking on
your e-mail address in the top right in Cloudflare's administration panel.

Congratulations on having secure and, unless you went absolutely nuts with
Cloudflare's applications, fast website! :stuck_out_tongue_winking_eye:
