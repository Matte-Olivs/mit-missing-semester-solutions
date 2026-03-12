What does the -l flag to ls do? Run ls -l / and examine the output. What do the first 10 characters of each line mean? (Hint: man ls):
- The -l flag returns the name of each file in the current directory along with informations about them: permissions, user who created the file, date and hour of creation.


In the command find ~/Downloads -type f -name "*.zip" -mtime +30, the *.zip is a “glob”. What is a glob? Create a test directory with some files and experiment with patterns like ls *.txt, ls file?.txt, and ls {a,b,c}.txt. See Pattern Matching in the Bash manual.  
- A “glob” is a way for the interpreter to autocomplete the file path and find each file ending the specified file extension, such as *.zip or *.txt.


```What’s the difference between 'single quotes', "double quotes", and $'ANSI quotes'? Write a command that echoes a string containing a literal $, a !, and a newline character. See Quoting.```  
- Enclosing characters in single quotes (‘'’) preserves the literal value of each character within the quotes.
- Enclosing characters in double quotes (‘"’) preserves the literal value of all characters within the quotes, with the exception of ‘$’, ‘`’, ‘\’, and, when history expansion is enabled, ‘!’.
- Character sequences of the form $'string' are treated as a special kind of single quotes. The sequence expands to string, with backslash-escaped characters in string replaced as specified by the ANSI C standard.

- Some testing:
```
matteo@PC-di-Matteo:~$ ls
cartella_prova
matteo@PC-di-Matteo:~$ echo "Ciao mondo!!"
echo "Ciao mondols"
Ciao mondols
matteo@PC-di-Matteo:~$ echo "Ciao mondo!!"
echo "Ciao mondoecho "Ciao mondols""
Ciao mondoecho Ciao mondols
matteo@PC-di-Matteo:~$ echo 'Ciao mondo!!'
Ciao mondo!!
matteo@PC-di-Matteo:~$ echo $'Ciao mondo!!'
Ciao mondo!!
matteo@PC-di-Matteo:~$ echo "Ciao \\ mondo"
Ciao \ mondo
matteo@PC-di-Matteo:~$ echo "Ciao \$ mondo"
Ciao $ mondo
```


The shell has three standard streams: stdin (0), stdout (1), and stderr (2). Run ls /nonexistent /tmp and redirect stdout to one file and stderr to another. How would you redirect both to the same file? See Redirections. 
- Separate files:  ls /nonexistent /tmp 1>file1.txt 2>file2.txt
- Both in the same file:  ls /nonexistent /tmp &>samefile.txt


$? holds the exit status of the last command (0 = success). && runs the next command only if the previous succeeded; || runs it only if the previous failed. Write a one-liner that creates /tmp/mydir only if it doesn’t already exist. See Exit Status.

```
#!/bin/bash
[ -d /tmp/mydir ] || mkdir /tmp/mydir
```


Why does cd have to be built into the shell itself rather than a standalone program? (Hint: think about what a child process can and cannot affect in its parent.)
A child process operates in its own space in memory and can’t change the parent’s internal state such as variables, working directory and shell options.
- If cd was a standalone program, it would create a child process and only change the directory in that process.


Write a script that takes a filename as an argument ($1) and checks whether the file exists using test -f or [ -f ... ]. It should print different messages depending on whether the file exists. See Bash Conditional Expressions. 

```
#!/bin/bash
if [ -f “$1” ]; then echo "File found"; else echo "File not found";fi
```


Save the script from the previous exercise to a file (e.g., check.sh). Try running it with ./check.sh somefile. What happens? Now run chmod +x check.sh and try again. Why is this step necessary? (Hint: look at ls -l check.sh before and after the chmod.) 
- Without using chmod +x, we get “Permission denied”. Using chmod +x we give the user permission to execute the selected file. Before, the user only had rw(read and write) permissions.


What happens if you add -x to the set flags in a script? Try it with a simple script and observe the output. See The Set Builtin.
- It prints out to the terminal the lines that get executed.


Write a command that copies a file to a backup with today’s date in the filename (e.g., notes.txt → notes_2026-01-12.txt). (Hint: $(date +%Y-%m-%d)). See Command Substitution.

```
#!/bin/bash

if [ -f "$1.$2" ];
then 
 cp "$1.$2" "$1_$(date +%Y-%m-%d).$2"
else 
 echo "File not found"
fi
```


Modify the flaky test script from the lecture to accept the test command as an argument instead of hardcoding cargo test my_test. (Hint: $1 or $@). See Special Parameters.

```
#!/bin/bash
set -euo pipefail

# Start CPU stress in background
stress --cpu 8 &
STRESS_PID=$!

# Setup log file
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# Run tests until one fails
RUN=1
while cargo test “$@” > "$LOGFILE" 2>&1; do
    echo "Run $RUN passed"
    ((RUN++))
done

# Cleanup and report
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE"
```


Use pipes to find the 5 most common file extensions in your home directory. (Hint: combine find, grep or sed or awk, sort, uniq -c, and head.) 

```
find ~ -type f -name "*.*" → find each file in the home directory ~ 
awk -F. '{print $NF}'  → separate in columns using “.” (file.txt has file as the first column and txt as the second column). $NF takes the last “.” of the file to ensure it only extracts the file extensions.
find ~ -type f -name "*.*" | awk -F. '{print $NF}' | sort | uniq -c | sort -rn | head -5
```


xargs converts lines from stdin into command arguments. Use find and xargs together (not find -exec) to find all .sh files in a directory and count the lines in each with wc -l. Bonus: make it handle filenames with spaces. (Hint: -print0 and -0). See man xargs.

- find . -name "*.sh" -type f -print0 | xargs -0 wc -l


Use curl to fetch the HTML of the course website (https://missing.csail.mit.edu/) and pipe it to grep to count how many lectures are listed. (Hint: look for a pattern that appears once per lecture; use curl -s to silence the progress output.) 

- curl -s https://missing.csail.mit.edu/ | grep -c '<a href="/2026/.*">'


jq is a powerful tool for processing JSON data. Fetch the sample data at https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json with curl and use jq to extract just the names of people whose version is greater than 6. (Hint: pipe to jq . first to see the structure; then try jq '.[] | select(...) | .name') 

- curl -s https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json | jq '.[] | select(.version >= 6.0) | .name'


awk can filter lines based on column values and manipulate output. For example, awk '$3 ~ /pattern/ {$4=""; print}' prints only lines where the third column matches pattern, while omitting the fourth column. Write an awk command that prints only lines where the second column is greater than 100, and swaps the first and third columns. Test with: printf 'a 50 x\nb 150 y\nc 200 z\n' 

- printf 'a 50 x\nb 150 y\nc 200 z\n' | awk '$2 > 100 {print $3, $2, $1}'


- Dissect the SSH log pipeline from the lecture: what does each step do? Then build something similar to find your most-used shell commands from ~/.bash_history (or ~/.zsh_history).

```
ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

- ssh myserver tries to connect to “myserver” using secure shell
- journalctl is used to print log entries from the systemd journal, with:
	-u showing messages for the specified systemd unit UNIT
 	-b showing messages from a specific boot(1 here)
- grep prints the lines that match the pattern “Disconnected from”
- sed formats the given lines, with:
	-E specifying to use extended regular expressions in the script
	/s attempting to match regexp against the pattern space 	special
	\1 refers to the corresponding matching sub-	expression
- sort sorts lines by name
- uniq -c prefixes lines by the number of occurences
-  sort -nk1,1
	-n numeric sort, using the first column
	-k sort via key, using the first column
- tail n10 prints the last 10 lines of each FILE to standard output.
- awk '{print $2}' prints the second column of each line
- paste -sd merges the given lines, with:
	-s  paste one line at a time instead of in parallel
	-d  reuse characters from LIST instead of TABs, using the following list: 	postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root

- To find the most used commands from ~/.bash_history:
```
awk -F '[|; ]+' '{print $1}' ~/.bash_history | sort | uniq -c | sort -rn | head
to include the recent history:
history | awk -F '[|; ]+' '{print $3}' | sort | uniq -c | sort -rn | head
→ not every command present in pipes will be included, just the first one (for ex,
awk … | sort will only count awk)
```
