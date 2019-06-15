# Tutorial

Welcome to the Docli Tutorial! This tutorial is meant to introduce the Docli concepts. If you get stuck at any point during the tutorial, feel free to download https://github.com/ember-learn/super-rentals for a working example of the completed CLI.

## Confirming that Docli is installed

Before starting the tutorial, let's make sure that you have the latest version [installed]({{< relref "/docs/installation" >}}). Go ahead and create a `test.go` file and paste the following content inside:

{{<highlight go>}}
package main

import (
	"github.com/alecthomas/repr"
	"github.com/celicoo/docli"
)

func main() {
  args := docli.Args()
  repr.Println(args)
}
{{</highlight>}}

Let’s build and run it:

{{<highlight text>}}
$ go build
$ ./test

docli.args{
}
{{</highlight>}}

The output is the [Abstract Syntax Structure](https://en.wikipedia.org/wiki/Abstract_syntax_tree) of the command-line arguments. It’s empty because we didn't pass any arguments. Go ahead and check what happens when you call it passing different arguments.

## Creating a new command-line interface

### Directory structure

If you've ever used [Cobra](http://cobra.dev), you'll feel at home about this. While you are welcome to provide your own organization, typically a Docli-based application will follow the following directory structure:

{{< highlight text >}}
▾ app/
  ▾ cmd/
    search.go
    root.go
  main.go
{{< /highlight >}}

Let's take a closer look at the directory and files.

**app**: This is the root of your program. The `main.go` file goes in this directory, and it serves one purpose: initializing your CLI.

**cmd**: This is where your commands are stored. Every file in this directory holds the logic of a single command. The majority of your coding on an Docli-based application happens in this directory.

Go ahead and create the above structure. If you prefer you can clone the repository:

{{<highlight bash>}}
$ git clone git@github.com:celicoo/lolcat.git && cd lolcat/
{{</highlight>}}

### Writing the doc string

Like YAML or Python, the docstring is a line-oriented language that uses indentation to define structure. Lines beginning with either spaces or tabs are used to register argument identifiers. Here is an example of how to correctly define one argument with two identifiers:

{{<highlight text>}}
Options:
  -v, —-verbose     verbose output
{{</highlight>}}

The identifiers can have letters of any language, numbers and dashes. Unlike other libraries, Docli doesn't enforce the `short-long` convention.

Now that you've learned how to define argument identifiers, let's go back to our lolcat CLI and paste the following code to `cmd/root.go`:

{{<highlight go>}}
package cmd

import (
	"fmt"
	"log"

	"github.com/celicoo/docli"
)

type Git struct {
	Clone   Clone
	Version Version
}

func (g *Git) Doc() string {
	return `usage: git <command>
commands:
  clone    clone a repository into a new directory
  version  print version
	
Use "git <command> help" for more information about the <command>.`
}

func (g *Git) Run() {
	repr.Println(g)
}

func (g *Git) Help() {
	fmt.Println(g.Doc())
}

func (g *Git) Error(err error) {
	switch err.(type) {
	case *docli.InvalidArgumentError:
		// Ignore InvalidArgumentError.
		g.Run()
	default:
		log.Fatal(err)
	}
}

func Execute() {
	var g Git
	docli.Args.Bind(&g)
}
{{</highlight>}}

By looking at our command-line interface, can you guess how many **arguments** does it have? Try to make a guess before moving forward.

If you thought **two** you got it! Let's build and rerun it to confirm:

{{< highlight text >}}
$ go build
$ ./git

args.Args{
  text: text.Text{
    Arguments: []text.Argument{
      text.Argument{
        Indentation: "\n     ",
        Identifiers: []text.Identifier{
          text.Identifier{
            Name: "init",
          },
        },
      },
      text.Argument{
        Indentation: "\n  ",
        Identifiers: []text.Identifier{
          text.Identifier{
            Name: "h",
            Separator: ", ",
          },
          text.Identifier{
            Name: "help",
            Separator: ", ",
          },
          text.Identifier{
            Name: "助けて",
          },
        },
      },
    },
  },
}
{{</highlight>}}

The **h**, **help**, and **助けて** are identifiers of the same argument. If you are confused, please go back to the [Write a command-line interface](#write-a-command-line-interface) section.

### Binding struct fields to command-line arguments

Now that we’ve written our command-line interface and verified that Docli correctly parses it, it’s time to write a struct to be bind to the command-line arguments values. To do so, paste the following struct after the doc string:

{{<highlight go>}}
type Git struct {
	Init,
	Help bool
}
{{</highlight>}}

**Note:** you can use **any** primitive type in your struct fields and use embedded structs to separate the categories of your arguments, you'll find examples of the latest in the [examples page](https://github.com/celicoo/docli/tree/master/examples).

Replace the call to `repr.Println(args)` with:

{{<highlight go>}}
var git Git
args.Bind(&git)
if git.Help {
    fmt.Println(doc)
}
{{</highlight>}}

The final code should be similar to the following:

{{<highlight go>}}
package main

import (
	"log"

	"github.com/celicoo/docli"
)

var doc = `
Simple git

Usage: git [command]

Commands:
     init         create an empty Git repository or reinitialize an existing one
  h, help, 助けて  help message
`

type Git struct {
	Init,
	Help bool
}

func main() {
	args, err := docli.Parse(doc)
	if err != nil {
		log.Fatal(err)
	}
    var git Git
    args.Bind(&git)
    if git.Help {
        fmt.Println(doc)
    }
}
{{</highlight>}}

If we now build and run it:

{{< highlight text >}}
$ go build
$ ./git 助けて

Simple git

Usage: git [command]

Commands:
     init         create an empty Git repository or reinitialize an existing one
  h, help, 助けて  help message
{{</highlight>}}

Boolean arguments are special because they don't require explicitly assignment like arguments of other types.

For example, if we decide to add an argument `--author` of type `string`, the correct way to bind a value to it would be:

{{<highlight text>}}
$ ./git init --author=Docli
{{</highlight>}}

If you have any question, please don't hesitate to [open an issue](https://github.com/celicoo/docli/issues).

<!-- ## Concepts

Docli resolves around Commands and Arguments, you will see that those two abstractions are all you need to create more powerful abstractions.

### Commands

Commands represent actions, and they should be named as verbs. 

### Arguments

Lorem Ipsum..... -->

<!-- ## Help Command

Cobra automatically adds a help command to your application when you have subcommands. This will be called when a user runs ‘app help’. Additionally, help will also support all other commands as input. Say, for instance, you have a command called ‘create’ without any additional configuration; Cobra will work when ‘app help create’ is called. Every command will automatically have the ‘–help’ flag added. -->
