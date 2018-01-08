[![Build Status](https://travis-ci.org/googlei18n/fontdiff.svg)](https://travis-ci.org/googlei18n/fontdiff)

# fontdiff

`fontdiff` is a utility for testing fonts. When you modify a TrueType
or [OpenType](http://opentypecookbook.com/index.html) font, fontdiff
generates a PDF showing the typeset text both before and after the
change. You can use this PDF to easily review the changes and spot any
errors caused by a font switch.

For every line in the text sample, the tool renders two raster images
in high resolution. One image is typeset using the original font, and
the second uses the new font.  If there is any difference between
these two high-resolution images (even if it’s just whitespace caused
by kerning changes), the output PDF highlights the difference in
color.

![Screenshot](doc/fontdiff-1.png)

Since fontdiff is using
[HarfBuzz](https://www.freedesktop.org/wiki/Software/HarfBuzz/) for
OpenType shaping, text gets rendered in a similar way across Firefox,
Chrome, Android, LibreOffice, XeTeX, GNOME, KDE, ChromeOS, and other
products.


## Sample text

To provide sample text, use HTML with embedded CSS styling:
```html
<html lang="la">
  <p>Lorem <span style="font-weight:700">ipsum</span> dolor sit amet.</p>
</html>
```

The `lang` tag can make a difference for rendering. For example, when
diffing a font with [Polish letter
forms](https://glyphsapp.com/tutorials/localize-your-font-polish-kreska),
you can write `<span lang="pl">ćńóśź</span>` to see the Polish
variants of accented letters, or `<span lang="und">ćńóśź</span>`
to see the default design.  As with HTML, you can combine spans in
multiple languages into the same document.  fontdiff recognizes the
same [language tags](https://www.w3.org/International/articles/language-tags/)
as modern web browsers; “und” is the “undefined” or default language.


## Limitations

This is not an official Google product. We needed a lightweight tool
to test our font production pipeline, so we built fontdiff, and we hope
you’ll find it useful, too.  However, please understand that there are
some limitations:

* **Not a full renderer:** HTML and CSS are much richer than what this
  little testing tool supports. If you miss something, please do not
  hesitate to make the change yourself and send a pull request.

* **Lousy codebase:** The current state of the codebase isn’t something
  we’d be proud to call production-ready: no tests, too many dependencies,
  and so on. fontdiff isn’t a “real” product, and definitely not a showcase
  for writing production code.

* **Not secure:** Do not run this tool on untrusted input. It parses
  complicated input formats (TrueType, OpenType, HTML) but it isn’t
  hardened in any way.  Use it at your own risk.

* **Not a sample generator:** This tool compares how an existing
  sample text gets rendered in two versions of the same font. However,
  it won’t find text for you to render.  Some font designers may want
  to manually craft samples. Others may want to see how an entire
  volume of text is rendered, or check the output of a font-affecting
  tool or script.  Because the use cases for fontdiff are wildly
  different, we felt it would be best to limit its scope to the actual
  diffing operation.


## Building

```sh
$ git clone --recursive https://github.com/googlei18n/fontdiff.git
$ cd fontdiff
$ ./src/third_party/gyp/gyp -f make --depth . --generator-output build src/fontdiff/fontdiff.gyp
$ make --directory build
$ build/out/Default/fontdiff \
    --before path/to/old/FooFont*.ttf \
    --after path/to/new/FooFont*.ttf \
    --specimen src/fontdiff/specimen.html \
    --out out.pdf
```

This incantation works on reasonably recent versions of Linux and MacOS X.
If you want to support other platforms, feel free to make the change.
Pull requests are very welcome!


## Contributing

Here are a few features and concepts we’d love your help implementing,
if you’re bored and/or want to get your hands dirty (or are just a
magnanimous person who loves fontdiff):

* **Port to other platforms.** This should be a very easy starter task
  for people familiar with the respective platforms. It shouldn’t
  take more than a few hours, since the project already uses
  cross-platform libraries and a cross-platform build system.

* **Testing.** Currently, the codebase is entirely untested, which makes
  it hard to maintain the code.

* **More constructs from CSS and HTML.**

* **MathML.** With (even limited) support for MathML rendering,
  font designers could use fontdiff to test their mathematical fonts.

* **Sandboxing.** To protect users against attacks that use malicious
  font or text sample files, set up a
  [sandbox](https://en.wikipedia.org/wiki/Sandbox_%28computer_security%29)
  early in the tool’s execution and then do all the rendering inside
  the sandbox.

* **CSS box model.** It might be nice to properly support the layout of
  CSS and HTML, where boxes can be inside boxes. Not sure if this is
  terribly important for testing/diffing fonts, but if this tickles
  your interest, go ahead.

* **Paragraph layout.** The current paragraph layout is rather
  ad-hoc. On the positive side, fontdiff uses
  [ICU](http://site.icu-project.org/) for finding potential line
  breaks, so it can be used in testing fonts for Thai and
  other languages that don’t mark word boundaries. However, there is
  currently no hyphenation or justification, and the current support
  for bidirectional text is rather iffy. One option for doing this
  properly could be [Raqm](https://github.com/HOST-Oman/libraqm),
  another might be
  [Minikin](https://android.googlesource.com/platform/frameworks/minikin/+/master). If
  you want to look into this, make sure to think about the special
  needs for diffing fonts. In particular, a changed glyph width should
  not require a re-flow in the entire paragraph — the
  resulting diffs would become meaningless. When fontdiff breaks
  paragraphs into lines, it currently measures the width of text runs
  using _both_ the old and the new font version. The decision about
  line breaking is then made based on the maximum of both these
  values, which means fontdiff doesn’t run into this reflowing
  problem.  It will probably be difficult to replicate this
  logic when using an existing paragraph layout library. However,
  being able to precisely locate the difference between two fonts is
  important for font designers, which is why we wrote
  the tool in the first place.

* **Reduce binary size.** To avoid dependency hell and to maintain
  reproducible versioning, fontdiff is statically linked to all its
  dependencies.  This is a very intentional choice, but ICU’s data
  files are _huge_.  By building a static ICU data library with only
  the needed data files (the break iterators), the size of the
  compiled fontdiff binary would shrink by about 20MB. If you are
  interested in this task, check out how
  [NodeJS](https://github.com/nodejs/node/blob/master/tools/icu/README.md)
  does it. This task shouldn’t be very difficult, but it’s probably also
  not the biggest problem fontdiff needs to solve.

Have fun!
