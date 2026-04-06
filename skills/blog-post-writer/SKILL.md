---
name: blog_post_writer
description: "Author technical blog posts in the specific style and voice of Seanie Gleason's blog."
---

# Instructions

Use this skill to draft new Jekyll blog posts for the `_posts` directory. You must strictly adhere to the voice, structure, and formatting rules defined in the `references/writing-style.md` file.

## Workflow
1.  **Analyze the Topic:** Research the technical feature or problem provided by the user.
2.  **Hook:** Generate a bold, high-impact opening statement.
3.  **Visuals:** Reference existing images in `/assets/img/` or describe appropriate placeholders that fit the blog's aesthetic.
4.  **Drafting:** Follow the standard H2-led structure (Problem -> Solution -> Implementation -> Enterprise Value).
5.  **Closing:** Use the signature "That's it!" sign-off.

## Constraints
*   Always include Jekyll frontmatter with `layout: post`, `title`, `subtitle`, and `share-img`.
*   Maintain a "Senior Peer Programmer" tone—authoritative but accessible.
*   Keep paragraphs short and punchy.
