---
title: "Setting up the Chirpy theme"
layout: post
categories: [Github Pages, Chirpy]
image:
  path: /assets/2024/themes/chirpy.png
  alt: Chirpy
---
After setting up my initial website, I used a theme I found on the [Jekyll Themes website](https://jekyllthemes.io/). While it worked, it lacked some features I wanted, like smaller images on the homepage and an index. After searching for alternatives, I found [Chirpy](https://chirpy.cotes.page/) and decided to switch. The installation process was straightforward, though I did encounter a few minor issues.


# Switching to the Chirpy Theme
Initially, I tried replacing the theme by downloading the repository from GitHub and swapping the folder in my local repository. It worked for the first theme I used, so I thought it would work again. Unfortunately, this didn’t work with the Chirpy theme.

To resolve this, I followed the official guide and used the [Chirpy starter](https://chirpy.cotes.page/posts/getting-started/). From the Chirpy GitHub page, I clicked `Use this template`, then `Create a new repository`. The new repository needs to be named after your GitHub username.

# Setting up the Environment
For faster testing and site previews, I wanted to run the site locally. I chose to set up the environment [natively](https://jekyllrb.com/docs/installation/) by installing [Ruby](https://www.ruby-lang.org/en/downloads/) and [RubyGems](https://rubygems.org/pages/download).

To verify the installations, you can use the commands `ruby -v` and `gem -v`.

In the folder where I stored my website files, I cloned the newly created repository using `git clone`. Next, I installed all the dependencies for Chirpy by running the `bundle` command from the root directory of the repository. I encountered an error with the `wdm` gem: `An error occurred while installing wdm (0.1.1), and Bundler cannot continue.`. This could be fixed by modifying the `Gemfile` by changing the line `gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]` to `"wdm", "~> 0.1", :platforms => [:mingw, :x64_mingw, :mswin]`. After this, the installation completed successfully.

# Running the Jekyll Server
With everything installed, I could preview the site locally. I ran `bundle exec jekyll s` from the local root folder of my website, and the server was accessible at [http://127.0.0.1:4000](http://127.0.0.1:4000).

# Configuring the site
The `_config.yml` file contains the general configurations for the website. I customized the following settings: timezone, title, description, URL, github, socials and avatar.

To update the "About" page, I edited the `about.md` file in the `_tabs` folder. For contact settings, I modified the `_data/contact.yml` file, either commenting out options or removing the comments entirely.

Lastly, I replaced the site’s favicons. The original icons were located in the `_site/assets/img/favicons/` folder. I created new ones, renamed them to match the existing favicon naming convention, and replaced the files. After rebuilding the site, the new favicons appeared.

# Setup GitHub repository
![github](/assets/2024/themes/github.png)
_Fig.1 Github settings_

A few minor adjustments are needed in the GitHub repository settings. First, navigate to **Settings**, then to **Code and automation** and select **Pages**. Under the **Build and deployment** section, change the **Source** to "Deploy from a branch."

Next, set the **Branch** to "main" and choose **(root)** for the directory. Don't forget to save the changes.

# Conclusion
Once the setup was complete, I uploaded the code using the following commands:
```shell
git add --all 
git commit -m "<Commit comment>" 
git push -u origin main
```
With that, my new site was live! The process was straightforward, and I’m really happy with the final result, especially with how it looks and the local testing options after installation.



