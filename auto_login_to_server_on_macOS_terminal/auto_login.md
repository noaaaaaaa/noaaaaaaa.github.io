# Auto login to server on macOS terminal

If you are a user of serveral servers and want to simplify the process everytime you login, then you will love this trick.

## Create a `computerInfo.ini` file to store your login information

`cd .shh`
`touch computerInfo.ini`
`vim computerInfo.ini`
`i` to enter edit mode
put in `ip port user_name password name_of_server`, separate your account in different lines

## Create a `core.ex` file to mimic the pipeline of entering information

```js
#!/usr/bin/expect
set ip [lindex $argv 0]
set port [lindex $argv 1]
set username [lindex $argv 2]
set password [lindex $argv 3]
set timeout -1
spawn ssh -p $port $username@$ip
expect {
    "password" {send "$password\r";}
    "yes/no" {send "yes\r";exp_continue}
}
interact
```

## Creat `login.sh` to decrypt both `core.ex` and `computerInfo.ini` file

`touch login.sh`
`vim login.sh`
`i`
Put the text below in the `login.sh` file

```js
#!/bin/bash
file="computerInfo.ini"
awk '{if (NR > 1 && $1 != ""){printf "%-2s %-45s %-15s \n","("NR-1")",$5,$1}}' $file
echo "please choose which machine to login:"
read number
number=$[number+1]
read ip port user password <<< $(echo `awk 'NR=="'$number'"{print $1,$2,$3,$4}' $file`)
./core.ex $ip $port $user $password
```

### Warning

Don't forget to `chmod 777 [file]` all these 3 files respectively

## Install `expect`

Notice that `core.ex` file can only be run with `expect`

`brew install expect`

## Alias the command to make login easier

Put `alias login='/Users/apple/.ssh/login.sh'` in `.bashrc`

and `source .bashrc`

## Well done, type `login` to see what you can get!
