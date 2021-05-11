---
title: "Writing a simple external data oriented Sphinx extension"
date: 2021-05-10T15:24:25+01:00
image: /images/2021/05/parse_rst_carbon.png
summary: "Sphinx is an extremely powerful and elegant way to generate documentation from code.
In this guide we're going to explore how to write a simple Sphinx extension that pulls in data from
outside our normal RST files and displays it in any way RST allows us."
draft: false
---

# What is Sphinx?

I love Sphinx. I was a little hesitant about RST at first coming from a MARKDOWN OR DEATH background I was
nervous about it being Yet Another Markup Language to learn like wikicode and its ilk, but I was quickly won
over by the power of directives.

But let's backup. Sphinx allows you to build a website like those you've seen under the [readthedocs.io](readthedocs.io) domain like [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html).

I built [CloudWanderer's](https://www.cloudwanderer.io) documentation with Sphinx, and that automatically generates page after page of documentation by pulling in data from CloudWanderer and Boto3 and merging them together in a way that makes sense.

Here's how I did it.

# Creating a new Sphinx project

Grab yourself an empty folder and let's install and initialise a new Sphinx project.

{{< highlight bash >}}
$ pip install sphinx sphinx_rtd_theme
$ sphinx-quickstart

Welcome to the Sphinx 4.0.0 quickstart utility.
Selected root path: .

> Separate source and build directories (y/n) [n]:
> Project name: Test
> Author name(s): Sam
> Project release []:
> Project language [en]:

Creating file ...

Finished: An initial directory structure has been created.
{{< /highlight >}}

Great! Now open up the `conf.py` that's been created and change
{{< highlight python>}}
extensions = []
{{< /highlight >}}
to
{{< highlight python>}}
extensions = ["sphinx_rtd_theme"]
{{< /highlight >}}
and
{{< highlight python>}}
html_theme = "alabaster"
{{< /highlight >}}
to
{{< highlight python>}}
html_theme = "sphinx_rtd_theme"
{{< /highlight >}}

This will replace the slightly old school default theme with the read the docs theme and make us feel a bit more like we're in 2021!

## Build HTML

Now you can build and open the html

### MacOS

{{< highlight bash>}}
$ make html
$ open _build/html/index.html
{{< /highlight >}}

### PowerShell on Windows

{{< highlight powershell>}}
> ./make.bat html
> ii _build/html/index.html
{{< /highlight >}}

### Voila! A newly built Sphinx documentation page!

![Newly built Sphinx site](/images/2021/05/test-html.png)

### Writing plain old RST

Open up `index.rst` and remove everything under:

{{< highlight rst>}}
Welcome to Test's documentation!
================================
{{< /highlight >}}

---

**Tip:** you don't want to remove that, as that's the RST equivalent of a H1 header.)

---

