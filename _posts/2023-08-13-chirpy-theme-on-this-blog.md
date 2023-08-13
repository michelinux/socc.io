---
layout: post
title: Chirpy Theme on this blog
date: 2023-08-13 09:11 +0200
toc: true
---

## Going for Chirpy

Before Jekyll + Chirpy, I evaluated `hugo` and Wordpress. I almost went with Wordpress, but engineers are attracted to complexity as moth to the light. At the beginning I was a bit doubtful about Chirpy because the Getting Started page is not mentioning that one could also install it using `gem`. It took me longer than I care to admit, but of course you can use `gem`, and it's clearly written in the README.

I won't add anything that is not already well documented about `jekyll`.

## Change Date Format

> I later realized that this trick is messing with the download of some js libraries. I have
> yet to find the right way of doing it. In the while I will revert it and accept the ugly
> American date format.
{: .prompt-danger}

I prefer reading the day before the month (sorry US people!). The Chirpy theme does not have a `en_GB`. I cannot contribute it, I am not British and I feel it would not be appropriate to provide that contribution from a non-native. So I have instead added the `en_michele`.

It’s basically just the following:

```bash
mkdir -p _data/locales
wget https://raw.githubusercontent.com/cotes2020/jekyll-theme-chirpy/master/_data/locales/en.yml -O _data/locales/en_michele.yml
sed -i .bak 's/strftime: "%b %e, %Y"/strftime: "%d %b %Y"/' _data/locales/en_michele.yml
rm _data/locales/en_michele.yml.bak

sed -i .bak 's/lang: en$/lang: en_michele/' _config.yml
rm _config.yml.bak
```

> The `bak` in `-i bak` is because I am on Mac and the `sed` is a bit sadder than on Linux.
{: .prompt-info}

Clean up and restart your jekyll to check

```bash
jekyll clean
jekyll serve
```

Commit your changes.

```bash
git add _data/locales/en_michele.yml _config.yml
git commit -m 'fix: date format'
```

## Adding LinkedIn contact

Head to the `_data/contact.yml` and add something like:

```yaml
- type: linkedin
  icon: 'fab fa-linkedin'                      # icons powered by <https://fontawesome.com/>
  url:  'https://www.linkedin.com/in/soccio'   # Fill with your Linkedin homepage
```

In a similar way you can add something for the share under each article.

## Automating GitHub Actions

### Mac vs Linux

If you are running on Mac like me you need to make sure your `bundle` can also run on Linux:

```bash
bundle lock --add-platform x86_64-linux
```

From [this question on StackOverflow](https://stackoverflow.com/questions/65082929/how-do-i-fix-this-error-your-bundle-only-supports-platforms-but-your-local).

### Publishing on your domain with SFTP

I don't want to dedicate my whole domain just to the blog. I'm hosting a few other minor things, and I just want to `ftp` stuff from time to time.

Instead I upload the generated content via `sftp`. (My hosting plan does not include `ssh`.)

I found the best solution being using [wlixcc/SFTP-Deploy-Action](wlixcc/SFTP-Deploy-Action): simple enough that it's almost convenient than me writing the `sftp` command. In these cases I always prefer to _outsource_, despite what [Robert Lefkowitz may think of it](https://learning.oreilly.com/videos/technical-debt-a/0636920338031/0636920338031-video327221/).

### Automatic publishing of content from other repositories

I have a private repository with some content. GitHub Actions is not available on private repositories. But nothing prevents you from using the actions of a public repo to automate stuff on a private one. 

I want to copy from my private content repo to the folder `content` in [socc.io](http://socc.io).

I need to use private keys to do that, because it’s a private repo.

First thing, create new keys with:

```bash
ssh-keygen -t ed25519 -C 'mc@socc.io' -f id_ed25519
```

This generates two files:

```
id_ed25519
id_ed25519.pub
```

1. Open the _Settings_ of your private repository and add the **public** key in the _Deploy Keys_ section.

2. In the repository running the action, add a new _secret_ called `PRIVATE_KEY_CONTENT_REPO` which will contains the **private** key.

This setup is very similar to the setup you have when you want to `ssh` into a machine: you add the public key in the `~/.ssh/authorized_keys` of the _destination_ account and your private key is safely stored in the `~/.ssh` folder of the source account.

#### References

- https://parkerrobert.medium.com/clone-private-repository-for-github-actions-of-another-repository-ada04dc0e195
- https://stackoverflow.com/questions/57612428/cloning-private-github-repository-within-organisation-in-actions

## Copyright License

I'm OK for the CC BY 4.0 that is used by default on Chirpy. However it seems the only way to change it is to update the locale file, as I explained above for the date format. Confirmed on [Issue 972](https://github.com/cotes2020/jekyll-theme-chirpy/issues/972).

## Markdown in titles

I tried to add a `_layouts/post.html` copying it from Chirpy and using what suggested here: https://stackoverflow.com/questions/40470161/markdownify-post-title-in-jekyll.

```html
<h1 data-toc-skip>{{ page.title | markdownify | remove: '<p>' | remove: '</p>'}}</h1>
```

It works, but the result is not good: the `code` part is written much smaller than the rest of the title. There is some work to be done in the css, which for now I won’t do it. It could be a nice addition though.

## The _Time Ago_ date format

There are some website using an old version of Chirpy where you can see a very elegant IMHO _x days ago_.

![2023-08-13-jekyll-chirpy-x-days-ago](/assets/img/2023-08-13-jekyll-chirpy-x-days-ago.webp)

The example is taken by the [blog of Sandor Dargo](https://www.sandordargo.com). If you are interested in C++, pay it a visit.

Unfortunately this functionality is gone, [confirmed by Chirpy's author](https://github.com/cotes2020/jekyll-theme-chirpy/issues/1170). There are ways to bring it back, especially if you didn't use the `gem` way to install it. Just have a look at commit [6d35f5f](https://github.com/cotes2020/jekyll-theme-chirpy/commit/6d35f5f8da044cfad071628bb53776de03efaae4).
