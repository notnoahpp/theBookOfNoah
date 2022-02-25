# bash scripting

- most of this should have come straight from the bash pocket ref (bash v4.4)
- hopefully this will be my last bash cheatsheet and I can simplify `

- reading 96
- copying page 24 function traps

## links

- likely should check out the scripting.md file for links

- [heredoc](https://linuxize.com/post/bash-heredoc/)
- [herestring](https://bash.cyberciti.biz/guide/Here_strings)

## basics

### terms

- bourne shell
  - original bourne shell was 1979 via the V7 Unix
  - found in `/bin/sh` (but I think somewhere this is no longer sh, but some other version on new systems)
- POSIX standard: defines the `standard shell` language & behavior based on the System V Bourne SHell
- Bash: by the Free Software Foundation; clone of the Bourne Shell written from scrach

  - in/output redirection
  - wildcard chars for filename abreviatiation
  - shell vars & options for customizing the env
  - builtin command set for writing shell programs
  - shell functions
  - job control
  - command-line editing (using the cmd syntax of vi/emacs)
  - editable cmd history
  - integer arithmetic
  - arrays and airthmetic expressions
  - cmd name abbreviation (aliases)
  - upwards compliance with POSIX
  - internationalization facilities
  - arithmetic for loop

- shell types
  - login shell:
    - whenever the `-l` option is set
    - reads `/etc/profile`
    - reads the first found: `~/.bash_profile > ~/.bash_login > ~/.profile`
  - nonlogin shells:
    - if invoked as `sh` or with `--posix` reads $ENV
    - else reads `~/.bashrc`
  - interactive shell:
    - uses the prompt for input
    - reads `~/.bashrc`
  - restricted shell:
  - priveledged shell:
    - do not read $ENV or $BASH_ENV
    - do not import functions from the environment
    - ignore values in BASHOPTS, CDPATH, GLOBIGNORE, and SHELLOPTS

### concepts

- how bash reads scripts
  - one line at a time
  - parses each line completely before executing any of the cmds on that line
    - you cannot define an alias ad use it on the same line
    - cmds that affect script parsing should be placed before the lines they affect
- how bash reads functions
  - functions are parsed all at once
    - options enabled in one function are thus enabled in all functions
    - thus dont enable/disable options within functions, but at the very top of scripts

### files and directories

- `/bin/sh` usually a link to bash
- `/dev/fd/n` a duplicate of file descriptor n
- `/dev/stderr` duplicate of file descriptor 2
- `/dev/stdin` duplicate of file descriptor 0
- `/dev/stdout` duplicate of file descriptor 1
- `/dev/tcp/host/port` bash opens tcp connection to host (hostname/ip address) on port, and uses the file descriptor in redirections
- `/dev/udp/host/port` same as the tcp counterpart, but for udp
- `/etc/passwd` the user database (except on networked systems)
- `/etc/profile`
- `~/.bash_logout` read when an interactive shell exits, or `exit` builtin cmd (non-interactive shells)
- `~/.bash_profile`
- `~/.bashrc`
- `~/.profile`

### env vars (move to separate file when focusing on these)

- BASH_ENV
- BASHOPTS
- CDPATH
- ENV
- GLOBIGNORE
- SHELLOPTS
- $? exit status of the previous executed cmd
- $PWD
- $OLDPWD
- $PS0-4 prompt strings

### arguments

- bash arguments are assigned to positional params $1, $2, etc
- if the first arg is a script
  - cmds are read from the script
  - the scrpit doesnt need to be executable, but must be readable
  - the script path is assigned to $0

### exit status

- when a cmd exits, it provides a numerical exit status between 0-255
  - external cmds (e.g. `ls`) provides the value to the operating system
  - internal cmds (e.g. `cd`) provides this value directly to the shell
- cmd exit values (by convention)
  - 0: the only one that === true/success
  - 2: usage errors
  - 126: cmd was found but not executable
  - 127: cmd not found
  - 128 + N: cmd died due to receiving signal number N

### filename metacharacters

- `*` any string of zero/more chars
- `?` singel char
- `[abc...]` one of the enclosed characters
- `[a-z...]` range of characters
- `[!abc...]` any char not within brackets
- `~` home dir of cur user
- `~username` home dir of username
- `~+` $PWD
- `~-` $OLDPWD
- requires `extglob` to be enabled
  - `?(pattern)` match zero/one
  - `*(pattern)` match zero/more
  - `+(pattern)` match one/more
  - `@(pattern)` exactly one
  - `!(pattern)` anything except pattern
- requires `globstar`
  - `**` match all files & zero/more sudirs
  - `**/` only match dirs & subdirs
- character classes `[[:poop:]]`
  - alnum: alpha numeric
  - alpha: alphabetic
  - ascii: ASCII (not posix)
  - blank: space/tab
  - cntrl: control characters
  - digit: decimals
  - graph nonspace characters
  - lower: lowercase chars
  - print: printable chars
  - punct: punctuation
  - space: whitespace
  - upper: uppercase
  - word: not posix,
  - xdigit: hexadecimal digits

### brace expansion

- pre{X, Y[, Z...]}post === preXpost, preYpost, etx
- pre{start..end[..incr]}post
  - start & end signify ranges, e.g. 1..10, a..z
  - incr is an integer, determining how to increment (e.g. by 1 defualt, 2 etc)

### escape sequences

- occur in 4 contexts
  - the $'...' quoted string
  - arguments to echo -e and printf %b
  - format strings for printf
  - value of PS0-4 prompt strings

```bash
  # i skipped a bunch, swing back through later
  \b # backspace
  \c # supress the terminating new line, do not print any following chars
  \f # formfeed
  \n # newline
  \r # carriage return
  \t # tab

