# python


# Install mkdocs and material theme
pip install mkdocs mkdocs-material

# Optional: Install useful plugins
pip install mkdocs-git-revision-date-localized-plugin mkdocs-git-committers-plugin-2

# install mermaid plugin
pip install mkdocs-mermaid2-plugin


# Serve locally for development
mkdocs serve

# for live reloading
mkdocs serve --livereload

# Build the site
mkdocs build

# Deploy (if using mkdocs gh-deploy)
mkdocs gh-deploy