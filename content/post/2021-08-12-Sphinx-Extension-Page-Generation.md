---
title: "Sphinx Extension Page Generation"
date: 2021-08-12T19:03:12Z
image: /images/2021/08/write_pages_index.png
summary: |
    Following on from a previous blog post on writing Sphinx extensions, a natural requirement is the ability to automatically generate not just content on a single page with your extension, but also content on wholly new pages! Let's see how that's done.

draft: false
---

## Objective

Following on from [External Data Sphinx Extension]({{< ref "2021-05-10-External-Data-Sphinx-Extension.md" >}}), a natural requirement is the ability to automatically generate not just content on a single page with your extension, but also content on wholly new pages!

![Sphinx pages](/images/2021/08/write_pages_index.png)

## The story so far

You can find all the code for this example in the repo here: https://github.com/Sam-Martin/sphinx-write-pages-tutorial 

If you're confused about what Sphinx is, why you would want to write an extension, or how to get started, you should check out the start of the [External Data Sphinx Extension]({{< ref "2021-05-10-External-Data-Sphinx-Extension.md" >}}) post.

## The basis for this example

My basis for writing this example in this fashion is by following along the approximate process of the built in [autosummary extension](https://www.sphinx-doc.org/en/master/usage/extensions/autosummary.html) provided by Sphinx.

You can see the full code for this extension [on GitHub](https://github.com/sphinx-doc/sphinx/tree/4.x/sphinx/ext/autosummary). Any mistakes I make in this implementation are my own, and you should defer to the autosummary extension as the canonical way to do this if you want a more complete example.

## Writing the extension

We'll start with the usual Sphinx build and creation of an `_ext` subfolder.

{{< highlight shell>}}
$ sphinx-quickstart 
Welcome to the Sphinx 4.0.0 quickstart utility.
...

$ mkdir _ext
$ touch _ext/write_pages.py
{{< /highlight >}}

Our folder now looks like this:

![Folder contents](/images/2021/08/folder_contents.png)

As we open up our new `write_pages.py` we'll need to create four things.

1. `main` function
2. `setup` function
3. `ListPagesDirective` class
4. `WritePages` class

The `setup` function and `<something>Directive` class we've seen before, so what are the other two?

### The `WritePages` class

This is the class that is going to write our `.rst` files to disk so that Sphinx can discover them and render the rst within into html/pdf/whatever. 
This needs to be seperate from our directive because it needs to run on the [`builder-inited`](https://www.sphinx-doc.org/en/master/extdev/appapi.html#event-builder-inited) event which we will subscribe to with `app.connect()` and which does not provide the arguments required to instantiate our directive.  
We also use it as the source of truth for our list of files.  

#### Getting lists of files from directives
[autosummary does its own parsing of `.rst` files](https://github.com/sphinx-doc/sphinx/blob/1acdf4f8735d40b875161a94c1e0124aceb14217/sphinx/ext/autosummary/generate.py#L463) in order to pull in the list of files to create from its `.. autosummary::` directives. It does this, presumably, because it wants to create `.rst` files, which needs to be done before the `.rst` files are read by Sphinx, otherwise they won't get included in the final output.  
If it waited until Sphinx had found all instances of its directives in the other `.rst` files it would be too late to write its own `.rst` files to be read by Sphinx.

Creating custom directive parsing is well outside the scope of this tutorial, so we'll be using a simple list as a class property on `WritePages` as our source list of files to create.


### The `main` function

This function is a little shim that instantiates our `WritePages` class and calls the `write_pages` method on it.
In theory we could have the `main` function write the pages itself, but we need a source for the list of pages that's accessible both in the `main` function and in our `ListPagesDirective` class.


### The code

{{< highlight python >}}
iimport pathlib
from typing import List

from docutils.nodes import Node
from docutils.frontend import OptionParser
from docutils.utils import new_document
from sphinx.application import Sphinx
from sphinx.parsers import RSTParser
from docutils import parsers

from sphinx.util.docutils import SphinxDirective


class WritePages:
    files = [f"file_{i}" for i in range(1, 10)]
    relative_path = pathlib.Path(__file__).parent.absolute() / ".."

    def write_pages(self) -> None:
        for file_name in self.files:
            with open(self.relative_path / f"{file_name}.rst", "w") as f:
                f.write(f"Test - {file_name}\n==============")


class ListPagesDirective(SphinxDirective):
    has_content = True

    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
        self.pages = WritePages()

    def run(self) -> List[Node]:
        rst = ".. toctree::\n"
        rst += "   :maxdepth: 2\n\n"
        for file_name in self.pages.files:
            rst += f"   {file_name}\n"
        return self.parse_rst(rst)

    def parse_rst(self, text: str) -> List[Node]:
        ...


def main(app: Sphinx) -> None:
    w = WritePages()
    w.write_pages()


def setup(app: Sphinx) -> None:
    app.add_directive("list-pages", ListPagesDirective)
    app.connect("builder-inited", main)



{{< /highlight>}}

For brevity I've omitted the `parse_rst` method contents, again you can see the full implementation [in GitHub](https://github.com/Sam-Martin/sphinx-write-pages-tutorial/blob/main/_ext/write_pages.py).

Let's go through this one step at a time.

1. Our `main` will be called, which will
2. Call our `write_pages` method which will
3. Write the raw contents of our `.rst` files specified in `WritePages.files`
4. Then our `ListPagesDirective`'s `run` method will be called which will get the list of files from `WritePages.files` and turn it into a `toctree`, then convert that into docutils nodes with `parse_rst` and return it.

{{% note %}}
**Tip:** The `WritePages.write_pages` method wants the file names *with* `.rst` but when writing the `toctree` in `ListPagesDirective.run` we want them *without* the `.rst` extension. This always catches me out when writing one of these!
{{% /note %}}

### Adding the toctree

We can now use our `list-pages` directive to add a toctree!

{{< highlight rst >}}

Write Pages Tutorial
====================

.. list-pages ::

{{< /highlight >}}

And now we can run 

{{< highlight shell>}}
$ make html
{{< /highlight >}}

Now you'll notice that your folder now has 10 new files in it!

{{< highlight shell >}}
$ ls -lha
total 120
...
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_1.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_2.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_3.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_4.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_5.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_6.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_7.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_8.rst
-rw-r--r--   1 sammartin  staff    28B 13 Aug 17:53 file_9.rst
{{< /highlight >}}

And when you open your new `_build/html/index.html` file you will see your rendered `toctree` and the links to the newly created files!

![Sphinx pages](/images/2021/08/write_pages_index.png)

🎉 Congratulations! You've written a Sphinx extension that can organise your auto-generated documentation into multiple sub pages!

### On the deletion of files

If you play around with the list of files specified in `WritePages.files` you may notice that there's no mechanism to delete the files. This behaviour reflects that of autosummary.

If you have extremely volatile file lists and are likely to be frequently orphaning pages it may be a good idea to configure your extension to write your files to a dedicated subfolder that's deleted as part of your `make` command.
