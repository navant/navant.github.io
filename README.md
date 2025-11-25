# navant.github.io

Personal blog powered by [Hugo](https://gohugo.io/) with the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) (version 0.112.0 or later)
- Git

## Installation

1. Clone the repository:
```bash
git clone https://github.com/navant/navant.github.io.git
cd navant.github.io
```

2. Initialize the theme submodule (if not already done):
```bash
git submodule update --init --recursive
```

## Creating a New Blog Post

### Method 1: Using Hugo Command (Recommended)

```bash
hugo new content/posts/your-post-name.md
```

This will create a new markdown file in `content/posts/` with the archetype template.

### Method 2: Manual Creation

1. Create a new markdown file in `content/posts/` directory:
```bash
touch content/posts/your-post-name.md
```

2. Add the following front matter at the top of your file:

```yaml
---
title: "Your Post Title"
date: 2024-10-20T11:30:03+00:00
draft: false
tags: ["tag1", "tag2"]
author: "Navan Tirupathi"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "A brief description of your post"
canonicalURL: "https://navant.github.io/posts/your-post-name/"
disableHLJS: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>"
    alt: "<alt text>"
    caption: "<text>"
    relative: false
    hidden: true
editPost:
    URL: "https://github.com/navant/navant.github.io/tree/main/content/"
    Text: "Suggest Changes"
    appendFilePath: true
---
```

3. Write your content below the front matter using Markdown syntax.

## Front Matter Explained

- **title**: The title of your blog post
- **date**: Publication date (format: YYYY-MM-DDTHH:MM:SS+00:00)
- **draft**: Set to `false` to publish, `true` to keep as draft
- **tags**: Array of tags for categorization (e.g., `["Data", "AWS", "LLMs"]`)
- **author**: Author name
- **description**: Brief description for SEO and previews
- **showToc**: Show table of contents (true/false)
- **cover.image**: Path to cover image (place images in `static/` folder)

## Adding Images

1. Place your images in the `static/` directory:
```bash
cp your-image.png static/
```

2. Reference them in your markdown:
```markdown
![Alt text](/your-image.png)
```

Or use the cover image in front matter:
```yaml
cover:
    image: "/your-image.png"
    alt: "Description"
    hidden: false
```

## Local Development

### Start the development server:
```bash
hugo server -D
```

This will:
- Start a local server at `http://localhost:1313/`
- Auto-reload on file changes
- Include draft posts (remove `-D` to exclude drafts)

### View your site:
Open your browser to `http://localhost:1313/`

## Building for Production

Generate the static site:
```bash
hugo
```

This creates the site in the `public/` directory.

## Deployment

This site is configured for GitHub Pages. To deploy:

1. Make sure all changes are committed:
```bash
git add .
git commit -m "Add new blog post"
```

2. Push to GitHub:
```bash
git push origin main
```

3. GitHub Pages will automatically build and deploy from the `public/` directory (or as configured in your repository settings).

## Project Structure

```
.
├── archetypes/          # Content templates
│   └── post.md         # Default post template
├── assets/             # Custom CSS/JS
│   └── css/
│       └── extended/
├── content/            # Your blog content
│   └── posts/         # Blog posts go here
├── static/            # Static files (images, etc.)
├── themes/            # Hugo themes
│   └── hugo-PaperMod/ # PaperMod theme
├── hugo.toml          # Hugo configuration
└── public/            # Generated site (git ignored)
```

## Configuration

Edit `hugo.toml` to modify:
- Site title and URL
- Menu items
- Social links
- Theme settings
- SEO parameters

## Useful Commands

```bash
# Create a new post
hugo new content/posts/my-post.md

# Start development server with drafts
hugo server -D

# Start development server without drafts
hugo server

# Build the site
hugo

# Check Hugo version
hugo version

# Clean the public directory
rm -rf public/
```

## Writing Tips

- Use descriptive titles and tags for better discoverability
- Add a clear description in the front matter for SEO
- Use headers (`##`, `###`) to structure your content
- Include code blocks with syntax highlighting:
  ````markdown
  ```python
  def hello():
      print("Hello World")
  ```
  ````
- Keep drafts as `draft: true` until ready to publish

## Troubleshooting

**Issue**: Changes not showing up
- **Solution**: Stop the server (Ctrl+C) and restart with `hugo server -D`

**Issue**: Images not displaying
- **Solution**: Make sure images are in the `static/` folder and referenced with a leading `/`

**Issue**: Theme not loading
- **Solution**: Update submodules: `git submodule update --init --recursive`

## Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [PaperMod Theme Guide](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Markdown Guide](https://www.markdownguide.org/)

## License

This blog content is © Navan Tirupathi. The PaperMod theme is licensed under MIT.
