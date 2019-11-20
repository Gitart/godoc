# How To Create Your Own CLI — With Golang

## An introduction to making CLI’s with Golang

[![Jeroenouw](https://miro.medium.com/fit/c/96/96/1*5bW5k2LxVehZcCSPvmm4xg.png)](https://itnext.io/@jeroenouw?source=post_page-----3c50727ac608----------------------)

[Jeroenouw](https://itnext.io/@jeroenouw?source=post_page-----3c50727ac608----------------------)

[Follow](https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fitnext.io%2Fhow-to-create-your-own-cli-with-golang-3c50727ac608&source=-dc695ec3c2c2-------------------------follow_byline-)

[Mar 4](https://itnext.io/how-to-create-your-own-cli-with-golang-3c50727ac608?source=post_page-----3c50727ac608----------------------) · 4 min read

![](https://miro.medium.com/max/60/0*a6IE1sqbgR7SHiou?q=20)

![](https://miro.medium.com/max/3780/0*a6IE1sqbgR7SHiou)

![](https://miro.medium.com/max/7560/0*a6IE1sqbgR7SHiou)

Photo by [Anastasia Yılmaz](https://unsplash.com/@anastasiayilmaz?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

In this article, I will explain how you can make a basic command line interface (CLI) with Golang. We are going to make a small pizza CLI, after following the article you will have an idea of how you set up one and create a custom CLI by yourself.

## Prerequisites

*   For this demo, I used: go version 1.11.

## Let’s start creating the code

Create a folder with a file named `main.go` , and run in the command line:

go get github.com/urfave/cli

Now we installed a well\-known [package](https://github.com/urfave/cli) globally to easily make a CLI with Golang. This will saves us a lot of time.

In your `main.go` , let’s instantiate a `NewApp` from the CLI package and make it able to run by calling the `Run` method. We also want to log an error if something went wrong:

var app = cli.NewApp()func main() {
  err := app.Run(os.Args)
  if err != nil {
    log.Fatal(err)
  }
}

This is enough to make your CLI work but without any options. We also want some basic info about the CLI. Above the `main` function, make another function called `info` with some details:

func info() {
  app.Name = "Simple pizza CLI"
  app.Usage = "An example CLI for ordering pizza's"
  app.Author = "Jeroenouw"
  app.Version = "1.0.0"
}

Next, we are going to define a new string array containing a basic sentence, which we will extend after running the CLI with some custom command. Please put the next above the `info` function:

var pizza = \[\]string{"Enjoy your pizza with some delicious"}

The last thing we need to complete our Golang CLI is by adding some of our custom commands, we make a new function called `commands` :

func commands() {
  app.Commands = \[\]cli.Command{
    {
      Name:    "peppers",
      Aliases: \[\]string{"p"},
      Usage:   "Add peppers to your pizza",
      Action: func(c \*cli.Context) {
        pe := "peppers"
        peppers := append(pizza, pe)
        m := strings.Join(peppers, " ")
        fmt.Println(m)
      },
    },
    {
      Name:    "pineapple",
      Aliases: \[\]string{"pa"},
      Usage:   "Add pineapple to your pizza",
      Action: func(c \*cli.Context) {
        pa := "pineapple"
        pineapple := append(pizza, pa)
        m := strings.Join(pineapple, " ")
        fmt.Println(m)
      },
    },
    {
      Name:    "cheese",
      Aliases: \[\]string{"c"},
      Usage:   "Add cheese to your pizza",
      Action: func(c \*cli.Context) {
        ch := "cheese"
        cheese := append(pizza, ch)
        m := strings.Join(cheese, " ")
        fmt.Println(m)
      },
    },
  }
}

In `commands` function we call the `Commands` method from the imported CLI package. Then we make an array of commands containing 3 objects; our custom commands.

We define a `Name` , `Aliases` , `Usage` and an `Action` . There are more options available in this package to explore, the one we are using is just the most common ones.

*   As `Name` you will set the command name, in this case, toppings for your pizza.
*   As `Aliases` we make a string array which can contain one or more aliases for our command.
*   For the `Usage` we shortly describe what this command does, so we can see this in the `help` overview.
*   In the `Action` we provide our code which will be executed. In this case, we append our topping to the pizza string array. Followed by making it a string.

## Local usage

To build our CLI, please run:

go build main.go

After this, you can run the following two existing commands:

./main \-\-help
./main \-\-version

Our output by running `./main — help` will now look like this:

NAME:
   Simple pizza CLI \- An example CLI for ordering pizza'sUSAGE:
   main \[global options\] command \[command options\] \[arguments...\]VERSION:
   1.0.0AUTHOR:
   JeroenouwCOMMANDS:
     peppers, p     Add peppers to your pizza
     pineapple, pa  Add pineapple to your pizza
     cheese, c      Add cheese to your pizza
     help, h        Shows a list of commands or help for one commandGLOBAL OPTIONS:
   \-\-help, \-h     show help
   \-\-version, \-v  print the version

Our custom options can be called with a full name:

./main peppers
./main cheese
./main pineapple

or with the shorten alias:

./main p
./main c
./main pa

The output of `./main pa` will be looking like this:

Enjoy your pizza with some delicious pineapple

This is just a very basic CLI, you can do way more than this. But now you know the basics.

## Global usage

1.  After running `go build main.go`the executable file ‘main’ is made

2\. Run `cp main /usr/**local**/bin`

3\. To test: `main peppers`

This can be run globally now (credits to [Melody Anoni](https://medium.com/u/dc4c5ebeb5ea?source=post_page-----3c50727ac608----------------------))

## Final code (main.go)

![](https://miro.medium.com/max/24/1*UGrTOOrIuFwvnWGW8PX3tA.png?q=20)

![](https://miro.medium.com/max/1308/1*UGrTOOrIuFwvnWGW8PX3tA.png)

![](https://miro.medium.com/max/2616/1*UGrTOOrIuFwvnWGW8PX3tA.png)
