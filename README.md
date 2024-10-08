﻿# NAME

**ALE** - arcfide's line editor

# DEMO

```
      ⎕CS e
      RESET
      i _
hello world
.
      n _a
1  hello world
      _ t _
      n _a
1  hello world
2  hello world
      n _
2  hello world
      s'lo' 'lo happy'
      n _a
1  hello world
2  hello happy world
      u
      n _a
1  hello world
2  hello world
```

# DESCRIPTION

**ALE** is a line editor that I have created because I can. One of my all time
favorite editors to use in the classic Unix *ed(1)* editor. Line editors are,
in my opinion, grossly under-appreciated for their workflow and power as
interactive editors. With that said (or should I say, sed?), the pre-eminent
example of this in the modern computing world, *ed(1)*, does have a few issues
that I felt I could improve upon.

Furthermore, I was tired of the infrastructure setup that is required in order
to use a line editor inside of modern operating systems. They aren't convenient
to use by default in GUI environments, and there isn't a good, clean, fast way
to "just get" *ed(1)* on Windows, because so much of *ed(1)*'s design relies on
having the wider UNIX ecosystem available to it. This means, really, that if I
wanted to use *ed(1)* on Windows, I was more or less resigned to also installing
a rather heavyweight UNIX environment as well. This greatly interferes with the
simplicity of the editor itself, as well as integration.

Since almost all of my work is done with APL, I realized the potential value of
having an easy to use line editor that was written in a deeply integrated way
with the APL language. APL is itself a pretty good textual editing language, and
by simply creating a "dictionary" of editing commands, I could leverage the
power of the existing Dyalog APL session to do almost all the heavy lifting for
me.

The result is an editor that retains the benefits of the *ed(1)* editor with
some noticeable improvements in flexibility, while simultaneously making the
editor itself simpler, easier to maintain, and amenable to customized
extensions, macros, and other composability and programmability benefits
usually only seen in more complex editors, all with basically zero real
additional complexity.

In short, **ALE** is a programmable, composable, but simpler and more
streamlined, syntactically consistent replacement for the venerable
*ed(1)*.

# LINE ADDRESSING

Lines are origin-1 based references to lines in a file, with the 0th
line referencing the line before the first line in a file. Address
ranges are a pair of lines where the first line must be less than or equal
to the second line. They specify a contiguous region of lines in a file.

Traditional *ed(1)* had a number of different addressing shortcuts.
We do not have any of that in ALE at the moment, and instead prefer to
rely on the fact that we have full arithmetic expressive power through
APL. This means that the number of built-in addressing short cuts is
relatively limited. Instead, you should simply write an APL expression
to give you the appropriate range if you should want it.

<dl>
  <dt>_</dt>
  <dd>The address of the current line.</dd>
  <dt>_a</dt>
  <dd>Equivalent to `1,≢∆`.</dd>
  <dt>_s regex</dt>
  <dd>Gives the next line matching <em>regex</em> with wraparound.</dd>
  <dt>_r regex</dt>
  <dd>Gives the previous line matching <em>regex</em> with wraparound.</dd>
</dl>

# COMMMANDS

