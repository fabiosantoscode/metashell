
# metashell

This is an idea for enhancing standard input and standard output streams. It is opt-in, so it shouldn't break any of our old trusty commands, but should allow new commands to communicate better with each other, and with us humans.


# the problems
At the core of every unix-like operating system is the shell and the text streams. Piping these text streams from unix tool to unix tool is at the cornerstone of the unix philosophy, but the fact remains that text streams are just text.

The content format of these text streams is completely ad hoc. The data formats are many, the way tool developers format the data is free-form. Once a program is released, the output format cannot be changed, since other programs will be piping data from that one, and expect it to be in a certain format.

Think of a `foo` command. It counts the `foo`s in your system and prints the output:

        1500600.10 foos

Imagine a shell script `bar` which used this output. If the `foo` command was internationalized so as to display the number in a localized format (perhaps use a comma to split the decimal places, or split the thousands with `'` characters), this would break that shell script.

Adding decorations like ascii tables or horizontal lines and whatnot, are problematic because this modifies the output format. Writing text around the data (like you see below) is also not advisable. It's practically impossible to parse if you are to expect a few changes in the text or line wrapping.

        DANGER: /tmp directory has 123.1 foos
        1500600.10 foos were totally found in your system.
        123.1 were in your /tmp directory. You can always clear it.
        The maximum recommended is, like, 10. Take care!

There are ways to get around these problems. Some programs can be asked to produce machine-readable output (such as with the `--porcelain` switch in every git subcommand) and otherwise produce user-readable output, free-form. You can set one of the environment variables `$LANG, $LC_ALL` and company before calling someone else's tool, and have the output fed through the pipes in a known format. However, it's still text and the programmer still has to format it downstream, only to output his own output, which another programmer will have to format, and it's a waste of time all the way.

The PowerShell shell, developed by Microsoft, is a system that tries to break out of this paradigm by turning these text streams into actual object streams. They have known data types.
<!-- TODO write more about powershell when internet comes back up -->


## the graphical shell

Another hindrance we have is the text-based shell. Although I love it and use it intensively, the unix community has to admit that it is intimidating to non-power users, and provides almost no integration whatsoever with the rest of the system.

The exceptions are copy-paste which doesn't work well with anything which isn't pure text (and this even includes the `vim` text editor when you turn on line numbering, because you copy the line numbers along with your text), and highlighting URIs for opening them in browsers. We could have a lot more, but since it's all just text, it's hard to parse correctly with regular expressions, and even URI parsing (which is just a simple pattern, often breaks.

We can't have graphs, images and live views from our commands in our text-based command line. We could build a graphical shell, with proper highlighting, icons, live data updates, integrated sorting and filtering tools, right-click actions for exporting, archiving, sharing command results, actions specific to files, external resources, e-mails, phone numbers, but not out of just text streams.


## metadata for communicating with GUI applications

Since text streams do not have defined meta data, standard output from a tool is not easily understood by GUI applications.

This is one of the reasons why unix tools are mostly command-line based and don't have good GUI frontends. It is also why we cannot have our spreadsheet program access the untapped potential of data available to our hands in the command line.

We can create a script which `curl`s some web service and outputs a table with data found in it, together with the date and time when it is expected to be outdated. We can append a note saying "to refresh, run this command: `myscript data-address <your-password>`". But our spreadsheet program which we (painfully) paste that data into will not warn us that the data is getting outdated, cannot read the data all by itself, will ask us what format every column is in, and we should pray that there's nothing in there outside latin-1.


# This specification

This document attempts to describe a manner by which programs and tools can, by sending metadata along with the regular information, better communicate among themselves, with GUI applications, the desktop environment, and with the humans who read the output data.

These are the advantages it should bring

 - Ease of building command line tools to communicate with users and computers at the same time
 - Ease of interoperability between unix tools
 - Give the unix community the ability to, improve our text terminals by giving them more information about what the programs are outputting so as to allow them to integrate with the desktop environment
 - Ability to have rich, and easier to build, interactions between GUI tools and classical unix tools
 - 

And perhaps, be a step towards the Data Ecosystem we keep dreaming about.


# The data type format

Data types are MIME types starting with `x-type/*`. For example, an integer number is `x-type/int`, a HTTP URI is `x-type/http` and a phone number is `x-type/tel`.

Data is to be outputted in standard, widely known formats. When something is supposed to be relative to some other data, the "base" data should be specified. An example is a phone number list. If the elements in the list of phone numbers are relative to a certain area code, the area code is specified in the list itself, or in each element in the list, never to be left to interpretation of context. Computer programs tend to be terrible at that.


# The markup

It's a html/xml/sgml syntax, mainly because there are great tools and libraries to build and parse html.

Here is example output for the `ls` command:

		<!metashell>
		<list type="x-type/path" base="/home/fabio" unordered>
		<human>Contents of "~/":</human>
		<el>folder 1</el>		<el>folder 2</el>
		<el>amazing film.flm</el>	<el>ammz</el>
		<el>some article.html</el>	<el>some other article.html</el>
		<machine>
		<el>.ssh</el>
		<el>.vimrc</el>
		<el>.bashrc</el>
		<el>.secret folder</el>	
		</machine>
		<human>That's it.</human>
		</list>

Who sees what?

First, nobody sees the `<!metashell>` tag. That's supposed to be used only by the shell reading the output, and is useful to mark up metashell output when copying and pasting data around.

 - The user sees everything except for the contents inside the angle brackets (`<>`), and the contents inside the `<machine>` part. The `<machine>` part is meant to be read by programs only.

 - metashell-unaware programs see what the user sees, except for `<human>` tag contents, which the metashell-aware shell will strip away along with the other tags. metashell-unaware programs can see the `<machine>` parts.

 - metashell-aware programs can see the whole document as a text stream. By parsing it they obtain a list of paths. By querying this list for its attributes they can know more about the list and the data inside it. A metashell-aware terminal can offer to sort the list by any of the attributes associated with the file path, and since it knows they are paths, it can provide options to open the file or folder, upload it, e-mail it, etc.

The text contents of each `<el>` tag are to be concatenated (as strings) to the "base". This is because `x-type/path` is declared (somewhere else in this document, TODO) to be `concatenative`. `additive` types are to be added to the base, for example relative timestamps. If no base is declared, the data is supposed to be absolute.

Basic attributes are standardized in this document, and everyone can add new attributes, but they are asked nicely to prefix them with `x-*` if they are only used in their application or system.


# Opting in

metashell operates mainly by marking up the standard output stream. This would be just gibberish to a user, so there are ways to opt in and opt out of metashell's structured output.

The first part of the opt-in is the `$METASHELL_AWARE` environment variable. It should be set to a nonzero number and visible to the program which is going to output metashell stuff. The nonzero number indicates the version of this protocol, starting in 1 and always being an integer.

Only if the `$METASHELL_AWARE` environment variable is set to a nonzero number, to state that its output is metashell-enabled, a program starts it with the doctype-like `<!metashell>`. It should be in the first 300 characters, to be compatible with utf-8 BOM, text editor directives, shebang lines, and whatnot. When not to be taken as a metashell doctype-like, any non-metashell text containing a `<!metashell>` tag should have it escaped by replacing it with `<!metametashell>`. That way the system will know that it's not the start of wrapped content, and will treat it as just garbage with the `<!metashell>` characters in it.


## more to come