After that, type in any old thing you like, I find the best source of information on RST Sphinx markup is [Sublime and Sphinx Guide](https://sublime-and-sphinx-guide.readthedocs.io/en/latest/index.html), just ignore the fact that it's geared towards Sublime!

Once you've saved your `index.rst` with your new text, run `make html` or `./make.bat html` and refresh your page.

![Newly built Sphinx site](/images/2021/05/any-old-thing.png)

# Writing an extension.

Sphinx has a bunch of built in extensions, the most often used ones being things like [autodoc](https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html) and [doctest](https://www.sphinx-doc.org/en/master/usage/extensions/doctest.html) which respectively automatically generate documentation from your code's docstrings and allow you test the example code snippets you write into your documentation. Both of these are MIND blowingly awesome and you should check them out.

The tutorial I'm outlining here is based heavily on [the tutorial from the official Sphinx documentation](https://www.sphinx-doc.org/en/master/development/tutorials/todo.html), streamlined a bit for my learning style, with a few extra bits sprinkled in to answer some of the things I found difficult to track down and understand when I went through it the first time.

## Use Case

Sphinx's extensions are *hugely* powerful, and we're going to use it to bring data in from JSON files and make them part of our documentation in a useful fashion.

## Create a base extension

Start off by creating a folder called `_ext` with a file `my_first_sphinx_extension.py` inside it.
Your folder structure should now look a little like this.

{{< highlight bash>}}
$ tree -L 2
.
├── Makefile
├── _build
│   ├── doctrees
│   └── html
├── _ext
│   └── my_first_sphinx_extension.py
├── _static
├── _templates
├── conf.py
├── index.rst
└── make.bat
{{< /highlight >}}

Open up your `my_first_sphinx_extension.py` file and enter in the following:

{{< highlight python>}}
from docutils import nodes
from sphinx.util.docutils import SphinxDirective


class ShowJsonDirective(SphinxDirective):
    has_content = True

    def run(self) -> list:
        return [nodes.paragraph("", "Hello World.")]


def setup(app: object) -> dict:
    app.add_directive("show-json", ShowJsonDirective)
{{< /highlight >}}


Now update `conf.py` to change

{{< highlight python>}}
extensions = ["sphinx_rtd_theme"]
{{< /highlight >}}

to include your new extension

{{< highlight python>}}
extensions = ["sphinx_rtd_theme", "my_first_sphinx_extension"]
{{< /highlight >}}

and also add

{{< highlight python>}}
import sys
import os

sys.path.append(os.path.abspath("./_ext"))
{{< /highlight >}}

anywhere in your `conf.py` so that it can find your new extension

Then add a reference to your new directive into `index.rst`

e.g.

{{< highlight rst>}}
Welcome to Test's documentation!
================================


.. show-json ::

{{< /highlight >}}

And there we go!
![Hello World](/images/2021/05/hello-world.png)
Well... that's a bit underwhelming, let's get started with formatting our output at least!


## Formatting

This is the bit I could not for the life of me figure out how to do when I read the official tutorial.
You'll notice that we added our `Hello World` in a `docutils.nodes.paragraph` object.
So where do we find other semantic objects for us to place into our content?

So far as I can tell the official canonical reference is here: [Docutils docs](https://docutils.sourceforge.io/docs/ref/doctree.html) but as it's ironically littered with `to be completed` I found it incredibly hard to find and understand the formatting elements I needed.

So let's just use RST! Screw creating the objects themselves, let's just build out some RST markup and we'll let Sphinx handle the rest!

To do that we need to add the following `parse_rst` method to our `ShowJsonDirective` class

{{< highlight python>}}
    def parse_rst(self, text):
        parser = RSTParser()
        parser.set_application(self.env.app)
        settings = OptionParser(
            defaults=self.env.settings,
            components=(RSTParser,),
            read_config_files=True,
        ).get_default_values()
        document = new_document("<rst-doc>", settings=settings)
        parser.parse(text, document)
        return document.children
{{< /highlight >}}

This method expects us to supply 1 argument which is a multi-line string of RST markup and will return the
corresponding docutils objects so we can return them from our extension's `run` method.

To use this, we can change our content to look like this:

{{< highlight python>}}
    def run(self) -> list:
        return self.parse_rst("\n* hello\n* world")
{{< /highlight >}}

This uses the `*` RST markup to render an unordered list with two elements (hello and world).

Our whole extension file now looks like (note the new `new_document` import):

{{< highlight python>}}
from sphinx.parsers import RSTParser
from docutils.frontend import OptionParser
from sphinx.util.docutils import SphinxDirective
from docutils.utils import new_document


class ShowJsonDirective(SphinxDirective):
    has_content = True

    def run(self) -> list:
        return self.parse_rst("\n* hello\n* world")

    def parse_rst(self, text):
        parser = RSTParser()
        parser.set_application(self.env.app)

        settings = OptionParser(
            defaults=self.env.settings,
            components=(RSTParser,),
            read_config_files=True,
        ).get_default_values()
        document = new_document("<rst-doc>", settings=settings)
        parser.parse(text, document)
        return document.children


def setup(app: object) -> dict:
    app.add_directive("show-json", ShowJsonDirective)
{{< /highlight >}}

And if we run `make html` and refresh the page again we get...

![A successfully rendered unordered list](/images/2021/05/unordered-list.png)

---

**Tip:** If you run `make html` after editing your extension and nothing seems to have changed, it may be because Sphinx hasn't detected any change in the doc source (because, well, there hasn't been one!). You can force Sphinx to generate it again by removing the `_build` directory with `rm -rf _build && make html`

---

## Hooking up the JSON

Now the easy part! Let's make this dynamic!

Write a file called `airports.json` inside the base directory (alongside `conf.py`) and place the following inside it.
{{< highlight json>}}
[
    {"Name": "Heathrow", "Url": "https://www.heathrow.com/"},
    {"Name": "Gatwick", "Url": "https://www.gatwickairport.com/"},
    {"Name": "Luton", "Url": "https://www.london-luton.co.uk/"}
]
{{< /highlight >}}

Now update your extension's file to the following:

{{< highlight python>}}
import json

from sphinx.parsers import RSTParser
from docutils.frontend import OptionParser
from sphinx.util.docutils import SphinxDirective
from docutils.utils import new_document


class ShowJsonDirective(SphinxDirective):
    has_content = True

    def run(self) -> list:
        rst = ""
        with open("airports.json", "r") as f:
            airports = json.load(f)
        for airport in airports:
            rst += f"* `{airport['Name']} <{airport['Url']}>`_\n"
        print(rst)
        return self.parse_rst(rst)

    def parse_rst(self, text):
        parser = RSTParser()
        parser.set_application(self.env.app)

        settings = OptionParser(
            defaults=self.env.settings,
            components=(RSTParser,),
            read_config_files=True,
        ).get_default_values()
        document = new_document("<rst-doc>", settings=settings)
        parser.parse(text, document)
        return document.children


def setup(app: object) -> dict:
    app.add_directive("show-json", ShowJsonDirective)
{{< / highlight >}}

This will take our JSON, and convert it into the following RST:

{{< highlight rst >}}
* `Heathrow <https://www.heathrow.com>`_
* `Gatwick <https://www.gatwickairport.com/>`_
* `Luton <https://www.london-luton.co.uk/>`_
{{< / highlight >}}

Which creates a list of links as per the standard RST link syntax!
The `print` will also make the above appear so that we can see the RST before it's rendered when we run `make html`.

Once you have run `make html` again you can see our beautiful auto-generated output!

![A rendered list of links](/images/2021/05/london-heathrow-gatwick.png)

# Conclusion & Further Reading

That's all there is to it! It's really easy to get started with Sphinx extensions.

You can find the full code [on GitHub](https://github.com/Sam-Martin/sphinx-read-from-json-extension-tutorial/tree/main).

* [Sphinx Extension Tutorial: Hello World](https://www.sphinx-doc.org/en/master/development/tutorials/helloworld.html)
* [Sphinx Extension Tutorial: Todo Extension](https://www.sphinx-doc.org/en/master/development/tutorials/todo.html)
* [Sphinx Extension Tutorial: Recipe Extension](https://www.sphinx-doc.org/en/master/development/tutorials/recipe.html)
