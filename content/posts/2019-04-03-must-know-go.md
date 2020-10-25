---
title: "Golang Must Know"
description: "What is Go, and how can you make your experience better?"
keywords: [tutorial, golang, go, programming, tips & tricks, experience, pprof, tools, docs]
date: 2019-04-03
---

![Golang Mascot, the Gopher](/assets/images/gopher.png)


## Introduction
---

Go or Golang is an OpenSource programming language developed by Google.

What makes Go special?

* Fast compiler with many system performance tools
* Easy concurrency for multicore or network programming
* Fast garbage collector and runtime reflection
* Statically typed and machine compiled
* Modules (to help rid ourselves of the `$GOPATH`)

With all of its good parts it also has some flaws, just like any programming language:

* No generics
* Clunky error handling
- Lack of macros
- Poor vendoring and handling of modules

If you wish to [download](https://golang.org/dl/) or read more about [Golang](https://golang.org) there are plenty of [official](https://golang.org/doc/) and community docs.

## Improving your Go experience
---
While Golang comes with plenty of helper tools, there are other third-party services, community tools or lesser known practices which will definitely save you some time or overall improve your workflow.

### Runtime package

Maybe not the most useful thing for everyone, but definitely a standard package that you should be aware of. Understanding what it does helps you better grasp concepts like **goroutines** and some of the “behind the scenes” services of Go.

[Runtime Package Documentation](https://golang.org/pkg/runtime/)

### GoDoc

When dealing with Golang tools, the best way to figure out its usage is via `go help <TOOL NAME>`. For example `go help doc` will tell you all about `go doc`.

Calling `go doc PACKAGE_NAME` will present you with the general package documentation and a list of all the public functions and structs. Calling `go doc PACKAGE_NAME/SUB_PACKAGE` will list the same but for a sub-package. You can filter this even more by requesting the doc message for a specific function or variable (`go doc PACKAGE_NAME.PublicFunction`).

The way GoDoc works is by checking the comments before Public functions, variables or structs and the top comment before your `package` definition. For example:

```rust
/*
Package foo implements all the needed foo and bar.
*/
package foo

// DefaultName is the string "Jon Doe", used if name is empty string.
const DefaultName = "Jon Doe"

// Foo performs bar with special configs.
func Foo(name string) {
	if name == "" {
		name = DefaultName
	}
	// do stuff ...
}
```

Note that package comments must start with `Package <NAME> something something ...` and other comments must start with the name of the function, struct or variable: `// Foo does something ...`. I’ll admin, this might get annoying at times, but if you’re only experimenting with a function you can make it private or set it’s comment to `// Foo ...`, this will stop any linter from complaining.

The best part about having an official and widely accepted standard for writing documentation, is the docs visualisation and readability. For example you can use the tool `godoc` (not `go doc`) for starting a local web instance with all the documentation for all packages.

[godoc Documentation](https://godoc.org/golang.org/x/tools/cmd/godoc)

```shell
godoc --http localhost:8080
```

### pprof

Pprof is a large topic to cover, to put it extremely simply it’s the best performance analysis and debugging tool I’ve ever seen. Not only it offers you loads of insight into possible bottlenecks and flaws you didn’t even know your software had, but it’s also easy to use.

Just throw this somewhere in your `main` or `init`. It will create a `cpu.profile` file where all the debug data is.

```rust
	// CPU profiler
	f, err := os.Create("cpu.profile")
	if err != nil {
		log.Fatal(err)
	}
	pprof.StartCPUProfile(f)
	defer pprof.StopCPUProfile()
```

Another way of obtaining `pprof` data is when running your tests with special flags:

```shell
go test -cpuprofile cpu.profile
```

As always Go has a tool for visualising this, and I really mean visualise! `go tool pprof` can be used as a cli tool (and it’s not bad to use) or as a pretty modern web interface.

```shell
go tool pprof -http="localhost:8080" cpu.profile
```

In the interface you’ll be able to see memory usage, CPU usage, heap allocations, time/line (and you can go to the ASM level).

For a more in depth guide to using pprof, I recommend this video (Prashant Varansai - Uber, Gopherfest Sprint 2016)

<div class="video-container"><iframe src="https://www.youtube.com/embed/N3PWzBeLX2M" frameborder="0" allowfullscreen></iframe></div>





### Code coverage

Golang comes with builtin testing tools and benchmarking which are quite good and easy to use. A lesser known usage of tests is the code coverage. You can run your tests as you’d normally do `go test ./...` but by adding an extra flag `go test -coverprofile cover.out ./...`, this will generate a file containing the coverage data and might look something like this:

```text
mode: set
cpl.li/go/cryptor/crypt/hkdf/hkdf.go:25.64,30.27 2 1
cpl.li/go/cryptor/crypt/hkdf/hkdf.go:35.2,35.18 1 1
cpl.li/go/cryptor/crypt/hkdf/hkdf.go:30.27,32.3 1 1
...
```

Not exactly human readable. That’s why there is a tool! (there always is). Using `go tool cover -html=cover.out`, will open a webpage in your default browser, showcasing your package coverage as overall percentages and line by line.

![Code coverage Go](/assets/images/codecov-go.png)

This is very helpful as you may forget writing tests for an edge case.

Other visualisation tools can be integrated in your deployment pipeline and connected to your CI. For example [codecov.io](https://codecov.io) allows you to connect successful builds from Travis-CI and then further deploy only if code coverage is above a threshold.

![Codecov circle](/assets/images/codecov-io-circle.png)

### io.Reader, io.Writer

You probably already encountered these two **interfaces**. They are some of the most common and useful interfaces in Go. With these two you can simply and have your code accessible to others and vice-versa.

The nice thing about `Reader` and `Writer` is the level of abstraction that they provide. For example all you need to know about these two are the following definitions:

```rust
type Reader interface {
	Read(p []byte) (n int, err error)
}
```
```rust
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

Or you can have a combination of both:

```rust
type ReadWriter interface {
	Reader
	Writer
}
```

At first this might not look like much, but having `Reader` and `Writer` as the main way of passing a struct around can change everything in your design philosophy. Readers and Writers can be chained together allowing for a nice and easy to read pipeline. For example you could have a `Reader` for reading a text file into a `Writer` which compresses the contents of the file and then further passes down its output to another `Writer` that writes back to a different file.

Writers and readers can be easily scaled and allow for modular and idiomatic code. Another good use of a `Reader` that I personally encountered is for any struct that generates output.

```rust
// create generator
generator := example.NewGenerator(seed string)

// prepare variables
var output0 [20]byte
var output1 [50]byte
output2 := make([]byte, 100)

// fill in variables
generator.Read(output0[:])
generator.Read(output1[:])
generator.Read(output2)
```

This might replace some design choices where you would end up having either:

```rust
func (g Generator) Generate(size int) []byte { ... }
func (g Generator) Generate(out *[]byte) { ... }
```

With the `Reader` or `Writer` interface you can change what’s happening behind the scenes without having to introduce breaking changes. In the end it’s all a matter of preference but knowing where to use these two interfaces will help you write better Go code.

[I/O Package Documentation](https://golang.org/pkg/io/)

## Things I wish I knew sooner
---

I first jumped on the Golang wagon around version 1.6 (Feb 2016), my first experience with it was actually around 1.5 (Aug 2015) but learning the syntax at the time was not something I was up for. After wrapping my head around **for** as the only loop statement, **goroutines** and **channels**, forced tab indentation and the other aspects of **go vet**, and last but not least the lack of a direction for the language, Golang proved to be a nifty language. It enabled me to develop more robust code in less time, with support from the builtin **testing** and **benchmarking** tools, easy to read syntax and formatting, plus the “package manager” was new an interesting for me.

All in all, Go provided plenty of documentation and [good ways for newcomers](https://tour.golang.org/welcome/1) to accommodate. Despite the fact, there are plenty of thing which would have helped me build cleaner and more organised projects.

### Remote import path

Normally go expects its **import** or **get** path to be from one of the known version control providers (e.g. `import "github.com/cpl/cryptor"`).

For the same example you can setup a custom import path (`import "cpl.li/go/cryptor`). You can even enforce your custom path to be used as the **only** path, by placing a comment at the package definition:

```rust
/*
Package cryptor ...
*/
package cryptor // import "cpl.li/go/cryptor"
```


#### How to setup

You must have a web-server running on your wanted domain and have control over what it serves. For the example above, when a `go get cpl.li/go/cryptor` happens, a request for `https://cpl.li/go/cryptor?go-get=1` is sent and the expected response **must** contain the `<meta>` tag below:

```html
<meta name="go-import" content="import-prefix vcs repo-root">
<!-- WHERE THE FOLLOWING FIELDS MUST BE REPLACED -->
<meta name="go-import" content="cpl.li/go/cryptor git https://github.com/cpl/cryptor">
```

A lesser known aspect of remote import paths is the `vcs` field, which can replaced with the keyword `mod` and it’s the preferred description when providing a module.

### Modules

As much as I’d like to talk about them, I won’t. Modules are still a new thing and feel like they have plenty of issues and quirky design choices that makes using them not yet worth it. The intention behind it is nice but the implementation … we better talk about them some other time.

### Build tags & options

While Go may not have a nice MACRO system such as the one in C, it still have some build tags which I found useful on multiple occasions.

Imagine you have a version of your software that is for debugging only, for example you can have a `main.go` which you just simply compile by saying `go build` or `go build -o exec_name`. But did you know you can have another file, for example `main_debug.go` inside which you can have another `func main()`? The only thing you’ll need is a special comment at the top of the file `// +build debug`. This will tell `go build` to only compile `main_debug.go` if and only if the `debug` tag is provided: `go build -tags debug -o exec_name_debug`. 

The file name can be anything you wish `main_foo.go`, the only thing to remember is the `// +build TAG_NAME` must match `go build -tags TAG_NAME`.

This is pretty awesome as you can have a debug version of your executables which logs what’s happening and also record pprof stats. While your normal distribution is faster and smaller by not doing these. 

## Conclusion
---

Golang is a growing language. It has some rough edges but nothing that won’t be fixed in upcoming version. Go 2.0 possibly promises generics. Modules will hopefully get their stuff together soon. I am happy to have learnt Go when I did because now I appreciate all the robust software that I’ve written in it. I can rest assured that they won’t introduce computability breaking changes.

Bear in mind that programming languages are mere tools, and just like any tool, if it get’s the job done just as good as any other tool then there is no best or better tool. Some are more skilled with a certain tool and less with others. Use whatever helps you solve the problem in an elegant and efficient manner.

> Everything can be a hammer & nail if you’re ambitious enough, doesn’t mean they should be.

## References
---
- [Command go - The Go Programming Language](https://golang.org/cmd/go/)
- [Packages - The Go Programming Language](https://golang.org/pkg/)







