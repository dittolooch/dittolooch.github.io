---
layout: post
title: Using Custom Domain From Godaddy with Github Pages in 2019
---
## Github Pages & Jekyll

[Github Pages](https://pages.github.com/) is a static site hosting service designed to host your personal, organization, or project pages directly from a GitHub repository. It fully supports [Jekyll](https://jekyllrb.com/), the static site generator powered by Ruby, so that you don't need to worry setup a database to store your contents. You can follow this [awesome post](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/) written by Barry Clark if you need an easy-to-follow guide to help you setup the page.
<!--more-->

## Custom Domain with Github Pages

I had spent couple hours figuring out how to use my domain from GoDaddy with Github Pages before I finally got it up and running. Some tutorials could be found with a bit of Googling but due to GoDaddy's constant changes of their user interfaces, it just wasn't that straightforward. So here's a step by step:

1. Sign in to GoDaddy and click on the small icon on the upper right corner of the screen, and that'll bring up a drop-down menu

2. Click "My Products"

3. On the main page of "My Products", under "Domains", select the domain you want to use for the Github Pages and click "Manage"

4. Scroll down and click "Manage DNS"

5. Here comes the tricky part: open up your terminal and enter
```
$ dig YOUR_GITHUB_USER_NAME.github.io
```

6. You should see 4 IPs returned (as of April 2019). Github might change the setup so please just take notes of the IPs that were returned.

7. Get back to GoDaddy's Manage DNS page and in Records, you click "Add" and create as many A records as you saw on terminal in step 6. <strong> MAKE SURE YOU REMOVE EVERY PRE-EXISTING DNS RECORDS AND START FRESH</strong>

8. The A record format should be like this: Type = A, Host = @, Points to = ONE_OF_THE_IP_ADDRESSES, TTL=1 Hour; as of April 2019, you should create 4 A records

9. Create 2 CNAME records
  * Host: YOUR_DOMAIN
  * Points to: www.YOUR_DOMAIN
  * TTL: Custom 600
  * Host (second CNAME): www
  * Points to (second CNAME): YOUR_GITHUB_USER_NAME.github.io
  * TTL (second CNAME): Custom 600
  * <strong>So in my case...</strong>
  * Host: warrencheng.dev
  * Points to: www.warrencheng.dev
  * TTL: Custom 600
  * Host (second CNAME): www
  * Points to (second CNAME): jn8029.github.io
  * TTL (second CNAME): Custom 600

10. Wait for 5-10 minutes and open up your terminal and enter
```
$ dig YOUR_DOMAIN
```
You should see the returned IPs matching those of the new A records you just added in step 9, and also the CNAME that matches the CNAME you just added.

11. Then try another dig
```
$ dig www.YOUR_DOMAIN
```
The result should be the same as step 10. When I was working on this, I didn't add the 2 CNAME records and somehow my www.MY_DOMAIN and MY_DOMAIN (one with the triple w and one without) are pointing to different IPs! Using 2 CNAME records solved my problem.

12. Go to your Github Pages repository and click "Settings"

13. Under "Github Pages", enter your custom domain in the input field "Custom domain"

14. Click save and refresh the page, the "Enforce HTTPS" option will be available within couple minutes, you can come back later and tick "Enforce HTTPS"

15. With step 14, Github should automatically help you add a CNAME file in your repo with your custom domain in it.

That's it. Now you can go to your custom domain and it'll take you to your Github Pages.

Enjoy.
