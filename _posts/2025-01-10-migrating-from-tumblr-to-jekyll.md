---
title: "Migrating from Tumblr to Jekyll: A Fresh Start"
date: 2025-01-10
layout: post
description: "Reflections on moving my blog from Tumblr to Jekyll after an eight-year hiatus."
tags: [jekyll, tumblr, hacking]
---

It has been eight years since my last blog post. In that time, life has taken me on quite the journey—I moved to a new state, my wife and I welcomed two wonderful kids into the world, and I’ve found myself celebrating a sabbatical that provides fresh perspective. With the new year upon us, I’m ready to break that long blogging hiatus, and what better way to start than by refreshing my entire blogging platform? My first step: migrating from Tumblr to Jekyll.

Why the switch? Tumblr served me well for years, but I’ve always been concerned about its future and the lack of control I had. Jekyll’s open-source nature offers the customization I need for future projects, not to mention the improved performance benefits and the simplicity of hosting the blog on GitHub Pages for free.

Here’s a brief guide on how I managed the migration—and some lessons I learned along the way.

### Migrating Content from Tumblr to Jekyll

Jekyll provides an official [importer tool for Tumblr](https://github.com/jekyll/jekyll-import). This tool converts your posts, images, and metadata into Markdown files suitable for Jekyll. Follow the documentation to set up the importer and migrate your content.

While the import process works well, I noticed that some images were imported at lower resolutions. To preserve quality, I manually downloaded higher-resolution versions from Tumblr.

### Hosting with GitHub Pages
One of the major advantages of switching to Jekyll is free hosting with GitHub Pages. Jekyll integrates seamlessly, allowing you to deploy your blog without managing a server. Follow GitHub’s [documentation for GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages) to publish your site.

### Migrating Disqus Comments
Preserving comments was another challenge. Disqus offers [migration tools](https://help.disqus.com/en/articles/1717068-migration-tools) that include exporting your comments and updating URL mappings for your new Jekyll site. This process ensures your comment history remains intact.

### Conclusion
Migrating from Tumblr to Jekyll is a rewarding process, especially if you value the control and performance that static site generators offer. While it took some extra effort to perfect the import and handle comments, the overall process was smooth and well-documented.

Good luck with your migration! 🚀

