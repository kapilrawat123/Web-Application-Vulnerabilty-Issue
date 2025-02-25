# 🎃All about Yara 

_"The pattern matching swiss knife for malware researchers (and everyone else)" ([Virustotal., 2020](https://virustotal.github.io/yara/))_

With such a fitting quote, Yara can identify information based on both binary and textual patterns, such as hexadecimal and strings contained within a file.

  

Rules are used to label these patterns. For example, Yara rules are frequently written to determine if a file is malicious or not, based upon the features - or patterns - it presents. Strings are a fundamental component of programming languages. Applications use strings to store data such as text.

> [!NOTE]
> For example, the code snippet below prints "Hello World" in Python. The text "Hello World" would be stored as a string.
> 
> ```python
> print("Hello World!")
> ```
> 
>   
> 
> We could write a Yara rule to search for "hello world" in every program on our operating system if we would like. 

### Why does Malware use Strings?

Malware, just like our "Hello World" application, uses strings to store textual data. Here are a few examples of the data that various malware types store within strings:

| **Type**   | **Data**                                                                                                        | **Description**                                        |
| ---------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| Ransomware | [12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw](https://www.blockchain.com/btc/address/12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw) | Bitcoin Wallet for ransom payments                     |
| Botnet     | 12.34.56.7                                                                                                      | The IP address of the Command and Control (C&C) server |

---

# ✨Introduction to Yara Rules

### Your First Yara Rule

The proprietary language that Yara uses for rules is fairly trivial to pick up, but hard to master. This is because your rule is only as effective as your understanding of the patterns you want to search for.  
  
Using a Yara rule is simple. Every `yara` command requires two arguments to be valid, these are:  
**1)** The rule file we create  
**2)** Name of file, directory, or process ID to use the rule for.  
  
Every rule must have a name and condition. For example, if we wanted to use "myrule.yar" on directory "some directory", we would use the following command:  
`yara myrule.yar somedirectory`  
  
Note that **.yar** is the standard file extension for all Yara rules. We'll make one of the most basic rules you can make below.  
1. Make a file named "**somefile**" via `touch somefile`  
2. Create a new file and name it "**myfirstrule.yar**" like below:

> [!*]
> Creating a file named somefile
> 
> ```
> cmnatic@thm:~$ touch somefile
> ```  
> 

> [!*]
> Creating a file named myfirstrule.yar
> 
> ```
> cmnatic@thm touch myfirstrule.yar\
> ```
>  
> 

3. Open the "myfirstrule.yar" using a text editor such as `nano` and input the snippet below and save the file:

> [!NOTE]
> 
> ```yaml
> rule examplerule {
>         condition: true
> }
> ```
> 

> [!NOTE]
> Inputting our first snippet into "myfirstrule.yar" using nano
> 
> ```
> cmnatic@thm nano myfirstrule.yar   GNU nano 4.8 myfirstrule.yar 
> rule examplerule {
>         condition: true 
> }
> ```
>   

The **name** of the rule in this snippet is `examplerule`, where we have one condition - in this case, the **condition** is `condition`.

As previously discussed, every rule requires both a name and a condition to be valid. This rule has satisfied those two requirements.

Simply, the rule we have made checks to see if the file/directory/PID that we specify exists via `condition: true`. If the file does exist, we are given the output of `examplerule`

Let's give this a try on the file **"somefile"** that we made in step one:  
`yara myfirstrule.yar somefile`  
  
If "somefile" exists, Yara will say `examplerule` because the pattern has been met - as we can see below:

  

> [!NOTE]
> Verifying our the examplerule is correct
> 
> ```
> cmnatic@thm:~$ yara myfirstrule.yar somefile  examplerule somefile
> ```  

If the file does not exist, Yara will output an error such as that below:  

> [!NOTE]
> Yara complaining that the file does not exist
> 
>    ```
>    cmnatic@thm:~$ yara myfirstrule.yar sometextfile error scanning sometextfile: could not open file
>    ```

Congrats! You've made your first rule.


---

# ✨natomy of a Yara Rule

![a picture showing a Yara rule cheatsheet developed by fr0gger_](https://miro.medium.com/max/875/1*gThGNPenpT-AS-gjr8JCtA.png)

---

# ✨ YARA Modules

### Integrating With Other Libraries  

Frameworks such as the [Cuckoo Sandbox](https://github.com/cuckoosandbox/cuckoo) or [Python's PE Module](https://pypi.org/project/pefile/) allow you to improve the technicality of your Yara rules ten-fold.  

### Cuckoo

Cuckoo Sandbox is an automated malware analysis environment. This module allows you to generate Yara rules based upon the behaviours discovered from Cuckoo Sandbox. As this environment executes malware, you can create rules on specific behaviours such as runtime strings and the like.
### Python PE

Python's PE module allows you to create Yara rules from the various sections and elements of the Windows Portable Executable (PE) structure.


---

# ✨ Other tools and Yara

## Yara Tools

Knowing how to create custom Yara rules is useful, but luckily you don't have to create many rules from scratch to begin using Yara to search for evil. There are plenty of GitHub [resources](https://github.com/InQuest/awesome-yara) and open-source tools (along with commercial products) that can be utilized to leverage Yara in hunt operations and/or incident response engagements. 

#### 1. LOKI (What, not who, is Loki?)

LOKI is a free open-source IOC (_Indicator of Compromise_) scanner created/written by Florian Roth.

Based on the GitHub page, detection is based on 4 methods:

1. File Name IOC Check
2. Yara Rule Check **(we are here)**
3. Hash Check
4. C2 Back Connect Check

There are additional checks that LOKI can be used for. For a full rundown, please reference the [GitHub readme](https://github.com/Neo23x0/Loki/blob/master/README.md).

LOKI can be used on both Windows and Linux systems and can be downloaded [here](https://github.com/Neo23x0/Loki/releases).

#### 2. THOR (_superhero named programs for a superhero blue teamer)_

THOR _Lite_ is Florian's newest multi-platform IOC AND YARA scanner. There are precompiled versions for Windows, Linux, and macOS. A nice feature with THOR Lite is its scan throttling to limit exhausting CPU resources. For more information and/or to download the binary, start [here](https://www.nextron-systems.com/thor-lite/). You need to subscribe to their mailing list to obtain a copy of the binary. **Note that THOR is geared towards corporate customers**. THOR Lite is the free version.

#### 3. FENRIR (naming convention still mythical themed)

This is the 3rd [tool](https://github.com/Neo23x0/Fenrir) created by Neo23x0 (Florian Roth). You guessed it; the previous 2 are named above. The updated version was created to address the issue from its predecessors, where requirements must be met for them to function. Fenrir is a bash script; it will run on any system capable of running bash (nowadays even Windows).

#### 4. YAYA (_Yet Another Yara Automaton_)

YAYA was created by the [EFF](https://www.eff.org/deeplinks/2020/09/introducing-yaya-new-threat-hunting-tool-eff-threat-lab) (_Electronic Frontier Foundation_) and released in September 2020. Based on their website, "_YAYA is a new open-source tool to help researchers manage multiple YARA rule repositories. YAYA starts by importing a set of high-quality YARA rules and then lets researchers add their own rules, disable specific rulesets, and run scans of files._"


---

# ✨ Creating Yara rules with yarGen

with the help of yarGen we can create a yara rule a way faster rather then typing one by one rule 

https://github.com/Neo23x0/yarGen


---

# 🎃 **Valhalla**

**Valhalla** is an online Yara feed created and hosted by [Nextron-Systems](https://www.nextron-systems.com/valhalla/) (erm, Florian Roth). By now, you should be aware of the ridiculous amount of time and energy Florian has dedicated to creating these tools for the community. Maybe we should have just called this the Florian Roth room. (lol)

Per the website, "_Valhalla boosts your detection capabilities with the power of thousands of hand-crafted high-quality YARA rules._"

![A picture displaying the search menu for the Valhalla tool.](https://assets.tryhackme.com/additional/yara/yara13.png)  

From the image above, we should denote that we can conduct searches based on a keyword, tag, ATT&CK technique, sha256, or rule name. 

**Note**: For more information on ATT&CK, please visit the [MITRE](https://tryhackme.com/room/mitre) room. 

Taking a look at the data provided to us, let's examine the rule in the screenshot below:

![A picture displaying the results of a rule, depicting the rule name, description and the date it was submitted to Valhalla](https://assets.tryhackme.com/additional/yara/yara14.png)  

We are provided with the name of the rule, a brief description, a reference link for more information about the rule, along with the rule date. 

Feel free to look at some rules to become familiar with the usefulness of Valhalla. The best way to learn the product is by just jumping right in.

