---
title: Common Linux Commands
author: hcoco1
date: 2024-02-03 01:10:00 -0500
categories: [Programming, Linux]
tags: [bash, cli, linux]
toc: true
description: A practical reference of the most common Linux commands — file management, viewing, system info, networking, packages, permissions, and archiving — with examples and safety notes.
---

The Linux command line is a text interface to your computer. Often called the shell, terminal, console, or prompt, it can appear complex at first. But being able to copy and paste commands, combined with the power and flexibility the command line offers, makes it essential when following instructions online — including many on this site.

> New to the shell? Prefix any command below with `man` (for example, `man ls`) to read its full manual, and press `q` to exit.
{: .prompt-tip }

## File and Directory Management

- `ls` - List directory contents

  ```bash
  ls
  ls -l
  ls -lh
  ls -la
  ```

- `cd` - Change directory

  ```bash
  cd /path/to/directory
  cd ..
  cd ~
  cd -
  ```

- `pwd` - Print working directory

  ```bash
  pwd
  ```

- `mkdir` - Create a new directory

  ```bash
  mkdir directory_name
  mkdir -p /path/to/nested/directory
  ```

- `rmdir` - Remove an empty directory

  ```bash
  rmdir directory_name
  ```

- `rm` - Remove files or directories

  ```bash
  rm file_name
  rm -i file_name
  rm -r directory_name
  rm -rf directory_name
  ```

- `cp` - Copy files or directories

  ```bash
  cp source_file destination_file
  cp -i source_file destination_file
  cp -r source_directory destination_directory
  ```

- `mv` - Move or rename files or directories

  ```bash
  mv source_file destination_file
  mv -i source_file destination_file
  mv old_directory_name new_directory_name
  ```

- `touch` - Create an empty file or update its timestamp

  ```bash
  touch file_name
  ```

- `ln` - Create links between files

  ```bash
  ln -s /path/to/target link_name
  ```

- `find` - Search for files in a directory hierarchy

  ```bash
  find /path/to/search -name "file_name"
  find /path -type f -name "*.log"
  find /path -type d -name "cache"
  find . -name "*.tmp" -delete
  ```

- `tree` - Display directories as a tree

  ```bash
  tree
  tree -L 2
  tree -a
  ```

> There is no recycle bin on the command line. `rm -rf` deletes permanently and without confirmation. Double-check the path before running it, and never run `rm -rf` with a wildcard or a variable you haven't verified (`rm -rf $DIR/` becomes `rm -rf /` if `$DIR` is empty).
{: .prompt-danger }

> `cp` and `mv` overwrite the destination silently. Add `-i` to be prompted before replacing an existing file. The same applies to `find ... -delete` — test the match with `find ... -print` first.
{: .prompt-warning }

## File Viewing and Editing

- `cat` - Concatenate and display file content

  ```bash
  cat file_name
  cat file1 file2 > combined.txt
  ```

- `less` - View file content one screen at a time

  ```bash
  less file_name
  ```

- `more` - View file content one screen at a time

  ```bash
  more file_name
  ```

- `head` - Display the first lines of a file

  ```bash
  head file_name
  head -n 20 file_name
  ```

- `tail` - Display the last lines of a file

  ```bash
  tail file_name
  tail -n 20 file_name
  tail -f /var/log/syslog
  ```

- `grep` - Search text using patterns

  ```bash
  grep "pattern" file_name
  grep -i "pattern" file_name
  grep -n "pattern" file_name
  grep -r "pattern" /path/to/search
  ```

- `wc` - Count lines, words, and bytes

  ```bash
  wc file_name
  wc -l file_name
  ```

- `nano` - Simple text editor in the terminal

  ```bash
  nano file_name
  ```

- `vi` or `vim` - Advanced text editor in the terminal

  ```bash
  vi file_name
  vim file_name
  ```

> `tail -f` follows a file as it grows, which is the quickest way to watch a log in real time. Press `Ctrl+C` to stop.
{: .prompt-tip }

> Stuck in `vim`? Press `Esc`, then type `:wq` to save and quit or `:q!` to quit without saving. If `cat` on a binary file scrambles your terminal, run `reset` to restore it.
{: .prompt-tip }

## System Information and Management

- `top` - Display real-time system information

  ```bash
  top
  ```

- `ps` - Display information about active processes

  ```bash
  ps
  ps aux
  ps aux | grep process_name
  ```

- `df` - Display disk space usage

  ```bash
  df
  df -h
  ```

- `du` - Estimate file space usage

  ```bash
  du
  du -h
  du -sh directory_name
  ```

- `free` - Display memory usage

  ```bash
  free
  free -h
  ```

- `uptime` - Show how long the system has been running

  ```bash
  uptime
  ```

- `uname` - Display system information

  ```bash
  uname
  uname -a
  ```

- `systemctl` - Manage services on systemd distributions

  ```bash
  systemctl status service_name
  systemctl start service_name
  systemctl stop service_name
  systemctl restart service_name
  systemctl enable service_name
  ```

> Add `-h` to `df`, `du`, and `free` for human-readable sizes (KB/MB/GB instead of raw blocks). In `top`, press `q` to quit; `htop` is a friendlier alternative if it's installed.
{: .prompt-tip }

