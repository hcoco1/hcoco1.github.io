---
title: Common Linux Commands
author: hcoco1
date: 2024-02-03 14:10:00 +0800
categories: [Programming, Linux]
tags: [bash, cli, linux]
render_with_liquid: false
---

The Linux command line is a text interface to your computer. Often referred to as the shell, terminal, console, prompt or various other names, it can give the appearance of being complex and confusing to use. Yet the ability to copy and paste commands from a website, combined with the power and flexibility the command line offers, means that using it may be essential when trying to follow instructions online, including many on this very website!

## File and Directory Management

- `ls` - List directory contents
  ```bash
  ls
  ls -l
  ls -a
  ```

- `cd` - Change directory
  ```bash
  cd /path/to/directory
  cd ..
  cd ~
  ```

- `pwd` - Print working directory
  ```bash
  pwd
  ```

- `mkdir` - Create a new directory
  ```bash
  mkdir directory_name
  mkdir -p /path/to/directory
  ```

- `rmdir` - Remove an empty directory
  ```bash
  rmdir directory_name
  ```

- `rm` - Remove files or directories
  ```bash
  rm file_name
  rm -r directory_name
  ```

- `cp` - Copy files or directories
  ```bash
  cp source_file destination_file
  cp -r source_directory destination_directory
  ```

- `mv` - Move or rename files or directories
  ```bash
  mv source_file destination_file
  mv old_directory_name new_directory_name
  ```

- `touch` - Create an empty file or update the timestamp of an existing file
  ```bash
  touch file_name
  ```

- `find` - Search for files in a directory hierarchy
  ```bash
  find /path/to/search -name "file_name"
  ```

## File Viewing and Editing

- `cat` - Concatenate and display file content
  ```bash
  cat file_name
  ```

- `less` - View file content one screen at a time
  ```bash
  less file_name
  ```

- `more` - View file content one screen at a time
  ```bash
  more file_name
  ```

- `head` - Display the first few lines of a file
  ```bash
  head file_name
  head -n 10 file_name
  ```

- `tail` - Display the last few lines of a file
  ```bash
  tail file_name
  tail -n 10 file_name
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

## System Information and Management

- `top` - Display real-time system information
  ```bash
  top
  ```

- `ps` - Display information about active processes
  ```bash
  ps
  ps aux
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

- `uname` - Display system information
  ```bash
  uname
  uname -a
  ```

## Networking

- `ping` - Send ICMP ECHO_REQUEST to network hosts
  ```bash
  ping host_name_or_ip
  ```

- `ifconfig` - Configure a network interface (deprecated, use `ip` instead)
  ```bash
  ifconfig
  ```

- `ip` - Show/manipulate routing, devices, policy routing, and tunnels
  ```bash
  ip address
  ip link
  ```

- `netstat` - Network statistics (deprecated, use `ss` instead)
  ```bash
  netstat
  ```

- `ss` - Utility to investigate sockets
  ```bash
  ss
  ss -tuln
  ```

- `scp` - Securely copy files between hosts
  ```bash
  scp source_file user@remote_host:/path/to/destination
  ```

- `wget` - Non-interactive network downloader
  ```bash
  wget url
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

## Permissions and Ownership

- `chmod` - Change file modes or Access Control Lists
  ```bash
  chmod 755 file_name
  chmod -R 755 directory_name
  ```

- `chown` - Change file owner and group
  ```bash
  chown user_name:group_name file_name
  chown -R user_name:group_name directory_name
  ```

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

- `unzip` - Extract compressed files from a ZIP archive
  ```bash
  unzip archive_name.zip
  ```

## Miscellaneous

- `echo` - Display a line of text
  ```bash
  echo "Hello, World!"
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

- `man` - Display the manual for a command
  ```bash
  man command_name
  ```

- `history` - Display the command history
  ```bash
  history
  ```

