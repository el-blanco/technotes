# handling input and output

[Advanced Bash Scripting Guide](https://tldp.org/LDP/abs/html/) - A nice reference guide with lots of examples

## read

useful read options
```
	-p prompt
	-t timeout
	-s (=don't echo input)
	-n bytes (only read first 'bytes' bytes of input)
	-a arry
```

save read input to custom variable
```
$ read -p "enter a foobar value > " FOOBAR
enter a foobar value > 42    

$ echo $FOOBAR
42
```

read multiple args
```
$ read -p "enter space-delimited args here > " arg0 arg1 arg2 more_args
enter space-delimited args here > one too trey "lots more"

$ echo "$arg0 : $arg1 : $arg2 : $more_args"
one : too : trey : "lots more"
```
### Reading a file line by line

First, create a simple file

```
cat - > list.txt <<EOF
item 1
item 2
items 3 4 and 5
EOF
```
This works
```
while read line
do
   echo "line is : '$line'"
done < list.txt
```
This also works
```
cat list.txt | while read line
do
   echo "line is : '$line'"
done
```
I'm not aware of any differences between using file redirect vs. piping stdout

This does _not_ work
```
while read line < list.txt
do
   echo "line is : '$line'"
done
```
It doesn't work because

```read line < list.txt```

is interpreted as the full condition to the while statement, and reads always the first line of list.txt.  IE, it reads logically as

```while (read line < list.txt)```

instead of what was probably intended:

```(while read line) < list.txt```

_Note: the parentheses above are shown for demonstrating logical grouping - bash doesn't permit their use in this manner._

---
## I/O redirection

This captures stderr (redirects to stdout) globally within the script or scope of the function.  This only works when called within the context of a script, IE, not at an interactive shell prompt.

```exec 2>&1```

Save stdout to a file, then restore stdout to terminal (use lsof to see what's going on):
```
lsof -a -p $$ -d0,1,2,3

exec 3>&1
exec >/tmp/savefile
lsof -a -p $$ -d0,1,2,3

exec >&3
exec 3>&-
lsof -a -p $$ -d0,1,2,3

cat /tmp/savefile
```
note that the exec's can be combined:
```
exec 3>&1 >/tmp/savefile
...
exec >&3 3>&-
```
this is cool.  read and write from a tcp socket like a file:

```
## FIXME
> /tmp/input
> /tmp/output
tail -f /tmp/input | nc localhost 80 > /tmp/output &


exec 3>/tmp/input  ## this is the "write" channel (to the socket)
exec 4</tmp/output ## this is the "read" channel (from the socket)
tail -f /tmp/input | nc localhost 80 > /tmp/output &
echo -ne "GET / HTTP/1.1\r\nHost: localhost\r\n\r\n" >&3
cat <&4

exec 3>&-
```

---
## shopt
when invoking /bin/bash instead of /bin/sh (when for example /bin/sh is symlinked to /bin/dash), certain shell options are disengaged, like bash aliases.  To see how the options differ, try

```
$ /bin/bash -c shopt > shopt.scripted
$ shopt > shopt.interactive
$ sdiff -s shopt.scripted shopt.interactive
checkwinsize       off                          |    checkwinsize       on
expand_aliases     off                          |    expand_aliases     on
extglob            off                          |    extglob            on
hostcomplete       on                           |    hostcomplete       off
```

This is what it looks like on my ubuntu 8.10 system.  To change the options within the script just set them, typically at the head of the script:

```
#!/bin/bash
shopt -s expand_aliases   # enable (set)
shopt -u hostcomplete     # disable (unset)
```

---
## Restrict shell when sourcing files
If you have functions that are, say, bash-specific, you can prevent non-bash shells from sourcing the file.  Place the following code at the top of the file to be sourced (probably only works when sourcing from bourne-shell derivatives like dash or ash)

```
if ! test "${BASH}"
then
        echo "you must source from bash shell"
        false
        return
fi
```