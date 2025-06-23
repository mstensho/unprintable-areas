# Expose unprintable areas via CSS

See https://github.com/w3c/csswg-drafts/issues/11395 for initial proposal.

Most printers have a small region along each edge of the page sheet which is
unprintable, due to the printer's paper handling mechanism. See
https://drafts.csswg.org/css-page-3/#page-terms

The size of these unprintable areas are available to applications (such as
browsers) in most OSes, but currently it's not web-exposed. This means that
authors have no means of confidently setting page margins to prevent content
from getting clipped. This becomes a more obvious shortcoming for `@page` margin
boxes for e.g. custom headers and footers. See
https://drafts.csswg.org/css-page-3/#margin-boxes

We could use an env() variable for this.
https://drafts.csswg.org/css-env-1/#safe-area-insets may serve as an
inspiration. However, this is for non-rectangular displays, such as those with
rounded corners on mobile phones. Such displays may even have holes for cameras,
microphones and speakers. Furthermore, the device is capable of providing
reliable values for each of the four edges of the screen.

Some printers don't have the same unprintable area inset along each of the four
paper edges, but the printers may rotate the print output at their own
discretion. Example: the paper is fed with the short edge first into the
printer, i.e.  portrait mode, but the pages to be printed are in landscape
mode. Will the printer rotate the output clockwise, or counter-clockwise to make
it fit on paper? There's no way of telling. Therefore, only one value can be
reliably provided here: The larger of these four values.

The currently proposed name for this is `env(safe-printable-inset)`.

Example:

```css
<!DOCTYPE html>
<style>
  @page {
    margin: calc(env(safe-printable-inset) + 50px);
    margin-bottom: calc(env(safe-printable-inset) + 50px);
    margin-left: calc(env(safe-printable-inset) + 50px);
    margin-right: calc(env(safe-printable-inset) + 50px);

    @top-center {
      content: "";
      border: solid;
      margin-top: env(safe-printable-inset);
    }
    @bottom-center {
      content: "";
      border: solid;
      margin-bottom: env(safe-printable-inset);
    }
    @right-middle {
      content: "";
      border: solid;
      margin-right: env(safe-printable-inset);
    }
    @left-middle {
      content: "";
      border: solid;
      margin-left: env(safe-printable-inset);
    }
    @top-left-corner {
      content: "";
      width: 30px;
      height: 30px;
      border: solid;
      margin-bottom: auto;
      margin-right: auto;
    }
    @top-right-corner {
      content: "";
      width: 30px;
      height: 30px;
      border: solid;
      margin-bottom: auto;
      margin-left: auto;
    }
    @bottom-right-corner {
      content: "";
      width: 30px;
      height: 30px;
      border: solid;
      margin-top: auto;
      margin-left: auto;
    }
    @bottom-left-corner {
      content: "";
      width: 30px;
      height: 30px;
      border: solid;
      margin-top: auto;
      margin-right: auto;
    }
  }
</style>

Content.
```

If there's no unprintable area (when creating a PDF, for example), it would look like this:



If the unprintable area width is 16px, on the other hand, it would result in a print layout like this:



On paper it would look like this (if the printer doesn't lie too much about its capabilities):



One concern here is printers with large unprintable areas along one edge, and
more "reasonable" values along the other three edges.
