{:title "Using linux utilities in a real world usecase"
 :layout :post
 :tags  ["sed" "awk" "linux" "cut" "grep" "head"]}

## Real world problem
Today we are going to see what `sed` and a little bit of `awk` are and how we can use it in [Pen Testing](https://en.wikipedia.org/wiki/Penetration_test). I'll assume you are using any linux machine and are familiar with how to open the command line.

In 2011 a hacker group called [Anonymous](https://en.wikipedia.org/wiki/Anonymous_(group)) attacked [rootkit.com](rootkit.com) and uploaded the whole database of the site online. Its a huge database. With some *80,000+* users and many other information in it.

Lets say we are the Pen Testers. And we just got hold of that huge database. Inside the database there are many tables, but the one that is of your interest is called `people` and it has many fields, but you are interested in only the *username* and its *hashed password*. We want to extract all the username and its hash in this format

```
userid1:password1
userid2:password2
....
```

You certainly can't do this manually. And it doesn't suit us doing manual work, Ahmm .. `Ctrl-C -> Ctrl-V` :)

We like to do things fast and smart.... right? ;)
sed, grep & awk  too) to the rescue!

## What is sed?

So what is this... *sed*?

`sed` is a Stream Editor in which we feed some text, and it processes them line by line and performs some commands which manipulate the text in the way we want. For example We can replace all `<SPACE>` to `:` OR replace all the occurrences of the string `hello` to `hi` and many awesome stuffs.
Hold ...Hold, before you say  *"Big deal! I can do that with `Replace All` command in my Notepad"*  (yeah even I thought the same before I learned sed)

OK, Lets start the Magic Show!

## Explore the problem
Here's the [link](https://dl.dropboxusercontent.com/u/6104759/blog-data/rootkit_com_mysqlbackup_02_06_11.gz), download the gzip file `rootkit_com_mysqlbackup_02_06_11.gz`, and paste it in any folder in your Linux machine.

Once you have downloaded the file, rename the file so that its small

```bash
$ mv rootkit_com_mysqlbackup_02_06_11.gz database.gz
```

Now we need to decompress the file.

```bash
$ gunzip database.gz
```

Now that you have decompressed it, you should have a file called `database` (without any `.gz`)

OK. Just to get a feel of how BIG the database is just run `cat` command on the database file, go have some coke, sleep and come back. :) No, don't worry, it is possible to extract fields from this huge file in a very clever yet elegant way. Hold On, magic is about to begin.

OK. First we need to know what we are dealing with.
Open database file in `vim`.
We will search for all the Create Table Statements. Do

```
/CREATE TABLE [ENTER]
```

and keep pressing `n` for next occurrence of the given string. You will notice that there are so many tables in this database. Keep going till you hit the `people` database. Saw? Okay good. 

Now keep going down (using down arrow key) slowly and keep noticing the fields (yeah there are many fields in this table). You will hit the insert into table and notice that the insert into line span for multiple lines.

```
INSERT INTO `people` VALUES (1,'admin','51a42fa118e77f95f70d4efff4395f8d','rootkit sysop','hoglund@rootkit.com',10,0,'','','','','','',0,'http://www.rootkit.com/usericons/admin.jpg','',1296966693,'213.243.145.60',1296705113,1283501911,1296457930,1294556469,1294942812,0,0,'','','','','',-1,'P'),(2,'aaronh','7cb8b36d','Aaron Heady','hackdoctor@aol.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/aaronh.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''),(3,'abcdef','e80

..........................................................and so on
```
To be sure that one line is really spanning multiple lines do the following command inside Vim

```
<ESC> :set nu
```

And it should show you the line number at which insert into is. If you are at first insert into

```
(1,'admin','51a42fa118e77f95f70d4efff4395f8d','rootkit...
```

then the line should be `425`. Anyway the point was it is spanning multiple lines. It is important to know this. Why? Because `cut` command cuts per line. What is this `cut` command you ask? We will see that later in detail. The mist is about to clear. Forget `cut` for now.

Now that you know what lines you want to work on i.e. 

```
INSERT INTO `people`
```

(Note around people is not single quotes but backtick, which lies above the tab key)

## grep - print lines matching a pattern

To select only particular lines from this file we will use `grep` command.
`grep` command takes a file as input and one or more strings to match. The lines that are matched are returned from that file.

OK quit from the vim. To quit do

```
<ESC> :q!
```

