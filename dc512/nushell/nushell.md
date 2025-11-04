---
title: "Nushell - Like Powershell, but Functional"
sub_title: "or, How I Learned To Stop Worrying And Love POSIX Noncompliance"
author: Mussar
theme:
    name: catppuccin-mocha
---

whoami
---

```bash
❯ : sys users | where name == mussar
╭───┬────────┬─────────────────────────────────────────╮
│ # │  name  │              about me                   │
├───┼────────┼─────────────────────────────────────────┤
│ 0 │ mussar │ ╭────┬────────────────────────────────╮ │
│   │        │ │  0 │ appsec @ openlending           │ │
│   │        │ │  1 │ dc512 helped me get this job   │ │
│   │        │ │  2 │ nushell daily driver (~2 years)│ │
│   │        │ │  3 │ occasional nushell contributor │ │
│   │        │ │  4 │ python, golang, & nushell      │ │
│   │        │ │  5 │ i use neovim (btw)             │ │
│   │        │ │  6 │ (former pro) music producer    │ │
│   │        │ │  7 │ (former pro) sound designer    │ │
│   │        │ │  8 │ (new) 3d printing enthusiast   │ │
│   │        │ ╰────┴────────────────────────────────╯ │
╰───┴────────┴─────────────────────────────────────────╯
```

<!-- end_slide -->

what is nushell?
---


