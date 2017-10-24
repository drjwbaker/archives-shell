---
title: "Counting and mining with the shell"
teaching: 60
exercises: 30
questions:
- "How can I count data?"
- "How can I find data within files?"
- "How can I combine existing commands to do new things?"
objectives:
- "Demonstrate counting lines, words, and characters with the shell command wc and appropriate flags"
- "Use strings to mine files and extract matched lines with the shell"
- "Create complex single line commands by combining shell commands and regular expressions to mine files"
- "Redirect a command's output to a file."
- "Process a file instead of keyboard input using redirection."
- "Construct command pipelines with two or more stages."
- "Explain Unix's 'small pieces, loosely joined' philosophy."
keypoints:
- "The shell can be used to count elements of documents"
- "The shell can be used to search for patterns within files"
- "Command can be used to count and mine any number of files"
- "Commands and flags can be combined to build complex queries specific to your work"

---
##  Counting and mining data

Now that you know how to navigate the shell, we will move onto
learning how to count and mine data using a few of the standard shell commands.
While these commands are unlikely to revolutionise your work by themselves,
they're very versatile and will add to your foundation for working in the shell
and for learning to code. The commands also replicate the sorts of uses library users might make of library data.

## Counting and sorting

We will begin by counting the contents of files using the Unix shell.
We can use the Unix shell to quickly generate counts from across files,
something that is tricky to achieve using the graphical user interfaces of standard office suites.

Let's start by navigating to the directory that contains our data using the
`cd` command:

~~~
$ cd shell-lesson
~~~
{: .bash}

Remember, if at any time you are not sure where you are in your directory structure,
use the `pwd` command to find out:

~~~
$ pwd
~~~
{: .bash}
~~~
/Users/riley/Desktop/shell-lesson
~~~
{: .output}

And let's just check what files are in the directory and how large they
are with `ls -lh`:

~~~
$ ls -lh
~~~
{: .bash}
~~~
total 9672
-rw-rw-r--  1 jb677  1322   582K 17 Oct 15:45 33504-0.txt
-rw-rw-r--  1 jb677  1322   598K 17 Oct 15:45 829-0.txt
-rw-r--r--@ 1 jb677  1322   920K 20 Oct 16:55 TNA-download-FO-881-file-level-only-csv.tsv
-rw-r--r--@ 1 jb677  1322   1.2M 20 Oct 16:56 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
-rw-r--r--@ 1 jb677  1322   1.0M 20 Oct 16:57 TNA-download-WO-97-multilevel-xlsx.tsv
-rw-rw-r--  1 jb677  1322    16K 17 Oct 15:45 callnumbers.txt
-rw-rw-r--  1 jb677  1322    18K 17 Oct 15:45 diary.html
-rw-rw-r--  1 jb677  1322    16K 17 Oct 15:45 pdflist.txt
drwxr-xr-x  2 jb677  1322    68K Feb  2 00:58 backup
-rw-r--r--  1 jb677  1322   598K Jan 31 18:47 gulliver.txt
~~~
{: .output}

In this episode we'll focus on the three tsv files. These contain data from the UK National Archives 'Discovery' catalogue: 