Oh just press `Ctrl-l` to clear the screen. :) if you are wondering how to clear so much clutter on the screen.

OK Do,

```bash
$ grep "INSERT INTO \`people\`" database
```

You will see a huge amount of output even now, but don't worry we have extracted out only the following statements.

```
INSERT INTO `people`
```

Wait why we put those **\\** in front of Backticks?
Because backticks are special characters and we want linux to treat them as normal characters. To make any special character normal character we put **\\** in front of them.

Ok. Now Very Important part.
Each insert into statement is inserting many values within.

```
(1,'admin',...),(2, 'aaronh',....) etc
```

## Let's use sed

To simplify things we are going to put each row of value in seperate line.
How we are going to do this? By asking `sed` to substitute `),(` with a newline. Why `),(` ? Because that is where your one row is ending and a new one is beginning.

So we do,

```bash
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g
```

Ok. Is that too much? :) Let me explain.


1. Following is what we did before.
    ```
grep "INSERT INTO \`people\`" database
    ```
	
2. `|`
    - This is the pipe command. 
    - It passes the output of one command as input to the other command. 
    - The selected text from the grep command is passed as input to the sed command. Simple!
    
3. In the following:

```
sed s/\)\,\(/\\n/g
```

	- Here `s` stands for substitute.
	- what to replace is told after the first `/` i.e. to replace string `),(`
	- We added **\\** before `)` and `(` so that its treated as normal characters not special characters.
	- 2nd **/** specifies the string to replace with to replace with string is newline, i.e. `\n` but since **\\** is a special character we make it normal character buy adding one more **\\** :)
	- 3rd **/** specifies substitute all the occurrences (g = global) of the `,` to replace string
	- Note Replacing a string and putting a newline is something you cannot do in notepad with replace All :)

## peek the output with head

Now when you run the command, you yet cant see the output.
Append the command with a `head` command. By default `head` command outputs only first ten lines of text file given (and `tail` command does the opposite)

So just do

```bash
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | head  
```

and this should be the output

```
.....
INSERT INTO `people` VALUES (1,'admin','51a42fa118e77f95f70d4efff4395f8d','rootkit sysop','hoglund@rootkit.com',10,0,'','','','','','',0,'http://www.rootkit.com/usericons/admin.jpg','',1296966693,'213.243.145.60',1296705113,1283501911,1296457930,1294556469,1294942812,0,0,'','','','','',-1,'P'
2,'aaronh','7cb8b36d','Aaron Heady','hackdoctor@aol.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/aaronh.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
3,'abcdef','e80b5017','Ashish Rungta','ASHTME@YAHOO.COM',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/abcdef.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
4,'abel','cd779e8a','Adi A','adia@opsynet.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/abel.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
5,'abhi0070','a6a7c0ce','kbcack','unknownbuddy@yahoo.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/abhi0070.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
6,'abm','e50624ea','alex murphy','abm@mitre.org',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/abm.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
7,'abraxas','7f9cc44f','Alex Mellor','i_love_g0ats@hotmail.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/abraxas.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
8,'acc_chen','c4025c6f','Jun.chen','johnychen@netease.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/acc_chen.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
9,'access55','e9f5bda6','WHY YOU ASK','access55@manx.net',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/access55.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
10,'accobra','66f363a6','Rob','rlstephe@yahoo.com',1,0,'','','','','','',0,'http://www.rootkit.com/usericons/accobra.jpg','',0,'',0,0,0,0,0,0,0,'','0','','','',-1,''
..................
```

I can see a faint smile on your face now :) Don't worry you will be having a strong urge to show off at the end of this tutorial. Just a few steps more.

## cut me baby one more time...

Ok. Before we extract out the user and  password. You need to know what is `cut` command. The `cut` command works on field seperators.

To understand how `cut` works before we contd, lets take an example. Do

```bash
$ cat /etc/passwd
.....
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
........
```

All the fields in this file are separated by `:` as follows

1. userid
2. password
3. uid 
4. ... and so on.

What if you want to extract only user and uid from this file?
This is where `cut` comes into play. `cut` command works on field separators, for each line. By default field separator is space but we can specify any character as field separator.

```bash
$ cut -d":" -f1,3 /etc/passwd
........
root:0
daemon:1
bin:2
sys:3
..........
```

## It's time to cut!

Nice.. Now we are going to cut each line by comma. Why? Cause each field is seperated by commas. We want user and hash which is 2nd and 3rd field if the field separator is `,`

