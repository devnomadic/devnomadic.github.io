# Staging Directory

This directory contains markdown post files that are being staged for preview before publication.

## Structure:
- `staging/` - Contains draft/preview post files
- `_posts/` - Contains published post files

## Workflow:
1. During preview builds, new post `.md` files are copied here
2. Jekyll builds the site with both `_posts/` and `staging/` content
3. After review and approval, posts are moved from `staging/` to `_posts/`
4. Staging directory is cleaned up after merge