```

### special characters

- cmd execution
  - `;` cmd separator
  - `&` bg execution
  - `()` cmd grouping
  - `\`` tilda is for cmd substitution
- `|` pipe
- `< > &` redirection symbols
- `* ? [ ] ~ + - @ !` filename metacharacters
- `" ' \` used in quoting other chars
- `$` variable/cmd/arithmetic substitution
- `#` single line comment
- `"` literal quoting + special chars
  - `$ \`` still maintain their special meaning
- `'` literal quoting no special chars
- `$"..."` just like " but locale translation is performed
- `$'...'` similar to ' but the qouted text is processed for escape sequences

### cmd execution

```bash
  cmd & # execute cmd in the backgroun
  cmd1; cmd2 # cmd sequence, execute multiple cmds sequetially on the same line
  {cmd1; cmd2} # cmd group; in the current shell
  (cmd1; cmd2) # cmd group: in a subshell
  cmd1 | cmd2 # output of cmd1 as input to cmd2
  cmd1 `cmd2` # cmd substitution; cmd2 output as args to cmd1
  cmd1 $(cmd2) # cmd substution (POSIX)
  cmd $((expression)) # POSIX shell arithmetic substitution; expr output as args to cmd1
  cmd1 && cmd2 # AND short circuit; execut cmd1, if success, execute cmd2
  cmd1 || cmd2 # OR short circuit; execute cmd1, if failure, execute cmd2
  !cmd # NOT; execute cmd, flip the exit status of cmd
```

### redirection

- file descriptors: can be reasigned, but defaults are
  - 0 stdin, keyboard
  - 1 stdout, screen
  - 2 stdout, screen
  - in redirections, you can use {poop} instead of a number
    - bash auto picks a number greater than 9 for the file descriptor
    - bash assigns the picked number to variable $poop,
    - you can use $poop anywhere in the script, e.g. with exec
    - you have manually close the file descriptor saved in $poop
- /dev/null is a useful place to send stderr

