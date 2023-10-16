+++
title = 'Creating and Maintaining a Free Blog in Github Pages, Hugo and Markdown'
date = 2023-10-15T12:25:49+01:00
draft = false
+++

## Context

As a Software Engineer, Programmer or Developer, sharing your knowledge and experiences with the tech community is a valuable way to give back and establish your expertise. Maintaining a blog is a great way to achieve this, and it can also serve as a reference point for your personal growth and a platform to demonstrate your skills. In this guide, we'll walk you through creating and maintaining a tech blog using GitHub Pages, Hugo, and Markdown. We'll explore the rationale behind these choices and provide you with step-by-step instructions for setting up your blog.

Why GitHub Pages, Hugo, and Markdown?

1. GitHub Pages
   GitHub Pages is a free, cloud-based service offered by GitHub that allows you to host static websites directly from your GitHub repositories. As a software developer, you're likely already using GitHub for version control and collaboration, making GitHub Pages a natural choice. It simplifies the hosting process and integrates seamlessly with your existing workflow.

2. Hugo
   Hugo is a popular static site generator written in Go. It offers incredible performance and flexibility, making it an excellent choice for tech blogs. Hugo allows you to create content in Markdown, which is a lightweight markup language widely adopted by the tech community. It enables you to focus on writing content while Hugo takes care of generating the site. This combination ensures a lightning-fast, highly customizable, and easily maintainable blog.

3. Markdown
   Markdown is a simple and intuitive way to format your content. It's easy to learn and allows you to focus on writing without worrying about the complexities of HTML. GitHub Pages and Hugo both work seamlessly with Markdown, making it a perfect choice for your tech blog.

In this post, we will show how we can create a personal blog for free using the above technologies.

## Setting Up Your Blog

### Prerequisites

Before we get started, make sure you have the following:

- **GitHub Account**: If you don't have one, sign up a [GitHub](https://github.com/) account.
- **Git Installed**: Download and install [Git](https://git-scm.com/downloads).
- **Hugo Installed**: Download and install [Hugo](https://gohugo.io/installation/).
- **Text Editor**: You can use any text editor, but Visual Studio Code is an excellent choice.

### Step 1: Create a GitHub Repository

1. Log in to your GitHub account and click the '+' in the upper right corner to create a new repository.
2. Choose a name for your repository (e.g.,"my-tech-blog") and select "Public" or "Private" based on your preference
3. Initialize the repository with a README and a .gitignore for Hugo:

```
.gitignore
resources/
```

4. Create the repository.

### Step 2: Set Up Hugo

1. Open your terminal and navigate to the directory where you want to store your blog project.
2. Run the following Hugo command to create a new site:

```
hugo new site my-tech-blog
```

3. Change the working directory to your blog project:

```
cd my-tech-blog
```

### Step 3: Choose a Hugo Theme

1. Explore available [Hugo themes](https://themes.gohugo.io/) at Hugo Themes.
2. Select a theme and add it to your project as a Git submodule:

```
git submodule add <theme-repository-url> themes/<theme-name>
```

3. Configure the theme in your site's **config.toml **file. This file may be initially empty, so you can create it and add the following:

```
baseURL = "https://username.github.io/my-tech-blog/"
languageCode = "en-us"
title = "My Tech Blog"
theme = "<theme-name>"
```

See [here](https://gohugo.io/) for quickstart guide and full documentation.

### Step 4: Create Your First Blog Post

1. Run the fo1. llowing Hugo command to create a new blog post:

```
hugo new posts/my-first-post.md
```

2. Open the newly created Markdown file (content/posts/my-first-post.md) and start writing your content.

### Step 5: Preview Your Blog Locally

1. Run the Hugo server to preview your blog locally:

```
hugo server -D
```

2. Visit http://localhost:1313/ in your web browser to see your blog in action.

### Step 6: Deploy to GitHub Pages

1. Create a gh-pages branch in your GitHub repository:

```
git checkout -b gh-pages
```

2. Push your site to GitHub Pages:

```
git add .
git commit -m "Initial commit"
git push origin gh-pages
```

3. Your blog will be available at https://username.github.io/my-tech-blog/.

### Step 7: Automate Deployment with GitHub Actions

While the above steps allow you to manually deploy your blog to GitHub Pages, using GitHub Actions can streamline this process and provide several benefits:

- Automation: GitHub Actions allows you to automate the build and deployment process whenever you push changes to your repository, making it hassle-free.
- Continuous Integration: You can set up CI/CD pipelines to ensure your site is always up to date and error-free.
- Custom Workflows: Customize your deployment workflows to include additional steps like running tests or optimizing assets before deployment.

To set up GitHub Actions for your Hugo-based blog, create a **.github/workflows/main.yml** file in your repository and define your deployment workflow. Here's a simple example:

```
name: Deploy Hugo site via GitHub Pages

on:
  push:
    branches:
      - main    # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04  # set ubuntu version

    steps:
       - uses: actions/checkout@v2
         with:
           submodules: true # Fetch Hugo themes (true OR recursive)
           fetch-depth: 0 # Fetch all history for .GitInfo and .Lastmod

      - name: Set up Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build site
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v
        if: github.ref == 'refs/heads/main'
        with:
          personal_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: public
```

This example triggers a deployment when you push to the **main** branch. It builds your site using Hugo and deploys the contents of the **public** directory to GitHub Pages.

Don't forget to add your GitHub token as a secret (named **GITHUB_TOKEN**) in your repository to allow the action to access your GitHub Pages.

With this GitHub Action in place, your blog will automatically update whenever you push changes to your repository, saving you time and ensuring your readers always see the latest content.

## Conclusion

Maintaining a tech blog using GitHub Pages, Hugo, and Markdown offers an efficient and customizable way to share your knowledge and experiences with the tech community. With the power of Git for version control, the simplicity of Markdown for content creation, and the automation provided by GitHub Actions, you can focus on what matters most: your content. Happy blogging!

Now, go ahead and start writing and sharing your tech adventures with the world. Your blog is ready to shine on the web, and GitHub Actions will ensure it stays up to date effortlessly!