```bash
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | cut -d "," -f2,3 | head
```

* Field seperator is a comma => `-d ","`
* Select only column 2 and 3 => `-f2,3`
* And pass it to `head` so that we get first ten lines only (its easy to see output and be sure that the command is working)

It should give

```
.........
'admin','51a42fa118e77f95f70d4efff4395f8d'
'aaronh','7cb8b36d'
'abcdef','e80b5017'
'abel','cd779e8a'
'abhi0070','a6a7c0ce'
'abm','e50624ea'
'abraxas','7f9cc44f'
'acc_chen','c4025c6f'
'access55','e9f5bda6'
'accobra','66f363a6'
.....
```

[すごい (凄い)](http://haikugirl.me/2014/04/06/word-of-the-week-%E3%81%99%E3%81%94%E3%81%84/)

## Victory is close!

Now we just need to remove those single quotes. You know what to do :)

```bash
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | cut -d "," -f2,3 | sed s/\'//g | head
```

i.e. 

* replace all `(/g)`
* single quotes `/\'` as Single Quotes is a special character)
* with Nothing `//`

```
......
admin,51a42fa118e77f95f70d4efff4395f8d
aaronh,7cb8b36d
abcdef,e80b5017
abel,cd779e8a
abhi0070,a6a7c0ce
abm,e50624ea
abraxas,7f9cc44f
acc_chen,c4025c6f
access55,e9f5bda6
accobra,66f363a6
....
```

Last thing, and its Game Over, replace `,` with `:`

## Finish Him!

```bash
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | cut -d "," -f2,3 | sed s/\'//g | sed s/,/:/g | head
```

:) Done!

You can now remove the head command to check if its working for all files.

```bash
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | cut -d "," -f2,3 | sed s/\'//g | sed s/,/:/g
```

Awesome!

## Let's bring awk to the party!

Wait there's more.
What if you want hash first and user name later? i.e.

```
password1:user1
password2:user2
....

```

:)

With sed you can do, but its too complicated.
Meet Sed's Elder brother Awk. `Awk` is more powerful (and more complicated). Awk is used mainly for data extraction and reporting tool.

Do,

```
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | cut -d "," -f2,3 | sed s/\'//g | sed s/,/:/g | awk -F':' '{print $2 ":" $1}' | head
```

What we did?

* `-F` is like `-d` in `cut` command, i.e. specifying field separator.
* Each field is put in `$1 $2 $3` etc, where `$1` is username and `$2` is password.
* By `'{print $2 ":" $1}'` we are asking it to print it in reverse order. Don't give Comma in between `$2` and `$1` as it will replace the field separator with space.

Output ->

```
.......
51a42fa118e77f95f70d4efff4395f8d:admin
7cb8b36d:aaronh
e80b5017:abcdef
cd779e8a:abel
a6a7c0ce:abhi0070
e50624ea:abm
7f9cc44f:abraxas
c4025c6f:acc_chen
e9f5bda6:access55
66f363a6:accobra
..........
```

Now just redirect the final output (without the `head` command) to a file.

```
$ grep "INSERT INTO \`people\`" database | sed s/\),\(/\\n/g | cut -d "," -f2,3 | sed s/\'//g | sed s/,/:/g | awk -F':' '{print $2 ":" $1}' > output.txt
```

With `> output.txt` we are redirecting the final output of `awk` command into the file i.e. `output.txt` rather than on the terminal.

Now just for curiosity, to count the number of users we got we do

```
$ wc output.txt -l
81450
```

So we have **81450** users in this final `output.txt` Pheww Thats a lot!


OK.

## (Optional) Super Crisp Command Mode On!

Above command has Total 6 Commands (excluding `> output.txt`) i.e.

```bash
grep "INSERT INTO \`people\`" database
| sed s/\),\(/\\n/g
| cut -d "," -f2,3
| sed s/\'//g
| sed s/,/:/g
| awk -F',' '{print $2 $1}'
```

Do you think we can crisp it down to only 2 Commands!!
No, I am not crazy. It is possible.
Why do I want to do it? Just cause we can ;)


Wanna See ? :)
Here's the command

```bash
$ sed -n /"INSERT INTO \`people\`"/s/\),\(/\\n/pg database | gawk -F\' '{print $4 ":" $2}' | head
.....
```

PS: `head` command is for you to see the short version of the output, not required though

What is going on? Thats for you to figure it out.

Have Fun! :D
