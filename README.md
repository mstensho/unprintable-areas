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

If there's no unprintable area (when creating a PDF, for example), it would look
like this:

![image](https://github.com/user-attachments/assets/94bc5e1b-ad04-46aa-ba85-5a1d824c4221)

If the unprintable area width is 16px, on the other hand, it would result in a
print layout like this (see how the corner boxes are still in the corners,
whereas the ones along the top, right, bottom and left edges are shifted away
from the edge, due to usage of `env(safe-printable-inset)`:

![image](https://github.com/user-attachments/assets/1f25679a-58d6-4cb1-b371-3c4958073e73)

On paper it would look like this (if the printer doesn't lie too much about its
capabilities):

![image](https://github.com/user-attachments/assets/667b10fa-521f-4dbb-ac9c-4f424a197e05)

One concern is printers with large unprintable areas along one edge, and more
"reasonable" values along the other three edges. It may be necessary to
non-normatively tell authors about this. Maybe they want to use some sort of
`min(env(safe-printable-inset), 42px)`, to prevent too large margins. At the
same time, with printers that actually reach such a limit, content may end up
missing or clipped.

## What do web developers currently have to do, without this?

Since there's currently no API for this, authors have to either leave the
margins at the default settings (which should take unprintable areas into
account), or be very conservative about placing content near the paper edges.

## Alternatives

An alternative could be to provide an option in the (print preview) user
interface to fit the output to the printable area, by scaling it down. That
would make real-world measurements off, though.

## Are there any privacy, security and accessibility considerations?

This environment variable is only available (non-zero) when generating print
layout. Exposing it for screen layout is meaningless. It's still possible for a
site to obtain the safe printable inset. Is this a concern?

Example:

```html
<!DOCTYPE html>
<style>
  @page {
    margin: 0;
  }
  body {
    margin: 0;
  }
  #foo {
    margin: env(safe-printable-inset);
  }
</style>
Hello.
<div id="foo">The margins will be inset to steer clear of unprintable areas.</div>

<script>
  window.matchMedia("print").addListener(()=> {
    console.log("Margin-left: " + getComputedStyle(foo).marginLeft);
  });
</script>
```