> `systemctl start`, `stop`, `restart`, and `enable` change system state and require `sudo`. `status` is read-only and safe to run as a normal user.
{: .prompt-info }

## Networking

- `ping` - Send ICMP ECHO_REQUEST to network hosts

  ```bash
  ping host_name_or_ip
  ping -c 4 host_name_or_ip
  ```

- `ifconfig` - Configure a network interface

  ```bash
  ifconfig
  ```

- `ip` - Show/manipulate routing, devices, and tunnels

  ```bash
  ip address
  ip a
  ip link
  ip route
  ```

- `netstat` - Network statistics

  ```bash
  netstat
  netstat -tuln
  ```

- `ss` - Investigate sockets

  ```bash
  ss
  ss -tuln
  ```

- `scp` - Securely copy files between hosts

  ```bash
  scp source_file user@remote_host:/path/to/destination
  scp -r source_directory user@remote_host:/path/to/destination
  ```

- `wget` - Non-interactive network downloader

  ```bash
  wget url
  wget -O output_name url
  ```

- `curl` - Transfer data from or to a server

  ```bash
  curl url
  curl -O url
  curl -L url
  ```

- `lsof` - List open files, useful to check ports

  ```bash
  lsof -i :port_number
  ```

- `kill` - Terminate a process by PID

  ```bash
  kill pid
  kill -9 pid
  ```

> `ifconfig` and `netstat` are deprecated on modern distributions and may not be installed. Use `ip` in place of `ifconfig` and `ss` in place of `netstat`.
{: .prompt-warning }

> `kill -9` sends `SIGKILL`, which the process cannot catch or clean up after — it can leave temp files or corrupt state behind. Try a plain `kill` (`SIGTERM`) first and only escalate to `-9` if the process ignores it.
{: .prompt-warning }

## Package Management

- `apt-get` - APT package handling utility (Debian/Ubuntu)

  ```bash
  sudo apt-get update
  sudo apt-get install package_name
  sudo apt-get remove package_name
  ```

- `yum` - Package manager (Red Hat/CentOS)

  ```bash
  sudo yum update
  sudo yum install package_name
  sudo yum remove package_name
  ```

- `dnf` - Package manager (Fedora)

  ```bash
  sudo dnf update
  sudo dnf install package_name
  sudo dnf remove package_name
  ```

> Run the `update` command before installing so you pull the latest package versions. On recent Debian/Ubuntu systems, `apt` is the modern front end and can replace `apt-get` in these examples.
{: .prompt-tip }

## Permissions and Ownership

- `sudo` - Run a command as the superuser

  ```bash
  sudo command
  ```

- `chmod` - Change file modes or Access Control Lists

  ```bash
  chmod 755 file_name
  chmod +x script.sh
  chmod -R 755 directory_name
  ```

- `chown` - Change file owner and group

  ```bash
  chown user_name:group_name file_name
  chown -R user_name:group_name directory_name
  ```

> Avoid `chmod 777` — it makes a file writable and executable by everyone, which is a common security hole. Grant the least access that works (often `644` for files, `755` for directories and scripts).
{: .prompt-danger }

> Recursive changes (`-R`) apply to every file underneath the target. Running `chmod -R` or `chown -R` on the wrong directory — for example a system path like `/` or `/etc` — can render the system unbootable. Confirm the path first.
{: .prompt-warning }

## Archiving and Compression

- `tar` - Store, list, or extract files in an archive

  ```bash
  tar -cvf archive_name.tar directory_name
  tar -xvf archive_name.tar
  tar -czvf archive_name.tar.gz directory_name
  tar -xzvf archive_name.tar.gz
  ```

- `zip` - Package and compress files

  ```bash
  zip archive_name.zip file1 file2
  zip -r archive_name.zip directory_name
  ```

- `unzip` - Extract files from a ZIP archive

  ```bash
  unzip archive_name.zip
  unzip archive_name.zip -d /path/to/destination
  ```

> The `tar` flags are easier to remember as a mnemonic: **c**reate, e**x**tract, g**z**ip compression, **v**erbose output, and **f**ile (the archive name, which must come last). So "create a gzipped archive verbosely from a file" is `-czvf`.
{: .prompt-tip }

## Miscellaneous

- `echo` - Display a line of text

  ```bash
  echo "Hello, World!"
  echo $HOME
  ```

- `date` - Display or set the system date and time

  ```bash
  date
  date -s "2024-07-31 12:34:56"
  ```

- `whoami` - Print the current user name

  ```bash
  whoami
  ```

- `which` - Locate a command's executable

  ```bash
  which command_name
  ```

- `man` - Display the manual for a command

  ```bash
  man command_name
  ```

- `history` - Display the command history

  ```bash
  history
  ```

- `clear` - Clear the terminal screen

  ```bash
  clear
  ```

> `date -s` sets the clock manually, requires root, and will fight an NTP time-sync service if one is running. On most systems, let NTP manage the time instead.
{: .prompt-warning }

> Speed up recall: press `Ctrl+R` to search your history interactively, and type `!!` to rerun the previous command. Avoid pasting passwords or tokens on the command line, since they are saved in `history` in plain text.
{: .prompt-tip }
