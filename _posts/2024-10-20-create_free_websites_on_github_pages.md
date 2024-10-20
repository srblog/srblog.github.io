---
layout: post
title:  "Create Free Websites on GitHub Pages"
date:   2024-10-20
tags: website GitHub github-pages DNS 
---

[GitHub Pages](https://docs.github.com/en/pages/quickstart) provides an effective way to host static web pages from GitHub repositories. You can link a custom domain to your GitHub page to create a professional site without a separate host. This guide explains how to set up a GitHub page and connect a GoDaddy domain. 

# Creating a GitHub Page

- Log into GitHub and create a _public repo_ named `<username>.github.io`, using either your personal or organization's GitHub `username`.
- You can simply add a `index.hmtl` with any associated CSS files and you are ready to go. 
- The site will be live at `https://username.github.io`
- GitHub supports static generator [Jekyll](https://jekyllrb.com/) to create some professional sites for free. See how to [add a Jekyll theme](https://docs.github.com/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll) to your site.

# Adding a Custom Domain

- Purchase a domain (say `mydomain.com`) using a domain manager like GoDaddy.
- In the _Domain Dashboard_ select the domain you want to point to the GitHub Page.
- Edit the _apex domain_ `A` record with the _Name_ **@** in it and edit the IP address to the following four IPs:

```bash
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
``` 

- You can check the DNS using a Linux bash shell using the command `dig`:
  - `dig mydomain.com +noall +answer -t A`

- Edit the `CNAME` (`www` subdomain) record with the _Type_ `www` and change the pointer to `<username>.github.io` and _Save_ it.

- You can check the DNS again: 
  - `dig WWW.MYDOMAIN.COM +nostats +nocomments +nocmd`

- In the GitHub repo, navigate to `Settings -> Pages -> Custom domain` and add the purchased domain eg. `www.mydomain.com` and _Save_ it.

- Allow for a little time for the DNS to propogate and now your custum domain (`www.mydomain.com`) should point to the GitHub Page (`<username>.github.io`)
  

# References

- [Creating a GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)
- [GitHub Pages: An Introductory Tutorial](https://builtin.com/software-engineering-perspectives/github-pages) 
- [Configuring a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)
- [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
- [About custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)
- [Verifying your custom domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages)
