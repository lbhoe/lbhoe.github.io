---
title: "Holiday Hack Challenge 2022: 1 - Recover the Tolkien Ring"
date: 2022-12-29 08:00:00 +0800
categories:
  - "Problem Solving"
  - "Holiday Hack Challenge 2022"
tags: [forensics]
image:
    path: /assets/img/holiday_hack_challenge_2022/holiday_hack_challenge_2022_banner.png
---
**Network Security**

Analyze a PCAP to identify malware, and then analyze logs and create IDS rules to detect such attacks.

## Wireshark Practice

>Difficulty: 1/5
>
>Use the Wireshark Phishing terminal in the Tolkien Ring to solve the mysteries around the suspicious PCAP. Get hints for this challenge by typing hint in the upper panel of the terminal
>
>Required File: `suspicious.pcap`

There are objects in the PCAP file that can be exported by Wireshark and/or Tshark. What type of object can be exported from this PCAP?

`http`

![image](/assets/img/holiday_hack_challenge_2022/da191c7470f5a76bc811dbcd2b8d4dbd0f0866c66c0c605e5a2111a99a0067d6.png)  

What is the file name of the largest file we can export?

`app.php`

![image](/assets/img/holiday_hack_challenge_2022/be1c31b9142937f5c122a95c9af5024bc4c5611f9093b252cf2dba9519b7853c.png)  

What packet number starts that `app.php` file?

`687`

What is the IP of the Apache server?

`192.185.57.242`

![image](/assets/img/holiday_hack_challenge_2022/9879aaea68ccd625e2b40473a00d9ccdbe3e97f0d1d8dcae234c2105af9c2f89.png)  

What file is saved to the infected host?

`Ref_Sept24-2020.zip`

![image](/assets/img/holiday_hack_challenge_2022/371e48bd03f989b7d6c44f9a40cc28f740dbbe862d26dcd3cbdb162a3e50949b.png)  

Command Line Solution

![image](/assets/img/holiday_hack_challenge_2022/bd5dc5b3e465a591a55afdaf36f80c757686f65e5c4d80b4645d872581cb53c1.png)  

```bash
$ tshark -r pcap_challenge.pcap --export-object http,export

$ cd export/

$ grep -E '.\.zip' *
app(1).php:    saveAs(blob1, 'Ref_Sept24-2020.zip');
```

Attackers used bad TLS certificates in this traffic. Which countries were they registered to? Submit the names of the countries in alphabetical order separated by a comma `(Ex: Norway, South Korea)`.

`Israel, South Sudan`

NetworkMiner `(Files > Filter for TlsCertificate > Observed that {C: IL, SS, US})`

![image](/assets/img/holiday_hack_challenge_2022/905388acaa6b4020c6687d87fd0a143e906f6eab38ba6707267f41020cf6f9d3.png)  

Command Line Solution

![image](/assets/img/holiday_hack_challenge_2022/3621d5d40311e7e47ee19314ea07ca7d8f96e9dae7082330c68a9ea7191c8239.png)  

```bash
$ tshark -r pcap_challenge.pcap -2R 'tls' -V | grep -oh -E 'countryName=..' | sort | uniq
countryName=IE
countryName=IL
countryName=SS
countryName=US
```

Is the host infected?

`Yes`

## Windows Event Logs

>Difficulty: 2/5
>
>Investigate the Windows event log mystery in the terminal or offline. Get hints for this challenge by typing hint in the upper panel of the Windows Event Logs terminal.
>
>Required File: `powershell.evtx`

Processed the file using EvtxECmd module with KAPE. Viewed output on TimelineExplorer.

What `month/day/year` did the attack take place? For example, `09/05/2021`.

`12/24/2022`

![image](/assets/img/holiday_hack_challenge_2022/04aeba38c722c0895887be33bcb96de2f365bbf8b5db3381da72979b3988e19e.png)  

An attacker got a secret from a file. What was the original file’s name?

`Recipe`

![image](/assets/img/holiday_hack_challenge_2022/a4d5c56ef33f2b65da2ce7824ca295efa8cc421f0590fc16578f694361688270.png)  

The contents of the previous file were retrieved, changed, and stored to a variable by the attacker. This was done multiple times. Submit the last full Powershell line that performed only these actions.

