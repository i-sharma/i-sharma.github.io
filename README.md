# Knowledge Dump for Future Me

This document contains a brief overview of how to work with this Jekyll-based blog.

## Development

To run the blog locally, use the following command:

```bash
bundle exec jekyll serve
```

## Git Workflow

- The `shishan` branch is the primary development branch. All new work should be done here.
- The `master` branch is the live branch that is deployed to the website. Merge changes from `shishan` to `master` to publish them.

## Content

### Posts

- Blog posts are located in the `_posts` directory.
- The filename format is `YYYY-MM-DD-title-of-post.md`.

### Future Posts

- Ideas for future posts are kept in the `future_posts` directory.

### Images

- Images used in posts are stored in `assets/img`. It's a good practice to create a subdirectory for each post within `assets/img/posts` to keep things organized.

## Tags

Tags are used to categorize posts and are linked to the main menu.

### How to Add/Change a Tag

1.  **Create the Tag Page**:
    - In the `_featured_tags` directory, create a new Markdown file (e.g., `new-tag.md`).
    - The content of the file should be:
      ```markdown
      ---
      layout: list
      title: The Display Name of the Tag
      slug: the-slug-for-the-tag
      ---
      ```
    - The `slug` is what you will use in the post's front matter.

2.  **Add to Menu**:
    - Open `_config.yml`.
    - Add a new entry to the `menu` list:
      ```yaml
      - title: The Display Name of the Tag
        url: /tag-the-slug-for-the-tag/
      ```
    - The URL is constructed as `/tag-<slug>/`.

3.  **Use in Posts**:
    - In the front matter of a post, add the `slug` to the `tags` list:
      ```yaml
      ---
      title: My Awesome Post
      tags: [existing-tag, the-slug-for-the-tag]
      ---
