+++
author = "Sam Martin"
categories = ["Go", "Mondo", "YNAB"]
date = 2016-05-15T00:00:00Z
description = ""
draft = false
image = "/images/2016/05/2016-05-15---GetMondo-1.png"
slug = "mondo-ynab-and-an-intro-to-go"
tags = ["Go", "Mondo", "YNAB"]
title = "Mondo, YNAB, and an Intro to Go"
aliases = ['/mondo-ynab-and-an-intro-to-go/']
+++

# Go
I've been wanting to get into [Go](https://golang.org/) for a while. Despite the logo it's a very respectable open source project developed by a team at Google that *"makes it easy to build simple, reliable, and efficient software"*.
Crucially, Go is a cross-platform, compiled language that's used notably by the [HashiCorp](https://www.hashicorp.com/) suite of products. That means that if I want to contribute modules to say, [Terraform](https://terraform.io) (and I do) I need to know Go well enough to write decent, tested code that will get accepted in a pull request.

# Mondo
Recently I received my [Mondo Card](https://getmondo.co.uk/).
![](/images/2016/05/2016-05-15---GetMondo.png)
It's currently a pre-paid Mastercard (once they get their banking license they're supposed to release a full Current Account) that allows you to instantly see when your card's used with much more metadata about the transaction.  
At the moment the advantage over standard banking is mostly shininess and instant transparency into foreign transaction costs (the latter being the *biggest* feature by far to me). But the big disadvantage is that it currently *only* has a phone app, no web app of any description. What that means is that it has no transaction export functionality.

# YNAB & OFX

I've been using [YNAB (You Need A Budget)](http://www.youneedabudget.com/) for a couple of years now, and am a huge fan of its budgeting methodology and workflow. They've recently released a shiny new web version of the application which automatically imports transactions from banks, but there's a problem. It's pretty much only US banks, so I'm still stuck on the older thick client until they allow manual transaction imports. 

To that end there's a standard XML banking transaction format called [OFX (Open Financial Exchange)](http://www.ofx.net/) which YNAB recognises and most internet banking sites allow you to export your transactions into.

Although Mondo doesn't offer native OFX export from its phone apps, it **does** [have an API](https://getmondo.co.uk/docs/). So in order to get my Mondo transactions into YNAB, all I need to do is extract the data from the API and format it in OFX! Simple!

