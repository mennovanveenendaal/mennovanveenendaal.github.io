---
title: "Building a website using Github pages"
layout: post
categories: [Github Pages, Setup]
image:
  path: /assets/2024/building/github.png
  alt: GitHub
---
Although I have some experience with building websites, I had never published one before. To streamline the process and minimize the time required to get a site live, I opted to use GitHub Pages and Jekyll. This combination allows for quick deployment of a static website, while keeping it simple and free.


## 1. Creating a New Repository
The steps to get started are outlined in https://pages.github.com/, but here's a quick rundown of what I did:
1. **Create a Repository:** I created a new repository named `username.github.io`—the `username` part must match your GitHub username. For example, if my username was `user1`, the repository would be named `user1.github.io`.
2. **Clone the Repository Locally:** Next, I installed Git using [this link](https://git-scm.com/download/win), and then cloned the repository to my local machine. To do this, I opened a command prompt in the folder where I wanted to store the repository and ran:
  `git clone https://github.com/user1/user1.github.io`.

## 2. Adding Initial Content
To display anything on the website, an index file (either `.html` or `.md`) is required. I chose to create a simple `index.html` file. Here's how I did it:
1. I navigated to the repository folder:
   `cd user1.github.io`
2. Created an `index.html` file with basic content:
   `echo "Hello World" > index.html` 
3. After creating the file, I added, committed, and pushed the changes to the GitHub repository using the following commands:
```shell
git add --all
git commit -m "<commit comment>"
git push -u origin main
```

With that done, browsing to `user1.github.io` showed my "Hello World" message.

## 3. Adding Styling with Jekyll
GitHub Pages integrates seamlessly with [Jekyll](https://jekyllrb.com/), a static site generator. It simplifies the process of setting up a website by offering themes, layouts, and other features out of the box. The [official documentation](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll) explains how to use Jekyll for GitHub Pages.

You can either build a Jekyll site from scratch or use a pre-made theme. I chose to use an existing theme, which is a time-saver for those who don't want to start from scratch.

### Finding and Applying a Theme
I found a suitable theme on [Jekyll Themes](https://jekyllthemes.io/) and downloaded it. Afterward, I added the theme’s files to my local repository, making sure to edit the `_config.yml` file as per my requirements. The `_config.yml` file controls the settings for the Jekyll site, including site title, theme configuration, and other options.

### Updating the Site
Once the theme was in place and customized to my liking, I added the changes to Git, committed them, and pushed them to GitHub:

I downloaded a theme and places the content in my repository folder. After altering the `_config.yml` file I pushed the files to github:
```shell
git add --all
git commit -m "Added Jekyll"
git push -u origin main
```

With that, my website had styling and a theme in place!
## Final Thoughts
Building a website using GitHub Pages and Jekyll is an efficient way to get a static site up and running with minimal effort. The steps are straightforward, and the process of adding themes or even creating a fully customized Jekyll site offers flexibility and control over the look and feel of your website.