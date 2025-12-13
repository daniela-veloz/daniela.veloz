---
title: 'Build Your Own Shell in Go'
description: 'Learn how to create a functional command-line shell from scratch using Go'
pubDate: 'Dec 11 2025'
heroImage: '../../assets/blog-placeholder-1.jpg'
---

A couple of weeks ago, I started learning Go. This is a language I've been wanting to learn for a while, and now 
I have a need for it! so I took a couple of tutorials, but because learning is better when you build something, 
I decided to learn by building my own shell.

Please note that this will be a very simple shell, but strong enough to:

1. help you get started or get better at Go
2. understand how your terminal actually works

In this post, I'll walk you through creating a basic but functional shell in Go.

## What is a Shell?

A shell is a program that takes commands from the keyboard and gives them to the operating system to perform. 
It's the interface between you and the operating system. Popular shells include bash, zsh, and fish.

## What We Will Build

We will build a shell following these steps:

1. building the simplest possible shell
2. add validation for non-existing commands
3. extend to support commands with arguments
4. add built-in commands for like for `cd`
5. add support for pipes
6. handle signals like `CTRL+C`
7. extend to support history

Don't worry if some of these terms sound unfamiliar,I try to cover everything step by step. Let's get started!

## Building the Simplest Possible Shell

We will build the simplest possible shell first and add on top of it. For a simple shell, it should be able to receive 
commands and use our OS to execute them.

#### Reading Input

We use a while loop (while loops don't exist in Go, so we use an infinite for loop) using [bufio](https://pkg.go.dev/bufio) 
to read from the standard input (os.Stdin).

#### Exit Support

Even if this is the simplest shell, we need to provide a way to terminate the program, so we're adding an 
'exit' custom command.

#### Executing Commands

To execute commands we will use the `os/exec` package. We need to send it the command, but we also need to wire up 
the standard streams (stdout and stderr).

```go
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
```

These two lines handle stream redirection, which is a fundamental concept in Unix-like systems. 
Every process has three standard streams:

- **stdin** (standard input) - where input comes from
- **stdout** (standard output) - where normal output goes
- **stderr** (standard error) - where error messages go

By assigning os.Stdout and os.Stderr to our command's output streams, we're telling Go: 
"Whatever this command prints, send it directly to the terminal." Without this wiring, the command's output 
would be lost in the void.

Here's our simplest shell so far:

```go
import (
	"bufio"
	"fmt"
	"os"
	"os/exec"
	"strings"
)

func main() {
	// read a line of input from the user standard input
	reader := bufio.NewReader(os.Stdin)

	for {
		// display the prompt
		fmt.Print("> ")
		// read the input
		input, _ := reader.ReadString('\n')
		input = strings.TrimSpace(input)

		if input == "exit" {
			os.Exit(0)
		}
		// run command
		cmd := exec.Command(input)
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		cmd.Run()
	}
}
```