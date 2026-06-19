## Command-line Environment

### Arguments and Globs

You might see commands like cmd --flag -- --notaflag. The -- is a special argument that tells the program to stop parsing flags. Everything after -- is treated as a positional argument. Why might this be useful? Try running touch -- -myfile and then removing it without --.

- It's useful for those file names that may contain special symbols such as ```-```. See the example:
```
matteo@PC-di-Matteo:~/missing_semester/cmd_line$ touch -- -myfile
matteo@PC-di-Matteo:~/missing_semester/cmd_line$ ls
-myfile
matteo@PC-di-Matteo:~/missing_semester/cmd_line$ rm -myfile
rm: invalid option -- 'm'
Try 'rm ./-myfile' to remove the file '-myfile'.
Try 'rm --help' for more information.
matteo@PC-di-Matteo:~/missing_semester/cmd_line$
```

Correct way to remove the created file:
```
matteo@PC-di-Matteo:~/missing_semester/cmd_line$ rm -- -myfile
```


Read man ls and write an ls command that lists files in the following manner:
- Includes all files, including hidden files.
- Sizes are listed in human readable format (e.g. 454M instead of 454279954)
- Files are ordered by recency
- Output is colorized

A sample output would look like this:
```
-rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
drwxr-xr-x   5 user group  160 Jan 14 09:53 .
-rw-r--r--   1 user group  514 Jan 14 06:42 bar
-rw-r--r--   1 user group 106M Jan 13 12:12 foo
drwx------+ 47 user group 1.5K Jan 12 18:08 ..
```
- a, list all
- lt, sort by the latest modifidied file
- h, human readable format

```
ls -a -lt -h --color 
```


Process substitution <(command) lets you use a command’s output as if it were a file. Use diff with process substitution to compare the output of printenv and export. Why are they different? (Hint: try diff <(printenv | sort) <(export | sort)).

- printenv provides a clean output of the environment variables, while export is a shell builtin command to export the environment variables using declare -x, another builtin command to declare variables and give them attributes, with the -x flag used to make NAMEs export. This means even child processes will be able to use those variables.


### Environment Variables

Write bash functions marco and polo that do the following: whenever you execute marco the current working directory should be saved in some manner, then when you execute polo, no matter what directory you are in, polo should cd you back to the directory where you executed marco. For ease of debugging you can write the code in a file marco.sh and (re)load the definitions to your shell by executing source marco.sh.

```
#!/usr/bin/env bash

marco () {
    # Save the path of the current directory
    saved_dir=$(pwd)

    echo 'marco!'
}

polo () {
    cd "$saved_dir"
    echo 'polo!'
}
```


### Return Codes

Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.

```
#!/usr/bin/env bash

n=$(( RANDOM % 100 ))

if [[ n -eq 42 ]]; then
    echo "Something went wrong"
    >&2 echo "The error was using magic numbers"
    exit 1
fi

echo "Everything went according to plan"
```

```
#!/usr/bin/env bash

n=0
while ./test_script.sh 1>stdout.txt 2>stderr.txt
do
    n=$((n + 1))
done

cat stdout.txt
cat stderr.txt
echo "Number of tries: $n"
```


### Signals and Job Control

Start a sleep 10000 job in a terminal, background it with Ctrl-Z and continue its execution with bg. Now use pgrep to find its pid and pkill to kill it without ever typing the pid itself. (Hint: use the -af flags).

```
matteo@PC-di-Matteo:~$ sleep 10000
^Z
[1]+  Stopped                 sleep 10000
matteo@PC-di-Matteo:~$ bg
[1]+ sleep 10000 &
matteo@PC-di-Matteo:~$ pgrep -a sleep
791 sleep 10000
matteo@PC-di-Matteo:~$ pkill -f "sleep 10000"
[1]+  Terminated              sleep 10000
```


Say you don’t want to start a process until another completes. How would you go about it? In this exercise, our limiting process will always be sleep 60 &. One way to achieve this is to use the wait command. Try launching the sleep command and having an ls wait until the background process finishes.

```
matteo@PC-di-Matteo:~$ sleep 60 &
[1] 1627
matteo@PC-di-Matteo:~$ wait $!; ls

-> after 60 seconds
[1]+  Done                    sleep 60
missing_semester
```

