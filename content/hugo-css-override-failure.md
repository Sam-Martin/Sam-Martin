---
title: "Hugo CSS Override Failure: Why is everything the same but also different"
date: 2020-02-04T20:26:21Z
draft: false
image: "/images/2020/02/2020-02-04-Unstyled-Hugo.png"
---

I recently swapped my blog from [Ghost](https://ghost.org/) to [Hugo](https://gohugo.io/).  
While I liked the former (being MarkDown based, lightweight and easy to use), I adore Hugo as you can host it for free with GitHub Pages.

## Automation

Whenever I setup a Hugo site I automate the deployment of it with [Travis CI](https://travis-ci.com/). This means that I can deploy updates to my blog just by editing and creating MarkDown files in GitHub, and as soon as they're committed to master the updated, rendered, website will be available in a matter of minutes.

## Styling

I customised the [m10c](https://github.com/vaga/hugo-theme-m10c) theme using an SCSS override. (Which is another thing I love about hugo, how easy it is to style.)

This worked by using the [Hugo template lookup order](https://gohugo.io/templates/lookup-order/) to override the empty default file `assets/css/_extra.scss` the theme provides by creating my own file in the same path in my Hugo project. Because project files supersede theme files in Hugo's lookup order, my file takes presedence. 

It did exactly what I wanted, which was to make it look as much like my old Ghost theme as possible.
{{<figure src="/images/2020/02/2020-02-04-Styled-Hugo.png" >}}

## The Problem

However, when I set up the Travis pipeline, instead of giving me the customised style above, it gave me a version of my site in the default styling.
{{<figure src="/images/2020/02/2020-02-04-Unstyled-Hugo.png">}}

Weird...  
I went over everything. I checked.

1. The commands Travis was running to generate the files vs what I was running on my laptop
2. The version of Hugo Travis was running
3. The version of template Travis was downloading
4. Whether I'd accidentally `.gitignored` the wrong file
5. Resetting the theme submodule
6. Checking out the theme separately not as a submodule
7. That I'd ignored the `resources` folder in the Hugo project to avoid polluting it
8. That I cleared the build folder in Travis before doing anything
9. The SHAsums of all the files on my laptop vs when checked out on Travis

_Everything_ was identical. I couldn't figure out what the issue was, so I uploaded the copy generated on my laptop and left it for a few weeks.

When I came back, I swapped over to using a Docker image both locally and in Travis and it _STILL_ rendered correctly on my laptop and incorrectly in Travis.

## The Solution

Eventually I discovered that in addition to the `resources` folder in the Hugo project (which holds a cache of rendered data from the last Hugo run, including CSS) that one got generated in the *THEME*.

So I cleared that, and re-ran the dockerised Hugo on my laptop.

```
ERROR 2020/02/04 20:00:10 Transformation failed: TOCSS: failed to transform "css/main.scss" (text/x-scss): this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
```

Turns out that there's something called Hugo Extended which has the capability to render SCSS (a dynamic CSS templating framework I'd never used before because I'm a decade out of date with web design).  
If you have Hugo Extended, your SCSS files will render and be cached in the `resources` folders of your project and theme every time you build your site in Hugo.  
If you don't have Hugo extended it will throw an error.

Okay, so that explains why it worked on my laptop (which had Hugo Extended) because my laptop rerendered the SCSS into CSS every time.  
But why didn't it throw that error on Jenkins/Docker when it couldn't render the SCSS into CSS?

Because as it turns out, the theme creator had (presumably accidentally) checked in the `resources` folder into Git. That meant that even though the version of Hugo running in Travis couldn't render the CSS (and thereby take into account my styling), it didn't throw an error because the default CSS was cached in the `resources` folder.

All I had to do was swap Travis to run the `-ext` Hugo docker image, and *voila* the site was styled correctly!

## Travis Build

The final `travis.yml` looked like this:

```
services:
  - docker
before_install:
  - docker pull klakegg/hugo:0.64.0-ext
script:
  - git clone https://github.com/Sam-Martin/hugo-theme-m10c.git themes/m10c
  - |
    docker run --rm -it \
    -v $(pwd):/src \
    -v $(pwd)/output:/target \
    klakegg/hugo:0.64.0-ext
deploy:
  provider: pages
  local_dir: output
  github_token: "$GITHUB_TOKEN"
  verbose: true
  skip_cleanup: true
  repo: Sam-Martin/sammartin.github.io
  fqdn: sammart.in
  target_branch: master
```

Super simple, and all it needed was `klakegg/hugo:0.64.0-ext` instead of `klakegg/hugo:0.64.0`.

Remember kids, error messages are a gift. When you get one, it is 1000x easier to figure out what's going on than if it errors without a message, or (as was the case here) it proceeds with insane behaviour.