```bash
  # no spacing is allowed between file descriptors and a redirectin symbol
  # simple redirection
  cmd > file # create/overwright file
  cmd >> file # create/append to file
  cmd < file # file contents (read only) is input to cmd
  cmd <> file # file content (read+write) is input to cmd
  cmd >| file # create/overwight file, ignoring noclobber option
  cmd <<< "this is a here string, check the nixCraft link"
  cmd <<- poop
    # pass multiple lines of input, the cmd is actually optional
    # if poop is unquoted, this block of text will undergo variable, cmd & arithmetic substitution
    # the - ignores all leading tabs, which is what you want for formatting this block
    # no whitespace can exist before the ending poop
    # poop can also be a variable
    # check the linuxize links for more juiciness
poop

  # single redirection using file descriptors
  cmd >&n # send output to n
  cmd m>&n # send output AND m to n
  cmd >&- # close standard output
  cmd >&n- # move n to standard output by duplicating it, then closing the original

  cmd <&n # cmd input from n
  cmd m<&n # cmd input from n, and any input from m also comes from n
  cmd <&- # close standard input
  cmd <&n- # move n to standard input by duplicating it, and closing the original

  # multiple redirection
  cmd 2> file # stderr to file
  cmd > file 2>&1 # stdout and stderr to file
  cmd >& file # stdout and stderr to file
  cmd &> file # stdout and stderr to file (preferred)
  cmd &>> file # append stdout & stderr to file
  cmd > fileA 2> fileB # stdout to fileA, stderr to fileB

  # using tee to duplicate output
  cmd | tee files # send output to stdout and files
  cmd 2>&1 | tee files # pipe stdout and stderr of cmd to tee and to files
  cmd |& tee files # pipe stdout and stderr of cmd to tee and to files

```

### quotes in bash

```bash
  echo "a string plus cmd output `ls`"
  echo "the value of \$x is $x"
  echo $'single quote with \t escape sequences'

```

### process substitution

- a way to create non-linear pipelines
- not available in POSIX, systems that dont support named pipes (FIFOS), systems that dont support accessing open files via filenames in `/dev/fd`

```bash
  # in all cases
  # run cmd with its input connected to a named pipe/open file in /dev/fd
  # place the file's name in the argument list of cmd

  # cmd can read the file to see the output of command
  # i.e. cmd should periodically check the file, to see the output of command
  cmd <(command)

  # output written by cmd to the file is input to command
  # i.e. output from cmd to file, is input to command
  cmd >(command)

```

## functions

- faster than and preferred over aliases
- funtion arguments are received as positional params identical to shell scripts
- redirections in the function definition are evaluated when the fn is invoked
- functions whose name dont include = or / can be exported with `export -f`
- function traps
  - page 24

```bash
  # POSIX format
  fn_name () {
    # some code
    # $0 = script name
    # $1 = fn arg 1, etc...
    exit 1 # will exit the entire script with status 1, not just the fn
    return 0 # will return exit status 0 to the caller
  } # redirections here

  # BASH format
  # () after fn_name are optional
  function fn_name {
    # same as posix
  } # redirections here
```

## control flow

### if

### while

### until

## todos: i want to capture these but categorize them later

- likely these should be a separate file, as all of these sections are really fkn super long

### invoking bash

```bash
  bash [options] [arguments]
  # options
    --debugger # read the debugger profile if its available at startup, turn on extdebug option to shopt
    --help # print help
    --noediting # do not use readline library for input
    --noprofile # do not read any startup rc files or /etc/profile
    --norc # do not read ~/.bashrc (e.g. when invoked as sh)
    --posix # turn on POSIX mode
    --version # print version
    -, -- # end option processing
    -c str # read cmds from string
    -D, --dump-strings # print all $".." strings in the program
    -i # create an interactive shell
    -init-file FILE, --rcfile FILE # use FILE instead of ~/.bashrc for interactive shells
    -l, --login # behave as a login shell
    -O SOME_OPT # enable some_option
    -p #start as a privileged user
    -r, --restricted # create a restricted shell
    -s, read cmds from stdin, builtin cmd output > descriptor 1, all other > descriptor 2
    -v, --verbose # print lines as the shell reads them
    +O SOME_OPT # disable some_option
  # options to enable/disable via -O/O+
    extglob # extended shell patterns, see filename metacharacters
    globstar # see ** in filename metacharacters
    noclobber

```

### bash scripts

```bash
  shopt -s extglob # enable extended shell patterns
```