`$foo = Get-Content .\Recipe| % {$_ -replace 'honey', 'fish oil'}`

![image](/assets/img/holiday_hack_challenge_2022/c91c595b84c9905835669653894389c4f4d57ab38f2fd9a82f8fa86fcb67c1af.png)  

After storing the altered file contents into the variable, the attacker used the variable to run a separate command that wrote the modified data to a file. This was done multiple times. Submit the last full PowerShell line that performed only this action.

`$foo | Add-Content -Path 'Recipe'`

![image](/assets/img/holiday_hack_challenge_2022/e972bc2a4a8170660af79064d9f2fd0159cadde8fcb0e4969a101cbc07bd417e.png)  

The attacker ran the previous command against one file multiple times. What is the name of this file?

`Recipe.txt`

![image](/assets/img/holiday_hack_challenge_2022/40c06b62f1c85c2427eb33e571e656a8464bf2df5d94fe371c6e45d51e4941e3.png)  

Were any files deleted? `(Yes/No)`

`Yes`

```
2022-12-24 11:05:51, ScriptBlockText: del .\recipe_updated.txt
2022-12-24 11:05:42, ScriptBlockText: del .\Recipe.txt
```

Was the original file `(from question 2)` deleted? `(Yes/No)`

`No`

What is the Event ID of the logs that show the actual command lines the attacker typed and ran?

`4104`

![image](/assets/img/holiday_hack_challenge_2022/82f870e9d79f2fb6c372f29ec4b3a25a6942c53301944e8a57ec12bafcf553f3.png)  

Is the secret ingredient compromised `(Yes/No)`?

`Yes`

![image](/assets/img/holiday_hack_challenge_2022/510324d8bab1bf94cb6e26b5f0317d1700ca47b403f085be6a9ee7c9a3738e6a.png)  

What is the secret ingredient?

`honey`

## Suricata Regatta

>Difficulty: 3/5
>
>Help detect this kind of malicious activity in the future by writing some Suricata rules. Work with Dusty Giftwrap in the Tolkien Ring to get some hints.

Useful commands to note

Remove last line from `suricata.rules`

```bash
$ sed -i '$d' suricata.rules
```

Run `rule_checker`

```bash
$ ./rule_checker
```

```
First, please create a Suricata rule to catch DNS lookups for adv.epostoday.uk.

Whenever there's a match, the alert message (msg) should read Known bad DNS lookup, possible Dridex infection.
```

```bash
$ echo 'alert dns any any -> any any (msg:"Known bad DNS lookup, possible Dridex infection";dns.query;content:"epostoday";)' >> suricata.rules
```

```
First rule looks good!

STINC thanks you for your work with that DNS record! In this PCAP, it points to 192.185.57.242.

Develop a Suricata rule that alerts whenever the infected IP address 192.185.57.242 communicates with internal systems over HTTP.

When there's a match, the message (msg) should read Investigate suspicious connections, possible Dridex infection
```

```bash
$ echo 'alert http 192.185.57.242/32 any <> any any (msg:"Investigate suspicious connections, possible Dridex infection";sid:5551;)' >> suricata.rules
```

```
Second rule looks good!

We heard that some naughty actors are using TLS certificates with a specific CN.

Develop a Suricata rule to match and alert on an SSL certificate for heardbellith.Icanwepeh.nagoya.

When your rule matches, the message (msg) should read Investigate bad certificates, possible Dridex infection
```

```bash
$ echo 'alert tls any any -> any any (msg:"Investigate bad certificates, possible Dridex infection";tls.certs;content:"heardbellith.Icanwepeh.nagoya";sid:5552;)' >> suricata.rules
```

```
Third rule looks good!

OK, one more to rule them all and in the darkness find them.

Let's watch for one line from the JavaScript: let byteCharacters = atob

Oh, and that string might be GZip compressed - I hope that's OK!

Just in case they try this again, please alert on that HTTP data with message Suspicious JavaScript function, possible Dridex infection
```

```bash
$ echo 'alert http any any -> any any (msg:"Suspicious JavaScript function, possible Dridex infection";file_data;content:"let byteCharacters = atob";sid:5553;)' >> suricata.rules
```

Challenge End.