However, this strategy will fail if we start in a different bash session, since wait only works for child processes. One feature we did not discuss in the notes is that the kill command’s exit status will be zero on success and nonzero otherwise. kill -0 does not send a signal but will give a nonzero exit status if the process does not exist. Write a bash function called pidwait that takes a pid and waits until the given process completes. You should use sleep to avoid wasting CPU unnecessarily.

```
#!/usr/bin/env bash

pidwait () {
    n=0
    while kill -0 "$1"
    do
        echo $n
        ((n=n+1))
        sleep 1
    done
}
```

```
In the terminal: pidwait pid
→ pid is "$1" = arg1
```


### Files and Permissions

(Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?

→ Most recently modified file
```
#!/usr/bin/env bash

find "$(pwd)" -type f -printf "%C@\t%CF %CH:%CM\t%p\n" | sort -rn | head -1 | awk -F'\t' '{print $2, $3, $4}'
```

→ All files
```
#!/usr/bin/env bash

find "$(pwd)" -type f -printf "%C@\t%CF %CH:%CM\t%p\n" | sort -rn | awk -F'\t' '{print $2, $3, $4}' 
```


### Aliases and Dotfiles

Create an alias dc that resolves to cd for when you type it wrong.
```
alias dc="cd"
```


Create a folder for your dotfiles and set up version control.
- Local version control setup using git:
```
matteo@PC-di-Matteo:~/dotfiles$ git init
Initialized empty Git repository in /home/matteo/dotfiles/.git/
matteo@PC-di-Matteo:~/dotfiles$ git add .
matteo@PC-di-Matteo:~/dotfiles$ git commit -m "My dotfiles"
[master (root-commit) 485b3d1] My dotfiles
 3 files changed, 68 insertions(+)
 create mode 100644 .bash_aliases
 create mode 100644 .tmux.conf
 create mode 100644 .vimrc
```


Add a configuration for at least one program, e.g. your shell, with some customization (to start off, it can be something as simple as customizing your shell prompt by setting $PS1).

- .vimrc configuration file:
```
" Add syntax check
filetype on
filetype plugin indent on
syntax on

" Change the color scheme
colorscheme desert

" Add numbers to each line on the left-hand side
set number
```


Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls ln -s for each file, or you could use a specialized utility.
- I used the specialized unit chezmoi, which allows for easy management of personal configuration files across multiple machines.


### Remote Machines (SSH)

Go to ~/.ssh/ and check if you have a pair of SSH keys there. If not, generate them with ssh-keygen -a 100 -t ed25519. It is recommended that you use a password and use ssh-agent, more info here.

Edit .ssh/config to have an entry as follows:
```
Host vm
    User username_goes_here
    HostName ip_goes_here
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888
```

Use ssh-copy-id vm to copy your ssh key to the server.

Start a webserver in your VM by executing python -m http.server 8888. Access the VM webserver by navigating to http://localhost:9999 in your machine.
- We can connect using ssh and the -L flag: 
```
ssh -L 9999:localhost:8888 matteo@192.168.1.57
```
- Then navigate to http://localhost:9999 using any browser.

Edit your SSH server config by doing sudo vim /etc/ssh/sshd_config and disable password authentication by editing the value of PasswordAuthentication. Disable root login by editing the value of PermitRootLogin. Restart the ssh service with sudo service sshd restart. Try sshing in again.
- It works because the local machine is now a trusted host and it identifies itself using the SSH keys. We are not able to access as root: ```ssh root@192.168.1.57``` will give a permission denied message.


(Challenge) Install mosh in the VM and establish a connection. Then disconnect the network adapter of the server/VM. Can mosh properly recover from it?
- It can properly recover from it because mosh is designed to handle intermittent connectivity and disconnections.
```
VirtualBox → Network → Connected to: not attached

mosh: Last contact 22 seconds ago. [To quit: Ctrl-^ .]

→ Reconnecting the network adapter re-establishes the connection.
```


(Challenge) Look into what the -N and -f flags do in ssh and figure out a command to achieve background port forwarding.
```
ssh -L 9999:localhost:8888 -f -N matteo@192.168.1.57
```