<!-- alignment: center-->
- cross-platform (linux, macos, windows) shell written in rust
- functional pipe-driven programming language
- rejects POSIX compliance to allow higher level functionality
- native support for parsing .json, .csv, .yaml/yml, .xml, SQLite, Excel docs,...
- first party plugin support for dataframes, .eml, .ics. ini, .plist, .jsonl, ...
- native support for HTTP requests, string manipulation, scoped envs, parallelism
- extensible via third party plugins (e.g. parsing HCL, PCAPs, x509 certs, system clipboard support, etc)
- allows for custom completion formatting for shell commands
- optional mcp support (if you're into that sort of thing...)

```bash
❯ : http get https://zenquotes.io/api/random | select q a
╭───┬──────────────────────────────────────────────────────────────────────────────────────┬──────────────╮
│ # │                                          q                                           │      a       │
├───┼──────────────────────────────────────────────────────────────────────────────────────┼──────────────┤
│ 0 │ Obstacles are those frightful things you see when you take your eyes off your goals. │ Sydney Smith │
╰───┴──────────────────────────────────────────────────────────────────────────────────────┴──────────────╯
```

<!-- end_slide -->

why do i use nushell?
---

<!-- alignment: center-->

- went on a "how far can i go with an 'only rust' approach to the CLI" bender
- came for the rust, stayed for the closures and structured data parsing
- forced me to think about my terminal in a new way, which is pretty fun
- fast, like bash; higher-level, like powershell; simple, like python
- to spite people who say things like "never use anything but defaults" /s

```nushell +exec
[m u s s a r] | each {|letter|
    if $letter in [a e i o u] {
        print $"gimme a ($letter)!" 
    } else {
        print $"gimme an ($letter)!"
    }
};
print "what's that spell???";
print "MUSSAR!"
```

<!-- end_slide -->
nushell 101: nushell basics
---

<!-- column_layout: [1, 1] -->

<!-- column: 0-->
nushell has your standard primitives like integers, floats, strings, and booleans.
it also has numeric types for dates, durations, and file sizes.

```bash
# dates:
> : 2025-11-03
Mon, 3 Nov 2025 00:00:00 +0000 (19 hours ago)
# durations
> : 1.5day
1day 12hr
# file sizes:
> : 1mB
1.0 MB
```

nushell uses closures `{}` to handle tasks like iteration and conditionals:

```bash
# iterate through a range:
> : 3..1 | each { print $in }; print blastoff
3
2
1
blastoff
# branch based on conditional execution
> : let dc512_is_cool = true
if $dc512_is_cool { print "you're god damn right it is" } else { print "shouldn't execute" }
you're god damn right it is
```

<!-- column: 1-->
lists are collections of items, like an array.
a record is a set of key value pairs, like a dictionary.
the primary data type in nushell is the table: a list of records

```bash
# list:
> : let cool_numbers = [42 69 420]
╭───┬─────╮
│ 0 │ 420 │
│ 1 │  69 │
│ 2 │ 420 │
╰───┴─────╯
# record:
> : let my_stats = {name: mussar, age: 0x25, favorite_color: purple}
╭────────────────┬────────╮
│ name           │ mussar │
│ age            │ 37     │
│ favorite_color │ purple │
╰────────────────┴────────╯
# table:
> : let some_table = [{column_a: foo, column_b: bar}, {column_a: 100, column_b: 1000}]
# or
> : let some_table = [[column_a column_b]; [foo bar] [100 1000]]
╭───┬──────────┬──────────╮
│ # │ column_a │ column_b │
├───┼──────────┼──────────┤
│ 0 │ foo      │ bar      │
│ 1 │      100 │     1000 │
╰───┴──────────┴──────────╯
```

<!-- end_slide -->
nushell 101: nushell commands
---

<!-- alignment: center-->
many common unix commands have been reimplemented in nushell to support tables.

for example, compare the output of the standard posix `ls -lh` vs the nushell `ls`:

```bash +exec +no_background
ls -lh ~/workspace/github.com/nushell
```

```nushell +exec +no_background
ls ~/workspace/github.com/nushell/
```

<!-- end_slide -->

nushell 101: nushell queries
---

<!-- alignment: center-->
because the output of most nushell commands are rows and columns, you can perform
operations on the output like with most shells that are context-dependent on the
schema:

```nushell +exec +no_background
ls ~/workspace/github.com/nushell/nushell/ | sort-by type
```

```nushell +exec +no_background
ls ~/workspace/github.com/nushell/nushell/ 
| sort-by type modified # sort by type, then by modified date
```
<!-- end_slide -->

nushell 101: finding what you want
---

<!-- alignment: center-->
in addition to filters, you can grab specific rows and columns in order to isolate
the specific contents you're searching for, and use that as the input for later
commands.

`select` creates a new table with specific rows/columns, `get` pulls out the value:

```nushell +exec +no_background
ls ~/workspace/github.com/nushell/nushell/
| where type == dir and modified  > (date now) - 4wk # directories changed within the past 4 weeks
| get name
| each {|directory| ls $directory | select name size } # can also use $in, e.g. each { ls $in }
```

```nushell +exec +no_background
ls ~/workspace/github.com/nushell/nushell/ 
| where type == file | sort-by size --reverse
| first 2 | get size | math sum # get total filesize of 2 largest files
```

<!-- end_slide -->

nushell 101: reading a structured file
---

<!-- alignment: center-->
in addition to formatting the output of coreutils binaries, nushell can parse
structured data files like JSON, YAML, and TOML and convert them into tables:

(if you want to use the original version of open, use the extern flag (`^`) or
the nushell builtin `start` for the same effect. e.g. `^open foo` and `start foo`
are roughly identical in behavior.)

```nushell +exec +no_background
open ~/workspace/github.com/nushell/nushell/Cargo.toml
```

<!-- end_slide -->

nushell 101: extracting data from a structured file
---

<!-- alignment: center-->
pulling data from a table can be done through sequential `select`/`get` statements
or by indexing via cell paths.

pull out row elements with index numbers and column elements or keyed values
with their associated names:

```nushell +exec
open ~/workspace/github.com/nushell/nushell/Cargo.toml | get package.version
```

```nushell +exec
open ~/workspace/github.com/nushell/nushell/Cargo.toml | get bin.0.path
```

<!-- end_slide -->

practical application: nmap scans
---

![image:width:50%](./Logo_nmap.png)

<!-- end_slide -->

nmap scans: finding my ip address
---

<!-- alignment: center-->
nushell allows for 'subcommands'; for example the `sys` command wraps a number
of common system binary commands like `sys cpu`, `sys mem`, `sys hosts`, and `sys net`:

```nushell +exec +no_background
sys net | where name == en1
```

we can filter through this in order to pull out our 1pv4 address on the local network:

```nushell +exec +no_background
sys net | where name == en1 | get ip.0 | where protocol == ipv4 | get address.0
```

the raw output of sys net, for reference:

```nushell +exec +no_background
sys net 
```

<!-- end_slide -->

nsmap scans: scan the /24 subnet (feat. fstrings)
---

<!-- alignment: center-->
this example uses the `$in` keyword in two ways: first, passing the string as-is
into the `--exclude` flag so nmap does not scan localhost; and second, inside a
format-string (`$""`) for interpolation:

```nushell +exec
sys net 
| where name == en1 | get ip.0 | where protocol == ipv4 | get address.0
| sudo nmap -vvv --exclude $in -A -T3 $"($in)/24" -oA dc512
```

<!-- end_slide -->

parsing the results
---

<!-- alignment: center-->
we can parse the xml file using the `from xml` conversion tool and pull out
specific attributes.

nmap's xml output is... kinda wonky, so forgive the strange parsing rules.

tl;dr is that i'm looking for any hosts that responded to a scan and have any
number of open ports, then returns the hostnames/IPs and open ports:

```nushell +exec +no_background
open ~/workspace/nmap/dc512/dc512.xml --raw
| from xml --allow-dtd
| get content
| where tag == host and content.0.attributes.state == up
| each {|row| return {
    hostname: ($row.content | where tag == address | get attributes),
    ports: ($row.content | where tag == ports | get content.0 
    | where tag == port and content.0.attributes.state == open 
    | get attributes) } }
```
<!-- end_slide -->

<!--jump_to_middle-->
THE END
---

<!-- end_slide -->

next steps
---

<!-- alignment: center-->
if you want to learn more, check out the nushell book at [https://nushell.sh/book](https://nushell.sh/book)!

this talk barely scratches the surface of what nushell can offer, so check it out
and come talk to me if you have any questions or want to see some more advanced
features like scoped environment variables, parallel processing, background jobs,
and the directory stack!

<!-- end_slide -->

`exit`
---
