# Shell Scripting - General Notes


## Shebang

Always start a script with "#!/bin/bash", or whatever environment is meant to
interpret your script. for example, a Python script would have "#!/usr/bin/python"
as its shebang.

```sh
#!/bin/bash

# Code here is interpreted as a bash script.
echo "Hello, this is a bash script."

exit
```

It's considered good practice to use the `env` command to read an alias rather than hardcoding in the name of the binary that will interpret your script. This allows for greater portability. 

```sh
#!/usr/bin/env python
```

Now we're using whatever the shell has `python` set to, which might be a link or alias to a different version or location than what we were using on the machine that we wrote the script on.


## Variables

Variables are assigned like this:

```sh
my_var=123
```

...and referred to like this:

```sh
$my_var     # shell autocompletes to fill in the value of the $my_var
```

The shell allows you to set a variable and immediately follow it with a command that will be run. This will set the new value of the variable only for the duration of the command. It looks like this:

```sh
MY_VAR="newvalue" read username
```

`$MY_VAR` will take the value of whatever it was previously after `read` has finished. This is a functionally similar and shorter version of this:

```sh
OLD_MY_VAR="$MY_VAR"
MY_VAR="newvalue"
read username
MY_VAR="$OLD_MY_VAR"
```

**It's important that there is no whitepace on either side of the '=' when assigning shell variables.**

Since the shell interprets and replaces any instance of $my_var as it reads a line, it's good practice to enclose our variables in double-quotes. This will prevent problems if the variable is unassigned. 

When substituting a varible into a string, it can be useful to enclose the name of the variable in curly-braces. This allows us to have non-whitespace immediately affter the variable.

```sh
x=abc

#interprets this as a variable named $x123, which doesn't exist.
echo "$x123"

# this protects $x and results in abc123, our desired output.
echo "${x}123"
```


## Command Substitution

Bash will interpret commands within `$(...)` as well as backticks. The `$(...)` syntax is more modern and generally preferred.

```sh
# the pwd command is run and the result replaces $(pwd) in the string.
echo "You are currently in $(pwd)
```


## HEREDOCS

Heredocs allow use to have a multi-line string literal. You create one with the `<<` operator followed by a delimeter string, and then terminate it with the same delimiter string on a newline without any leading whitespace:

*Note: markdown syntax highlighting is mangling these examples so I'm just using a generic code block.*

```
<< HEREDOC
This is my heredoc. The phrase that I use for the delimiter doesn't matter.
HEREDOC
```

Using the `<<-` operator instead of `<<` will trim all tabs (and only tabs, not spaces) from the input, allowing us to indent our heredoc to match with the indentation of the script.

```
<<- HEREDOC
        Any and all indentation here doesn't
matter, as it will
            all be stripped out at the end. It's also cool
        if I put my heredoc delimiter somewhere strange, like this:
                    HEREDOC
```

### Here Strings

A Here String is a single-string version of a Here doc. It is denoted with the `<<<` operator and uses either single- or double-quotes as a delimiter rather than a word.

Here strings can be used to feed a string into a command. They seem to be most commonly used in places where pipes wouldn't fit, which seems to be due to the fact that piped command are run in subshells.

Consider the following example:

```sh
echo "hello world" | read first second
echo "$second" "$first"
```

This will output the single space between the two quotes and nothing else. The reason behind this is that when we piped the two words into `$first` and `$second` via `read`, those variables were assigned in the context the subshell that `read` is running in. When `read` has finished execution, the subshell is terminated and these variables are lost. 

The proper way of doing this would be to use a Here String, like so:

```sh
read first second <<< "hello world"
echo $second $first
```

This will result in `"world hello"`, which was the intended output of the original command.

This seems like a kind of esoteric use case but I know that I'm going to forget about this and wonder what it is later.



## Functions

Functions are created one of two ways:

```sh
my_function () {
    ...
    return
}

# invoke our function

my_function

function other_function {
    ...
    return
}

# invoke our other function

other_function
```

Note that unlike other languages, you don't put parenthesis at the end of the function name when you call it. You can think of functions as their own discrete scripts - bash now interprets the name of the just as it interprets any other command or alias.

### Local Variables

Functions can have local variables with the `local` keyword. These will only be accessible within the context of the script, and can have the same name as global variables while being distinct entities.

```sh
foo=1
bar=2

echo $foo               # 1
echo $bar               # 2

test_func () {
    local foo=50
    # without using local, this will modify the global variable:
    bar=60
    echo $foo           # 50
    echo $bar           # 60

    return 
}

echo $foo               # 1
echo $bar               # 60
```

## Exiting Scripts

The `exit` command is used to exit a script. When used by itself, it provides the exit code of the last command that was run. If a number is passed as an argument it will exit with the provided code. 

