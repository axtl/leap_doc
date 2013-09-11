LEAP DOCUMENTATION
=================================

Files in the directory "docs" show up automatically at https://leap.se/docs when this repository is pushed.


Directory structure
---------------------------------

Every directory corresponds to a single web page. Within each directory, there are source files for different languages.

For example:

    docs/
      client/
        en.haml    -- used for https://leap.se/en/docs/client
        es.haml    -- used for https://leap.se/es/docs/client

If you don't care about translating a page, and it has no children, you can omit the directory:

    docs/
      client.haml  -- user for https://leap.se/*/docs/client

Menu
---------------------------------

A page does not show up in the navigation unless it appears in menu.txt.

The order in menu.txt determines the order in the navigation.

Supported syntaxes
---------------------------------

Depending the the file extension, the file with be parsed like so:

    .haml       -- HAML
    .md         -- Markdown
    .markdown   -- Markdown
    .txt        -- Textile
    .textile    -- Textile
    .rst        -- ReStructuredText

Markdown is rendered using RDiscount, Textile is rendered using RedCloth, and RST is rendered using docutils. Markdown is recommended, because it supports table of contents, although the markup is limited.

There are a couple options to preview your source files without needing to run the web server:

* Markdown preview for Chrome: https://chrome.google.com/webstore/detail/markdown-preview/jmchmkecamhbiokiopfpnfgbidieafmd
* Markdown preview for Sublime: https://github.com/revolunet/sublimetext-markdown-preview
* Markdown preview for Firefox: https://addons.mozilla.org/de/firefox/addon/markdown-viewer/  (see https://addons.mozilla.org/en-US/firefox/addon/markdown-viewer/reviews/423328/ for rendering .md file extensions)

Setting page properties
---------------------------------

HAML files are rendered as templates, but the other lightweight markup files are treated as static files.

The one exception is that every file can have a "properties header". It looks like this:

    @title = "A fine page"
    @toc = false

    continue on here with body text.

The properties start with '@' and are stripped out of the source file before it is rendered. Property header lines are evaluated as ruby. All properties are optional and they are inherited, including `@title`.

Available properties:

* `@title` -- The title for the page, appearing as in an H1 on the top of the page and as the HTML title. Also used for navigation title if `@nav_title` is not set. The inline H1 title does not appear unless `@title` is explicitly set for this page (i.e. the inherited value of `@title` does not trigger the automatic H1).
* `@nav_title` -- The title for the navigation to this page, as well as the HTML title if @title is not set.
* `@toc` -- If set to `false`, don't include a table of contents when rendering the file. This only applies to .rst and .md files.
* `@layout` -- Manually set the layout template to use for rendering this page.
* `@author` -- The author credit for the page.
