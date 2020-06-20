# Uri-Handler

I wanted to easily connect uri protocols to scripts on my Mac, and created this simple utility to make it work. One part is a macOS app that acts as a router, routing protocol requests to a script with the same name. 

## Installation

Download, unzip, and run setup

```bash
  ./setup
```

This will create a *~/.uri_handlers* directory with a sample handler called _edit_. It also copies the *uri-handler* macOS app to the _/Applications_ directory. This app acts as a router, routing protocol requests to scripts that handle the requests. Finally it creates a *.bash_profile* file in the *~/.uri_handlers* directory that is used by the macOS app to establish the environment used when running handlers. Initially the *.bash_profile* include your sessions path information. 

All handlers must reside in the *~/.uri_handlers* directory, and scripts are provided to add and remove handlers inside this directory (see below).

## Usage

### Test this install
Once installed test your setup by using pasting the following into your browsers address bar

```
edit://open/?file=~/.uri_handlers/.bash_profile
```
macOS will prompt you to approve both the protocol and the application the first time it is run. With each new protocol you add your will see a similar request to approve usage of the protocol. Be sure to __check__ the dialog box entry labeled _Always open these types of links in the associated app_ if you don't want to see this dialog every time you use the url protocol.

If everything is working you should see the custom *.bash_profile* file. You may want to add more to this file, or replace it with a symlink to your actual .bash_profile if you wish. 

### Add a handler
Handlers are simple shell scripts written in whatever language you want. The easiest way to add a handler is with the *add_handler* command. Simply provide the name of the protocol you wish to add, and the script will add a handler file, and also update the uri-handler app to register this new protocol with macOS. The default handler will simply open the TextEdit app, so you will want to customize the handler script. See the example below.

```
cd ~/.uri_handlers
./add_handler emacs
```

Edit your new handler to do what you want with the url information. For example the generated *~/.uri_handlers/emacs* script will initially be a basic template.

```python
#!/usr/bin/env python
import sys
import subprocess
from urllib.parse import urlparse, parse_qs

if __name__ == "__main__":
    if len(sys.argv) == 2:
        uri = sys.argv[1]
        result = urlparse(uri)
        query = parse_qs(result.query, keep_blank_values=True)
        filename = query["file"][0]
        subprocess.run("open -a TextEdit " + filename, shell=True)
    else:
        print("Error: no url provided")
```

I modified it to look like this.

```python
#!/usr/bin/env python
import sys
import subprocess
from urllib.parse import urlparse, parse_qs

if __name__ == "__main__":
    if len(sys.argv) == 2:
        uri = sys.argv[1]
        result = urlparse(uri)
        query = parse_qs(result.query, keep_blank_values=True)
        if "line" in query:
            line = query["line"][0]
        else:
            line = "0"
        if "column" in query:
            column = query["column"][0]
        else:
            column = "0"
        if "file" in query:
            filename = query["file"][0]
        else:
            filename = "*scratch*"
            line = "0"
            column = "0"

        subprocess.Popen(["/usr/local/Cellar/emacs/26.1_1/bin/emacsclient", "+" + line + ":" + column, filename])
    else:
        print("Error: no url provided")
```

I have a modified logger that adds file location links to debug messages in this format so I can easily open Emacs from debug messages in the terminal. Example
```
emacs://open/?file=~/.bash_profile&line=25&column=0
```

That is it, whenever you modify the handler script the changes are immediate. 

## Removing A Handler
To remove a handler call *remove_handler* with the name of the handler you want removed. The *remove_handler* script will delete the handler file and remove the entry from the *uri_handler* applications list of protocols it accepts.

```
cd ~/.uri_handlers
./remove_handler emacs
```

## Potential Issues
The biggest issue is that the *uri_handler* app relies on *~/uri_handlers/.bash_profile* to setup the environment for the spawned process, so if you are using Python, Ruby, etc or spawning apps like emacsclient then make sure to add everything you need specified in the environment into *~/uri_handlers/.bash_profile*.

Script can always be run from the command line, just pass in a uri string. So it makes them easy to test. In most cases if it doesn't work it is a script issue, so test your scripts in the terminal to see any errors.

```
./edit "edit://open/?file=~/.uri_handlers/.bash_profile"
```

If it works in the terminal but not using the browser address bar is likely an issue with the *~/uri_handlers/.bash_profile* missing some needed environment information. To test this case try this (this example assumes ~/.bash_profile is your startup profile, adjust accordingly)

```
cd ~/.uri_handlers/
mv .bash_profile .bash_profile_bk
ln -s ~/.bash_profile .bash_profile
```

If it works now then either keep the symlink, or work on fixing *~/.uri_handlers/.bash_profile* 


You can install [RCDefaultApp](http://www.rubicode.com/Software/RCDefaultApp/) to see if macOS is actually using the *uri-handler* app as the protocol default, and make changes if not. 

That's it. 