The standard exit code is `0` for success and `1` to `255` for anything else. `1` is considered the catchall exit code for general errors.

Bash also has a special variable `$?` which will return the exit code of the last command run. 


## Returning From Functions

The `return` command is used within functions as the `exit` command is used for the entire script. You use the return keyword followed by a number to indicate success or failure of the function in the same manner that you use `exit` for scripts. 

Within the context of the script, the  special variable `$?` will store the last function return value.

You can use `exit` within a function, but it will terminate the entire script. This is useful when a natural end to the script would occur (invalid argument or other error).

## Arguments

Scripts can be passed space-separated arguments like any other bash command. You can access your arguments within the script with a few special variables:

- `$#` - This integer holds the number of arguments passed to the script. A value off zero means that the script wasn't given any arguments. 
- `$0` - The full pathname and filename of the script 
- `$1` - The first argument
- `$2` - The second argument
- etc...


## Reading from Standard In

You can read a line of input from standard in and store it in a variable with the `read` command. It accepts the name of the variable to be assigned as an argument:

```sh
echo "Enter a value for my_var -> "
read my_var

echo "The value of \$my_var is $my_var"
```

Multiple arguments can be supplied to assign multiple variables at the same time. If a corresponding input is omitted the variables will be empty, and if more input is provided than variables all of the trailiing input will be added to the final variable.

```sh
echo "Enter 3 values: "
read var1 var2 var3
# given input of "a b c d e"...
echo $var1      # a
echo $var2      # b
echo $var3      # c d e
```

If no variabels are listed after the `read` command, a shell variable `REPLY` will be assigned all the input.

```sh
echo "Enter one or more values: "
read

echo "REPLY = '$REPLY'"
```

`read` has numerous command line flags to change how the input is read. One common one is `-p`, which will allow you to supply a string for a prompt rather than include an `echo` command before your input.

```sh
read -p "Enter a value for REPLY: "
```

## Controlling Script Flow

The `if` statement is kind of strange and obtuse in shell scripting. `if` will check for the result of a command and perform a given set of actions preceded by a `then` statement. The block of actions is terminated with `fi`, the reverse of the `if` command.

A command is used as the argument to `if`. The `test` command exists to compare values and check filetypes, allowing you to use the `if` statement as you would in another language like Python. 

You also have the standard `elif` and `else` statements, which work as you'd expect. It's important to note that the `elif` block is terminated by the next statement rather than the reverse of `elif`. `fi` would terminate the entire chunk.

All of this nonsense looks like this in practice:

```sh
echo "Enter an integer: "
read foo

# double-quoting my vars to be safe
if test "$foo" -gt 100; then
    echo "\$foo is greater than 100."
elif test "$foo" -gt 10; then
    echo "\$foo is greater than 10 but not greater than 100."
else
    echo "\$foo is less than or equal to 10."
fi
```

## Single Brackets in `if`-statements

`test` also has a slightly cleaner looking shortcut in `[`. This is another way to use the `test` statement that expects `]` as its final argument. The same conditional block can be written like this:

```sh
if [ "$foo" -gt 100 ]; then
    echo "\$foo is greater than 100."
elif [ "$foo" -gt 10 ]; then
    echo "\$foo is greater than 10 but not greater than 100."
else
    echo "\$foo is less than or equal to 10."
fi
```

**Note that you need the spaces padding each end of the brackets.** `[` is just a command and `]` is just another argument, so they have to be separate from any other characters in order to be interpreted properly by bash.

## Double Brackets in `if`-statements: A More Modern `test`

There's another shorthand for `test` that uses two square-brackets instead of one. 

```sh
if [[ "$foo" -gt 100 ]]; then
    echo "\$foo is greater than 100"
fi
```

This option is only supported by bash and newer shells such as zsh and ksh. It's not as portable to other legacy shells and versions of Unix (and therefore not POSIX-compliant), but it allows for a few more features such as matching a string to a regular expression. 

It's probably safe to use `[[ ... ]]` instead of `[ ... ]`, but it's not the super-pure-unix-way of doing things.

## Double Parentheses in `if`-statements

Another common expression you'll encounter in `if`-statements is the double parentheses - `(( ... ))`. This will perform arithmetic (**interpreting shell variables without a dollar-sign**) and is successful if the operation has a non-zero result. 

This can be used for more C-style comparisons of values with operators such as `<`, `==`, `=>`, etc. These logical operators return `0` if the statement is false and `1` if the statement is true, allowing us to progress into the body of the `if` statement as we'd expect.

```sh
echo "Enter an interger: "
read foo

# again, note the lack of dollar-sign in the expression
if (( foo > 10 )); then
    echo "\$foo is greater than 10."
fi
```