<dl>
  <dt>a line</dt>
  <dd>Appends text to the buffer after the addressed line. Input mode is used
  to enter text. The current address is set to the last line entered.</dd>

  <dt>c range</dt>
  <dd>Change the addressed range in the buffer. Replacement text is entered
  in input mode. The current address is set to the last line entered.</dd>

  <dt>d range</dt>
  <dd>Deletes the addressed lines and sets the current address to the line
  after the deleted line if there is one, or to the line before the deleted
  range if there is not.</dd>

  <dt>e file</dt>
  <dd>Edits <em>file</em> and sets the default filename. Clears and replaces
  the existing edit buffer. The current address is set to the last line read.
  </dd>

  <dt>efn expr</dt>
  <dd>Edits the vector of strings returned by <em>expr</em>. The current
  address is set to the last line read. This function replaces the use of
  the shell commands with APL expressions.</dd>

  <dt>E file</dt>
  <dd>Unconditionally edits <em>file</em>. This is like <strong>e</strong> but
  it will erase a dirty buffer without warning.</dd>

  <dt>(1,≢∆){...}g 're'</dt>
  <dd>Applies the function `{...}` to the addressed lines matching
  regular expression `'re'`. For each match, the function is applied once
  after setting the current line to the matching line. The right argument
  to the function is the line number of the match. The line number is set
  to the last line modified by one of the commands issued inside of the
  function.</dd>

  <dt>(1,≢∆)G 're'</dt>
  <dd>Works much like the `g` command except that it prompts
  interactively for the commands to run. It does this for each matching
  line by first printing the line that matches, and then prompting with
  the evaluator to ask for the next commands to run. The format of the
  commands must be suitable for execution by the `eval` function.</dd>

  <dt>i line</dt>
  <dd>Insert text before the addressed line. Text is entered in input mode.
  The current address is set to the last line entered.</dd>

  <dt>j range</dt>
  <dd>Joins the addressed range. The range is replaced with a single line
  containing the joined lines. The current address is set to the joined
  line.</dd>

  <dt>line k string</dt>
  <dd>Marks the addressed line with the given <em>string</em>. Deleting or
  modifying the line will result in that mark being deleted. Commands that
  support it may use <em>string</em> in place of a line number.</dd>

  <dt>l range</dt>
  <dd>Print the addressed lines unambiguously. Otherwise works like the
  `p` command.</dd>

  <dt>range m line</dt>
  <dd>Moves the addressed range to right after the addressed line. The current
  address is set to the last line moved.</dd>

  <dt>n range</dt>
  <dd>Prints the range with line numbers. The current address is set to the
  last line printed.</dd>

  <dt>p range</dt>
  <dd>Prints the range. The current address is set to the last line printed.</dd>

  <dt>q</dt>
  <dd>Quits **ALE**.</dd>

  <dt>Q</dt>
  <dd>Quites **ALE** unconditionally.</dd>

  <dt>line r file</dt>
  <dd>Read file into the buffer after line. Sets the default filename to file
  if there was no default filename already set. Sets the current address to the
  last line read.</dd>

  <dt>line r expr</dt>
  <dd>Works the same way as the <strong>r</strong> command with a filename, but
  if the right argument to <strong>r</strong> is a vector of strings, then this
  will be used as the contents to read into the buffer. It does not set the
  filename, and the current address is set to the last line read.</dd>

  <dt>(_,_)s pat rep pos</dt>
  <dd>Performs regex substitution. The <em>pat</em> value is a string
  containing the regex to match against. The <em>rep</em> string is a
  regex replacement pattern. The optional <em>pos</em> argument indicates
  how many replacements to do, the default being 0, meaning all possible
  replacements. If a non-zero, positive value <em>n</em> is given, then
  only the first <em>n</em> values will be replaced per line. If a negative
  value <em>-n</em> is given, then only the specific <em>n</em>th value will
  be replaced. The current line is set to the last line that was modified.</dd>

  <dt>(1,≢∆)t line</dt>
  <dd>Transfers (copies) the addressed range to right after the addressed
  lines. Current address is set to the last line transferred.</dd>

  <dt>u</dt>
  <dd>Undoes the effect of the last command that modified anything in the buffer
  and restores the current address to what it was before the command. The
  global commands 'g', 'G', 'v', and 'V' are treated as a single command by
  undo. 'u' is its own inverse; it can undo only the last command.</dd>

  <dt>(1,≢∆){...}v 're'</dt>
  <dd>Does the same thing as the `g` command but applies the
  function to lines that do *not* match 're' instead of those
  that do.</dd>

  <dt>(1,≢∆)V 're'</dt>
  <dd>Behaves like `G`, but like the `v` command, it operates on the
  not matching lines, rather than the matching lines.</dd>

  <dt>(1,≢∆)w file</dt>
  <dd>Writes the addressed lines to <em>file</em>. Previous contents of
  <em>file</em> are clobbered without warning. If there is no default filename,
  then the default filename will be set to <em>file</em>, otherwise it is not
  changed.</dd>

  <dt>(1,≢∆)wq file</dt>
  <dd>Writes buffer to <em>file</em> as with <strong>w</strong> and then executes
  the <strong>q</strong> command.</dd>

  <dt>(1,≢∆)W file</dt>
  <dd>Works like the `w` command but appends to the end of the file
  instead of overwriting the file.</dd>

  <dt>(_+1)z n</dt>
  <dd>Scrolls <em>n</em> lines at a time starting at the addressed line.
  The current address is set to the last line printed.</dd>
