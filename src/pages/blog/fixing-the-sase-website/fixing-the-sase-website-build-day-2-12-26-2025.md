---
layout: ../../../layouts/BlogPost.astro
title: "Fixing the SASE Website - Build Day 2"
date: "2025-12-26"
description: "Troubleshooting WordPress installation issues on HostGator - PHP execution and DNS problems"
dayNumber: 5
---

left off at troubleshooting why `/wp-admin`  page doesnt pop up

---

## troubleshooting

2. Check if WordPress actually installed

hostgator —> cPanel —> file manager

![HostGator file manager showing public-html folder](/blog-images/fixing-the-sase-website-build-day-2/image.png)

there is a `public-html`  folder!

Look for WordPress files:

- `wp-config.php`
- `wp-admin` folder
- `wp-content` folder

they all exist

3. Check for .htaccess issues

1. Go to `public_html/.htaccess` in File Manager
2. Rename it to `.htaccess_backup`
3. Try loading `http://sasepurdue.com/wp-admin` again

this did not work lol

---

so far we know:

- not an SSL issue
- not an .htaccess issue

- `http://sasepurdue.com/wp-admin` ❌
- `https://sasepurdue.com/wp-admin` ❌
- No new `.htaccess` is being generated
- You only see `.htaccess_backup`
- HostGator claims SSL is active

then **WordPress is not being served from the directory your domain points to**.

This is a **HostGator document root / domain mapping issue**, which is *very common*.

---

**Find the REAL document root**

![HostGator cPanel showing document root](/blog-images/fixing-the-sase-website-build-day-2/image%201.png)

i think the document root is `home/public_html` …

this is not the problem

---

Given what you just showed, the problem is **not DNS, not addon domains, not SSL, and not document root confusion**.

👉 The problem is that **Apache/PHP is not executing WordPress at all inside `public_html`**, even though the domain is correctly mapped there.

This narrows it to **one of only three real causes on HostGator**. We’ll eliminate them in order.

---

**Does PHP run at all?**

1. cPanel → **File Manager**
2. Open `/public_html`
3. Create a new file: **`test.php`**
4. Paste this:

```php
<?phpecho"PHP IS WORKING";?>
```

1. Save
2. Visit:

```
http://sasepurdue.com/test.php
```

anddddd

php did not execute :c

![PHP test showing server not executing PHP](/blog-images/fixing-the-sase-website-build-day-2/image%202.png)

---

so this is NOT a wordpress issue

---

we’re trying hostgator support chatting

i chatted with this person named pooja lol

she said it was a DNS issue and fixed it for us

apparently we just have to wait now

.

.

.

we will revisit this tomorrow