### Arithmetic Expansion

Bash also uses the double parentheses preceeded by a dollar sign to perform arithmetic expansion. This will evaluate the arithmetic expression and return the value. 

``` sh
echo $((5+4))       # 9
```

These look similar but accomplish different tasks. 

## Single Parentheses in `if`-statements

Single parentheses are used to contain a command executed in the `if`-statement within a new subshell. This allows for a more safe execution of the command - any environmental variables that are set or modified during the execution of the command will end when the subshell completes and not propogate out to the current shell.

## Flow Control with `case` Statements

Like other languages, bash provides a control structure to use as a multiple choice compound command. This would be used in cases where you expect to match a pattern to one of a specific set of test cases and then branch program flow based on the pattern. 

The syntax is slightly different than the `if` statement but represents a more terse way of accomplishing the same thing.

```sh
read -p "Enter a word."

case "$REPLY" in
    [[:alpha:]])    
        echo "is a single alphabetic character" 
        ;;
    [ABC][0-9])     
        echo "is A, B, or C, followeod by a digit." 
        ;;
    ???)            
        echo "is three characters long." 
        ;;
    *.txt)          
        echo "is a word ending in '.txt'" 
        ;;
    *)              
        echo "is something else." 
        ;;
esac
```

Patterns are listed immediately following the `in` part of the `case` statement and are terminated with a `)`. The next block of code following from this match until a terminal `;;` will be executed. 

While not required, it's considered good practice to have a catch-all default case with a wildcard like `*)` that handles any input that wasn't expected. In this case you'd probably want to echo out to STDERR with `>&2` and follow up with a non-zero exit status to indicate that something went wrong. 

At the end of our `case` statement we terminate it in a similar manner as we would for an `if` statement by using `esac`, the reversal of `case`.

The whitespace in the above example is mostly cosmetic. The above `case` statement could also be written as such:

```sh
read -p "Enter a word."

case "$REPLY" in
    [[:alpha]])     echo "..." ;;
    [ABC][0-9])     echo "..." ;;
    ???)            echo "..." ;;
    *.txt)          echo "..." ;;
    *)              echo "..." ;;
esac
```

It is also possible to include multiple cases for a pattern with a vertical bar - `|`. This will create an "or" conditional pattern.

```sh
read -p "Enter a, b, or c: "

case "$REPLY" in
    a|A)    echo "You entered 'a'" ;;
    b|B)    echo "You entered 'b'" ;;
    c|C)    echo "You entered 'c'" ;;
    *)      echo "Come on man, you had one job. Was it really that hard?" >&2 ;;
esac
```

## Looping

`for` loops and `while` loops are possible in bash. Regardless of which type of loop you're using, the body of the loop is contained within a block started by `do` and ending with `done`.

`for` loops iterate over space-separated items:

```sh
for X in red green blue purple orange
do
    echo $X
done
```

If some of the items in your `for`-loop have spaces, they'll need to be protected in quotes.

```sh
color1="red"
color2="dark blue"
color3="light green"

for X in "$color1" "$color2" "$color3"
do
    echo $X
done
```

`while`-loops will execute the body of the loop as long as a condition is still met:

```sh
x=0

# this will output every number from 0 to 10
while [[ "$x" -le 10 ]]
do
    echo $x
    # arithmetic expansion returning our result and updating the value of x
    x=$(( x + 1 ))
done
```

There is a third loop structure, `until`, that functions as an inverse of `while`. An `until`-loop will continue until it receives a zero exit status, like so:

```sh
count=1

until [[ "$count" -gt 5 ]]; do
    echo "$count"
    count=$((count+1))
done
```

### Breaking out of a Loop

Bash provides two builtin command that can be used to control program flow inside off a loop. The `break` command will immediately terminate a loop and resume the program with the next statement immediately following the loop. The `continue` command will cause the remainder of the loop to be skipped, and the program will resume with next iteration of the loop.


### Reading Files with Loops

`while` and `until` can process standard input, which allows files to be processed with loops. The loop will continue until the final line of the file is read.

Assuming that we have a file called "people.txt", which looks like this:

```
john    28  blue
greg    23  green
nancy   34  red
sarah   22  brown
```

... we can loop over and reach each of these lines like so:

```sh
while read name age fav_color; do
    printf "Name: %s\tAge: %s\tFavorite Color: %s\n" \ 
        "$name" \
        "$age" \
        "$fav_color"
done < people.txt
```

Note that the name of the file is passed via the `<` operator to the `done` statement at the end of the loop. This loop will read through each of the lines in "people.txt" and parse each name, age, and favorite color into the specified shell variables. The loop will terminate once we've reached the end of thee file.