- `TNA-download-FO-881-file-level-only-csv.tsv` contains metadata for records in the [Foreign Office: Confidential Print (Numerical Series)](http://discovery.nationalarchives.gov.uk/results/r?_q=%22FO+881%22&_hb=tna&_d=FO&Refine+departments=Refine)
- `TNA-download-HO-396-tagged-in-Excel-xlsx.tsv` contains metadata for records in the [Home Office: Aliens Department: Internees Index](http://discovery.nationalarchives.gov.uk/results/r?_q=%22HO+396%22&_hb=tna&_d=HO&Refine+departments=Refine)
- `TNA-download-WO-97-multilevel-xlsx.tsv` contains metadata for records in the [Royal Hospital Chelsea: Soldier Service Documents](http://discovery.nationalarchives.gov.uk/results/r?_q=%22WO+97%22&_hb=tna&_d=WO&Refine+departments=Refine).

All content is available under the [Open Government Licence v3.0](http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).

> ## CSV and TSV Files
> CSV (Comma-separated values) is a common plain text format for storing tabular
> data, where each record occupies one line and the values are separated by commas.
> TSV (Tab-separated values) is just the same except that values are separated by
> tabs rather than commas. Confusingly, CSV is sometimes used to refer to both CSV,
> TSV and variations of them. The simplicity of the formats make them great for
> exchange and archival. They are not bound to a specific program (unlike Excel
> files, say, there is no `CSV` program, just lots and lots of programs that
> support the format, including Excel by the way.), and you wouldn't have any
> problems opening a 40 year old file today if you came across one.
{: .callout}
<!-- hm, reminds me of MARC -->

`wc` is the "word count" command: it counts the number of lines, words, bytes
and characters in files. Since we love the wildcard operator, let's run the command
`wc *.tsv` to get counts for all the `.tsv` files in the current directory
(it takes a little time to complete):

~~~~
$ wc *.tsv
~~~~
{: .bash}
~~~
    6000  151981  942164 TNA-download-FO-881-file-level-only-csv.tsv
   15025  128415 1249673 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
    4000  150915 1086654 TNA-download-WO-97-multilevel-xlsx.tsv
   25025  431311 3278491 total
~~~
{: .output}

The first three columns contains the number of lines, words and bytes
(to show number characters you have to use a flag).

If we only have a handful of files to compare, it might be faster or more convenient
to just check with Microsoft Excel, OpenRefine or your favourite text editor, but
when we have tens, hundreds or thousands of documents, the Unix shell has a clear
speed advantage. The real power of the shell comes from being able to combine commands
and automate tasks, though. We will touch upon this slightly.

For now, we'll see how we can build a simple pipeline to find the shortest file
in terms of number of lines. We start by adding the `-l` flag to get only the
number of lines, not the number of words and bytes:

~~~~
$ wc -l *.tsv
~~~~
{: .bash}
~~~
    6000 TNA-download-FO-881-file-level-only-csv.tsv
   15025 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
    4000 TNA-download-WO-97-multilevel-xlsx.tsv
   25025 total
~~~
{: .output}

The `wc` command itself doesn't have a flag to sort the output, but as we'll
see, we can combine three differen shell commands to get what we want.

First, we have the `wc -l *.tsv` command. We will save the output from this
command in a new file. To do that, we *redirect* the output from the command
to a file using the ‘greater than’ sign (>), like so:

~~~
$ wc -l *.tsv > lengths.txt
~~~
{: .bash}

There's no output now since the output went into the file `lengths.txt`, but
we can check that the output indeed ended up in the file using `cat` or `less`
(or Notepad or any text editor).

~~~~
$ cat lengths.txt
~~~~
{: .bash}
~~~
    6000 TNA-download-FO-881-file-level-only-csv.tsv
   15025 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
    4000 TNA-download-WO-97-multilevel-xlsx.tsv
   25025 total
~~~
{: .bash}

Next, there is the `sort` command. We'll use the `-n` flag to specify that we
want numerical sorting, not lexical sorting, we output the results into
yet another file, and we use `cat` to check the results:

~~~~
$ sort -n lengths.txt > sorted-lengths.txt
$ cat sorted-lengths.txt
~~~~
{: .bash}
~~~
    4000 TNA-download-WO-97-multilevel-xlsx.tsv
    6000 TNA-download-FO-881-file-level-only-csv.tsv
   15025 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
   25025 total
~~~
{: .output}

Finally we have our old friend `head`, that we can use to get the first line
of the `sorted-lengths.txt`:

~~~~
$ head -n 1 sorted-lengths.txt
~~~~
{: .bash}
~~~
    4000 TNA-download-WO-97-multilevel-xlsx.tsv
~~~
{: .output}

But we're really just interested in the end result, not the intermediate
results now stored in `lengths.txt` and `sorted-lengths.txt`. What if we could
send the results from the first command (`wc -l *.tsv`) directly to the next
command (`sort -n`) and then the output from that command to `head -n 1`?
Luckily we can, using a concept called pipes. On the command line, you make a
pipe with the vertical bar character |. Let's try with one pipe first:

~~~~
$ wc -l *.tsv | sort -n
~~~~
{: .bash}
~~~
    4000 TNA-download-WO-97-multilevel-xlsx.tsv
    6000 TNA-download-FO-881-file-level-only-csv.tsv
   15025 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
   25025 total
~~~
{: .output}

Notice that this is exactly the same output that ended up in our `sorted-lengths.txt`
earlier. Let's add another pipe:

~~~~
$ wc -l *.tsv | sort -n | head -n 1
~~~~
{: .bash}
~~~
    4000 TNA-download-WO-97-multilevel-xlsx.tsv
~~~
{: .output}

It can take some time to fully grasp pipes and use them efficiently, but it's a
very powerful concept that you will find not only in the shell, but also in most
programming languages.

![Redirects and Pipes](../fig/redirects-and-pipes.png)

> ## Pipes and Filters
> This simple idea is why Unix has been so successful. Instead of creating enormous
> programs that try to do many different things, Unix programmers focus on creating
> lots of simple tools that each do one job well, and that work well with each other.
> This programming model is called “pipes and filters”. We’ve already seen pipes; a
> filter is a program like `wc` or `sort` that transforms a stream of input into a
> stream of output. Almost all of the standard Unix tools can work this way: unless
> told to do otherwise, they read from standard input, do something with what they’ve
> read, and write to standard output.
>
> The key is that any program that reads lines of text from standard input and writes
> lines of text to standard output can be combined with every other program that
> behaves this way as well. You can and should write your programs this way so that
> you and other people can put those programs into pipes to multiply their power.
{: .callout}
<!-- Copied from https://swcarpentry.github.io/shell-novice/04-pipefilter/ -->

> ## Adding another pipe
> We have our `wc -l *.tsv | sort -n | head -n 1` pipeline. What would happen
> if you piped this into `cat`? Try it!
>
> > ## Solution
> > The `cat` command just outputs whatever it gets as input, so you get exactly
> > the same output from
> >
> > ~~~
> > $ wc -l *.tsv | sort -n | head -n 1
> > ~~~
> > {: .bash}
> >
> > and
> >
> > ~~~
> > $ wc -l *.tsv | sort -n | head -n 1 | cat
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Count, sort and print (faded example)
>To count the total lines in every `tsv` file, sorting the results and then print the first line of the file we use the following:
>
>~~~
>wc -l *.tsv | sort | head -n 1
>~~~
>{: .bash}
>
>
>Now let's change the scenario. Imagine that we have a directory containing 100 `csv` files. We want to know the 10 files that contain the most words. Fill in the blanks below to count the words for each file, put them into order, and then make an output of the 10 files with the most words (Hint: The sort command sorts in ascending order by default).
>
>~~~
>__ -w *.csv | sort | ____
>~~~
>{: .bash}
>
> > ## Solution
> >
> > Here we use the `wc` command with the `-w` (word) flag on all `csv` files, `sort` them and then output the last 10 lines using the `tail` command.
> >~~~
> > wc -w *.csv | sort | tail -n 10
> >~~~
> {: .solution}
>{: .bash}
{: .challenge}


> ## Counting number of files, part I
> Let's make a different pipeline. You want to find out how many files and
> directories there are in the current directory. Try to see if you can pipe
> the output from `ls` into `wc` to find the answer, or something close to the
> answer.
>
> > ## Solution
> > You get close with
> >
> > ~~~
> > $ ls -l | wc -l
> > ~~~~
> > {: .bash}
> >
> > but the count will be one too high, since the "total" line from `ls`
> > is included in the count. We'll get back to a way to fix that later
> > when we've learned about the `grep` command.
> {: .solution}
{: .challenge}

> ## Writing to files
> The `date` command outputs the current date and time. Can you write the
> current date and time to a new file called `logfile.txt`? Then check
> the contents of the file.
>
> > ## Solution
> > ~~~
> > $ date > logfile.txt
> > $ cat logfile.txt
> > ~~~~
> > {: .bash}
> > To check the contents, you could also use `less` or many other commands.
> >
> > Beware that `>` will happily overwrite an existing file without warning you,
> > so please be careful.
> {: .solution}
{: .challenge}

> ## Appending to a file
> While `>` writes to a file, `>>` appends something to a file. Try to append the
> current date and time to the file `logfile.txt`?
>
> > ## Solution
> > ~~~
> > $ date >> logfile.txt
> > $ cat logfile.txt
> > ~~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Counting the number of words
>
> Check the manual for the `wc` command (either using `man wc` or `wc --help`)
> to see if you can find out what flag to use to print out the number of words
> (but not the number of lines and bytes). Try it with the `.tsv` files.
>
> If you have time, you can also try to sort the results by piping it to `sort`.
> And/or explore the other flags of `wc`.
>
> > ## Solution
> >
> > From `man wc`, you will see that there is a `-w` flag to print the number of
> > words:
> >
> > ~~~
> >      -w      The number of words in each input file is written to the standard
> >              output.
> > ~~~
> > {: .output}
> >
> > So to print the word counts of the `.tsv` files:
> >
> > ~~~
> > $ wc -w *.tsv
> > ~~~
> > {: .bash}
> > ~~~
> >  151981 TNA-download-FO-881-file-level-only-csv.tsv
> >  128415 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
> >  150915 TNA-download-WO-97-multilevel-xlsx.tsv
> >  431311 total
> > ~~~
> > {: .output}
> >
> > And to sort the lines numerically:
> >
> > ~~~
> > $ wc -w *.tsv | sort -n
> > ~~~
> > {: .bash}
> > ~~~
> >  128415 TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
> >  150915 TNA-download-WO-97-multilevel-xlsx.tsv
> >  151981 TNA-download-FO-881-file-level-only-csv.tsv
> >  431311 total
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Mining or searching

Searching for something in one or more files is something we'll often need to do,
so let's introduce a command for doing that: `grep` (short for **global regular
expression print**). As the name suggests, it supports regular expressions and
is therefore only limited by your imagination, the shape of your data, and - when
working with thousands or millions of files - the processing power at your disposal.

To begin using `grep`, first navigate to the `shell-lesson` directory if not already
there. Then create a new directory "results":

~~~
$ mkdir results
~~~
{: .bash}


Now let's try our first search:

~~~
$ grep 1867 *.tsv 
~~~
{: .bash}

Remember that the shell will expand `*.tsv` to a list of all the .tsv files in the
directory. `grep` will then search these for instances of the string "1867" and
print the matching lines.

> ## Strings
> A string is a sequence of characters, or "a piece of text".
{: .callout}

Press the up arrow once in order to cycle back to your most recent action.
Amend `grep 1867 *.tsv` to `grep -c 1867 *.tsv` and hit enter.

~~~
$ grep -c 1867 *.tsv
~~~
{: .bash}
~~~
TNA-download-FO-881-file-level-only-csv.tsv:120
TNA-download-HO-396-tagged-in-Excel-xlsx.tsv:23
TNA-download-WO-97-multilevel-xlsx.tsv:0
~~~
{: .output}

The shell now prints the number of times the string 1867 appeared in each file.
If you look at the output from the previous command, this tends to refer to the
date field for each journal article.

We will try another search:

~~~
$ grep -c German *.tsv
~~~
{: .bash}
~~~
TNA-download-FO-881-file-level-only-csv.tsv:156
TNA-download-HO-396-tagged-in-Excel-xlsx.tsv:340
TNA-download-WO-97-multilevel-xlsx.tsv:14
~~~
{: .output}

We got back the counts of the instances of the string `German` within the files.
Now, amend the above command to the below and observe how the output of each is different:

~~~
$ grep -ci German *.tsv
~~~
{: .bash}
~~~
TNA-download-FO-881-file-level-only-csv.tsv:194
TNA-download-HO-396-tagged-in-Excel-xlsx.tsv:340
TNA-download-WO-97-multilevel-xlsx.tsv:15
~~~
{: .output}

**goes up. But why?. Do `grep -c german *.tsv` no hits. What is going on? (ALL CAPS!!)

This repeats the query, but prints a case
insensitive count (including instances of both `german` and `German` and other variants).
Note how the count has increased only modestly and in some cases not at all.

Let's dig a little more into what is going on. Try:

~~~
$ grep -c german *.tsv
~~~
{: .bash}
~~~
TNA-download-FO-881-file-level-only-csv.tsv:0
TNA-download-HO-396-tagged-in-Excel-xlsx.tsv:0
TNA-download-WO-97-multilevel-xlsx.tsv:0
~~~
{: .output}

We now see the string `german` appears nowhere in our data. What then accounts for the difference between `grep -c German *.tsv` and `grep -ci German *.tsv`? (hint: consider what other variants of `german` and `German` are found by `grep -ci German *.tsv`)

Note if you wanted to any of these outputs, cycling back and
adding `> results/`, followed by a filename (ideally in .txt format), will save the results to a data file.

So far we have counted strings in file and printed to the shell or to
file those counts. But the real power of `grep` comes in that you can
also use it to create subsets of tabulated data (or indeed any data)
from one or multiple files.  

~~~
$ grep -i german *.tsv
~~~
{: .bash}

This script looks in the defined files and prints any lines containing `german`
(without regard to case) to the shell.

~~~
$ grep -i german *.tsv > results/TNAdata-german-i.tsv
~~~
{: .bash}

This saves the subsetted data to file.

However, if we look at this file, it contains every instance of the
string 'german' including as a single word and as part of other words
such as 'Germany'. This perhaps isn't as useful as we thought...
Thankfully, the `-w` flag instructs `grep` to look for whole words only,
giving us greater precision in our search.

~~~
$ grep -iw german *.tsv > results/TNAdata-german-iw.tsv
~~~
{: .bash}

This script looks in both of the defined files and
exports any lines containing the whole word `german` (without regard to case)
to the specified .tsv file.

We can show the difference between the files we created.

~~~
$ wc -l results/*.tsv
~~~
{: .bash}
~~~
     549 results/TNAdata-german-i.tsv
      75 results/TNAdata-german-iw.tsv
     624 total
~~~
{: .output}

What can account for the difference?

Finally, we'll use the **regular expression syntax** covered earlier to search for similar words.

> ## Basic and extended regular expressions
> There is unfortunately both ["basic" and "extended" regular expressions](https://www.gnu.org/software/grep/manual/html_node/Basic-vs-Extended.html).
> This is a common cause of confusion, since most tutorials, including ours, teach
> extended regular expression, but `grep` uses basic by default.
> Unles you want to remember the details, make your life easy by always using
> extended regular expressions (`-E` flag) when doing something more complex
> than searching for a plain string.
{: .callout}

The regular expression 'fr[ae]nc[eh]' will match "france", "french", but also "frence" and "franch".
It's generally a good idea to enclose the expression in single quotation marks, since
that ensures the shell sends it directly to grep without any processing (such as trying to
expand the wildcard operator *).

~~~
$ grep -iwE 'fr[ae]nc[eh]' *.tsv
~~~
{: .bash}

The shell will print out each matching line.

We include the `-o` flag to print only the matching part of the lines e.g.
(handy for isolating/checking results):

~~~
$ grep -iwEo 'fr[ae]nc[eh]' *.tsv
~~~
{: .bash}

> ## Invalid option -- o?
> If you get an error message "invalid option -- o" when running the above command, it means you use a version of
> `grep` that doesn't support the `-o` flag. This is for instance the case with the version of `grep` that comes with
> Git Bash on Windows. Since the flag is not crucial to this lesson, please just relax and ignore the problem. If you
> really needed the flag, however, you could have installed another version of `grep`. The situation for Windows users
> also improves on Windows 10 with the new Bash on Windows.
{: .callout}

Pair up with your neighbor and work on these exercies:

> ## Case sensitive search
> Search for all case sensitive instances of
> a word you choose in all three tsv files in this directory.
> Print your results to the shell.
>
> > ## Solution
> > ~~~
> > $ grep hero *.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Case sensitive search in select files
> Search for all case sensitive instances of a word you choose in
> the 'FO 881' and 'HO 396' tsv files in this directory.
> Print your results to the shell.
>
> > ## Solution
> > ~~~
> > $ grep hero TNA-download-FO-881-file-level-only-csv.tsv TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Count words (case sensitive)
> Count all case sensitive instances of a word you choose in
> the 'FO 881' and 'HO 396' tsv files in this directory.
> Print your results to the shell.
>
> > ## Solution
> > ~~~
> > $ grep -c hero TNA-download-FO-881-file-level-only-csv.tsv TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Count words (case insensitive)
> Count all case insensitive instances of that word in the 'FO 881' and 'HO 396' tsv files
> in this directory. Print your results to the shell.
>
> > ## Solution
> > ~~~
> > $ grep -ci hero TNA-download-FO-881-file-level-only-csv.tsv TNA-download-HO-396-tagged-in-Excel-xlsx.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Case insensitive search in select files
> Search for all case insensitive instances of that
> word in the 'FO 881' and 'HO 396' tsv files in this directory. Print your results to a file `results/new.tsv`.
>
> > ## Solution
> > ~~~
> > $ grep -i hero TNA-download-FO-881-file-level-only-csv.tsv TNA-download-HO-396-tagged-in-Excel-xlsx.tsv > results/new.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Case insensitive search in select files (whole word)
> Search for all case insensitive instances of that whole word
> in the 'FO 881' and 'HO 396' tsv files in this directory. Print your results to a file `results/new2.tsv`.
>
> > ## Solution
> > ~~~
> > $ grep -iw hero TNA-download-FO-881-file-level-only-csv.tsv TNA-download-HO-396-tagged-in-Excel-xlsx.tsv > results/new2.tsv
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Finding unique values
> If you pipe something to the `uniq` command, it will filter out duplicate lines
> and only return unique ones. The [Royal Hospital Chelsea: Soldier Service Documents](http://discovery.nationalarchives.gov.uk/results/r?_q=%22WO+97%22&_hb=tna&_d=WO&Refine+departments=Refine) contains a number of date ranges (eg '1760-1854' or '1815-1833'). Open the solution tab to see how many uniq date ranges the data contains. Can you pull this code apart and figure out what each step is doing?
> Note: This exercise requires the `-o` flag. See the callout box "Invalid option -- o?"
> above.
>
> > ## Solution
> > ~~~
> > $ grep -Eo '\d{4}-\d{4}' *.tsv | uniq | wc -l
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}

> ## Counting number of files, part II
> In the earlier counting exercise in this episode, you tried counting the number
> of files and directories in the current directory.
>
> * Recall that the command `ls -l | wc -l` took us quite far, but the result was one
>   too high because it included the "total" line in the line count.
> * With the knowledge of `grep`, can you figure out how to exclude the "total"
>   line from the `ls -l` output?
> * Hint: You want to exclude any line *starting*
>   with the text "total". The hat character (^) is used
>   in regular expressions to indicate the start of a line.
>
> > ## Solution
> > To find any lines starting with "total", we would use:
> >
> > ~~~
> > $ ls -l | grep -E '^total'
> > ~~~
> > {: .bash}
> >
> > To *exclude those lines, we add the `-v` flag:
> >
> > ~~~
> > $ ls -l | grep -v -E '^total'
> > ~~~
> > {: .bash}
> >
> > The grand finale is to pipe this into `wc -l`:
> >
> > ~~~
> > $ ls -l | grep -v -E '^total' | wc -l
> > ~~~
> > {: .bash}
> {: .solution}
{: .challenge}