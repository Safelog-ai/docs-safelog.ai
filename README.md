# Safelog.ai Documentation

This repository contains the documentation for Safelog.ai background check services and integration APIs.

## GitHub Pages Setup

This documentation is designed to be hosted on GitHub Pages with Jekyll.

### Local Development

To run the documentation locally:

1. Install Jekyll and dependencies:
   ```bash
   gem install jekyll bundler
   ```

2. Navigate to the docs directory:
   ```bash
   cd docs
   ```

3. Install dependencies:
   ```bash
   bundle install
   ```

4. Start the development server:
   ```bash
   bundle exec jekyll serve
   ```

5. Open your browser to `http://localhost:4000`

### GitHub Pages Deployment

1. Push the `docs` folder to your repository
2. Go to repository Settings > Pages
3. Select "Deploy from a branch"
4. Choose the branch containing your docs folder
5. Select "/docs" as the folder
6. Save the settings

The documentation will be available at `https://your-username.github.io/your-repo-name/`

## Structure

```
docs/
├── _config.yml          # Jekyll configuration
├── _layouts/            # Custom layouts
│   └── default.html     # Main layout with sidebar
├── index.md            # Homepage
├── integration/        # Integration documentation
│   ├── index.md        # Integration overview
│   ├── partner-integration.md  # Partner integration guide
│   └── webhook-spec.md # Webhook specification
└── README.md           # This file
```

## Features

- 📱 Responsive design that works on desktop and mobile
- 🧭 Sidebar navigation with hierarchical structure
- 📖 Clean, readable documentation layout
- 🔗 GitHub integration for easy updates
- 🎨 Professional styling with Safelog.ai branding

## Content Management

### Adding New Pages

1. Create a new `.md` file in the appropriate directory
2. Add Jekyll front matter:
   ```yaml
   ---
   layout: default
   title: Page Title
   parent: Section Name (optional)
   nav_order: 1
   ---
   ```
3. Write your content in Markdown

### Navigation Structure

The sidebar navigation is automatically generated based on:
- Files in the root with `nav_order`
- Parent-child relationships using `parent` front matter
- Custom navigation defined in `_layouts/default.html`

## Support

For questions about the documentation:
- Create an issue in this repository
- Contact the development team
- Email: support@safelog.ai