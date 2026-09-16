# Docs

Documentation should be completed in Markdown, though supplemental files may be permitted, e.g., for large code samples.

The project-root `README.md` is the documentation landing page. `docs/design.md` is the root of the language-design documentation.

Documents or anchors within documents may be referenced within documents in the `docs` directory.
Larger topics may be extracted into their own Markdown documents as long as they are referenced in the root document.

Documents and their anchors should be referenced according to these rules:
- Reference link paths should be relative file paths to other files in the `docs` directory. Do not link to files outside the `docs` directory.
- Reference links can be appended to relevant sentences before ending punctuation.
- Reference links should be enclosed in parentheses. The parentheses should not be part of the link display text.
- The links should use display text that matches the basename of the file, e.g., `[quasi-purity](purity/quasi-purity.md)`.
- Prefer linking specific anchors within the file over simply linking the whole file.
- You may also link anchors within the same file, but do not link the entire file as a reference within the same file.

You _may_ use spaces in Markdown file names for docs as appropriate; do no use punctuation like hyphens or underscores to replace spaces in multi-word file names.

# Notes

The notes directory is unorganized and random by choice as it is a place for exploring and recoding thoughts. Do not modify anything in the `notes` directory unless specifically asked to do so. When asked, strictly adhere to modifying what was requested and do not make unrequested modifications.