# Creating the OFX Exporter in Go
I decided to create the tool to export the transactions to OFX in Go purely because I wanted to get my hands dirty with Go. I tried to use [Stephen Whitworth's](http://www.stephenwhitworth.com/) [Go Mondo Library](https://github.com/sjwhitworth/go-mondo) but couldn't get it to work as (at the time) it was using a password grant type, which no longer seems to be supported by the Mondo API.

Because of that fact it would have made *way* more sense to make this as an AWS Lambda or Azure Functions app. Mondo's API is now built around the referral methodology of [oAuth 2.0](http://oauth.net/2/). This means that you can't authenticate against the API without having a referral page to ingest the access code once the user has granted the application access to their account.

My initial toe-in-the-water with Go was [Go's tour](https://tour.golang.org/welcome/1) which goes from hand-holding one second to asking you to complete a [Fibonacci Closure](https://tour.golang.org/moretypes/26) the next. It actually gives a really concise introduction to the language, but if you (like me) are extremely rusty with the parlance and concepts of compiled languages it's very dense and pretty offputting. I eventually found [GoLangByExample](http://golangbyexample) thanks to [@MachielMolenaar](https://twitter.com/MachielMolenaar), and this gave me a lot of good examples for achieving my goal.

Fortunately with Go's [net/http](https://golang.org/pkg/net/http/) module it is trivial to get your Go package to host a page for the referral. I'm dead serious, this is the code to get a web server running in Go: 
```
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

It's also extremely simple to write XML (OFX's data structure of choice) in Go, and the [examples in Go's docs](https://golang.org/src/encoding/xml/example_test.go) are just what was required.

```
type Address struct {
	City, State string
}
type Person struct {
	XMLName   xml.Name `xml:"person"`
	Id        int      `xml:"id,attr"`
	FirstName string   `xml:"name>first"`
	LastName  string   `xml:"name>last"`
	Age       int      `xml:"age"`
	Height    float32  `xml:"height,omitempty"`
	Married   bool
	Address
	Comment string `xml:",comment"`
}

v := &Person{Id: 13, FirstName: "John", LastName: "Doe", Age: 42}
v.Comment = " Need more details. "
v.Address = Address{"Hanga Roa", "Easter Island"}

output, err := xml.MarshalIndent(v, "  ", "    ")
if err != nil {
	fmt.Printf("error: %v\n", err)
}

os.Stdout.Write(output)
```
While some of that syntax initially looks pretty esoteric to someone (like me) coming from PowerShell/Python/JavaScript/a-little-C#, once you get your head around it it's extremely elegant.
The `struct` is (as I understand it) basically the Model part of [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) in C#, or a `Prototype` of an object in JS, meaning you instantiate a copy of that struct and populate its values appropriately. While I have to profess I haven't totally got my head around the implications and applications of pointers (the `&` and `*` denotations), it's easy enough to muddle through as you learn to get the hang of it.

## Testing, Linting & Continuous Integration
As you might expect of a sexy cool hipster language, Go is very well equipped in the niceties of TDD, Linting, and CI.
Tests (while not as natural language as rSpec, Pester, etc.) are pretty straightforward.
The examples in [Go's docs](https://golang.org/pkg/testing/) are a little thin, so I ended up plagiarising the [tests from the Mondo Go module](https://github.com/sjwhitworth/go-mondo/blob/master/mondo_test.go) which introduced me to the [Testify suite of packages](https://github.com/stretchr/testify) that provide a lot of useful functionality on top of what's bundled natively with Go.
```
import (
  "testing"
  "github.com/stretchr/testify/assert"
)

func TestSomething(t *testing.T) {

  var a string = "Hello"
  var b string = "Hello"

  assert.Equal(t, a, b, "The two words should be the same.")

}
```
And to run it, it's as simple as `go test`. Love it!

Linting is similarly straightforward, `gofmt -w .`, bam all your `.go` files are properly whitespaced, etc.

As ever [Travis CI](travis-ci.org) continues to impress and getting it to run the tests automatically was three lines of code in the `.travis.yml`
```
language: go

go:
  - tip

```


# export-mondo-transactions
![](/images/2016/05/2016-05-15---export-mondo-transactions.png)

With those building blocks in place, I was able to create a simple Go application which will run on any system, create a webserver locally on `8080`, and once it receives the authentication code on the referral URL, reaches out to the Mondo API, retrieves the latest transactions and saves them to an OFX file in the appropriate format!
![](https://cloud.githubusercontent.com/assets/803607/15273116/8c8c2844-1a87-11e6-8207-8d79c600e1fd.png)
## Usage
1. Create a [client from Mondo's website](https://developers.getmondo.co.uk/apps/home) (redirect url doesn't matter, use google.com if you like).
2. Identify the `client_id` and `client_secret` from the newly created client
3. Download the [latest release](https://github.com/Sam-Martin/export-mondo-transactions/releases/latest)
4. Run the downloaded executable and enter your `client_id` and `client_secret` when prompted
5. Follow the instructions in the browser that opens to `localhost:8080` (don't close the application until you've got your ofx!

If you're not on Windows you'll need to replace step #4 with:  

1. Install [Go](http://golang.org)
2. Populate the environment variable `GOPATH` with an absolute path to an empty directory
3. `go get github.com/sam-martin/export-mondo-transactions/export-mondo-transactions`
4. `go get github.com/sam-martin/export-mondo-transactions/export-mondo-transactions`
5. `%GOPATH%/bin/export-mondo-transactions`