</dl>

**NOTE**: For any of the regex-consuming commands above—namely *_r*, *_s*,
*s*, *g*, and *v*—supplying the null regex `''` is equivalent to supplying the
previously-used regex. This "regex history" is shared among all the commands
and is initialized to the null regex.


# COMPARING ED(1) TO ALE

| ed(1)                   | ALE           | Comments
| ----------------------- | ------------- | --------
| .                       | _             | Current line
| (.)a                    | a line        |
| (.,.)c                  | c range       |
| (.,.)d                  | d range       |
| e file                  | e 'file'      |
| e !command              | efn expr      | Use APL, not *sh(1)*, expr must return vector of strings
| E file                  | E 'file'      |
| f                       | ⍙             | Prints default filename
| f file                  | ⍙←'file'      | Sets default filename
| (1,$)g/re/command-list  | (1,≢∆){...}g're' | ⍵ is _ for each match
| (1,$)G/re/              | (1,≢∆)G're' |
| H                       | N/A           | **ALE** errors are more verbose
| h                       | N/A           | See H
| (.)i                    | i line        |
| (.,.+1)j                | j range       |
| (.)k lc                 | line k string |
| (.,.)l                  | l range |
| (.,.)m(.)               | range m line  |
| (.,.)n                  | n range       |
| (.,.)p                  | p range       |
| P                       | N/A           | APL session prompt is used
| q                       | q             |
| Q                       | )off          |
| ($)r file               | line r file   |
| ($)r !command           | line r expr   | `expr` must return vector of strings
| (.,.)s/re/replacement/  | (_,_)s 're' 'replacement' 1  |
| (.,.)s/re/replacement/g | (_,_)s 're' 'replacement'    |
| (.,.)s/re/replacement/n | (_,_)s 're' 'replacement' ¯n |
| (.,.)s                  | N/A                          | Use the history mechanism
| (.,.)t(.)               | range t line |
| u                       |               | Not yet implemented
| (1,$)v/re/command-list  | range {...}v 're' |
| (1,$)V/re/              | range V 're' |
| (1,$)w file             | (1,≢∆)w file  | Use ⍙ for default file
| (1,$)wq file            | (1,≢∆)wq file | Use ⍙ for default file
| (1,$)w !command         | fn ∆          | Apply APL function to buffer
| (1,$)W file             | (1,≢∆)W file  | Use ⍙ for default file
| (.+1)z n                | (_+1)z n      |
| ($)=                    | N/A           | Ranges print themselves
| (.+1)newline            | N/A           | Use z instead
| !command                | ⎕SH 'command' | See Dyalog Documentation on ⎕SH

# TIPS AND TRICKS

Because **ALE** is basically just a slightly optimized APL session, you have
at your disposal the full power of APL. This leads to some interesting features
that **ALE** can "have" if you want it to.

# FILES

**ALE** does not currently use a file buffer. This means that files are loaded
into memory and stored there. This limits the size of files that can be loaded
to the size of your main memory and the swap space that you have available on
your machine.

# SEE ALSO

[The OpenBSD ed(1) man page](https://man.openbsd.org/ed)

# DIAGNOSTICS

All commands return an error code as the shy result of their execution. A 0
value indicates success, and a non-zero value indicates the type of error that
occured.

# CAVEATS

**ALE** is very new software right now, and it may not be fully stable. Beware
of this before using it in a mission critical environment.
