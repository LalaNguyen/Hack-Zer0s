---
title       : Source Code Auditing - Day 1
author      : Nguyen Hoang Minh
job         : Teaching Assistant
framework   : io2012
highlighter : highlight.js
hitheme     : github
widgets     : []  # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides

---

## Agenda
1. Introduction to Common Weakness Enumerator (CWE).
2. Source Code Auditing.
3. Memory Corruption.
4. Auditing Case: Improper Null Termination (CWE-170)
5. Countering the issue of Software Defects before Hackers strike.

--- .class2 #id2 bg:white

## About CWE

<q><span style="color:red">Targeted</span> to developers and security practitioners. CWE is a formal list of <span style="color:red">software weakness types</span>. CWE provides a baseline for <span style="color:red">vulnerability identification</span>, mitigation, and prevention efforts.</q> 

<span style="float:right;">source: official CWE site.</span>

---&twocol .class3 #id3 bg:white

## Types of Software Vulnerability   

*** =left

- Buffer Overflows, Format Strings, Etc.
- Structure and Validity Problems
- Common Special Element Manipulations
- Channel and Path Errors
- Handler Errors
- User Interface Errors

*** =right

- Pathname Traversal and Equivalence Errors
- Authentication Errors
- Resource Management Errors
- Insufficient Verification of Data
- Code Evaluation and Injection
- Randomness and Predictability

--- .class4 #id4 bg:white
## 
 <iframe src = 'https://cwe.mitre.org/data/definitions/170.html' height='1200px'></iframe>
 

--- .class5 #id5 bg:white

## Source Code Auditing

- The process of <span style="color:red">reading source code to find vulnerabilities</span>.
- Necessary for <span style="color:red">finding implementation bugs</span>
 * Memory Corruption (Heap / Stack Memory Errors, Overwritten Addresses)
 * Memory Leak (HeartBleed)
 * In-line Injection (SQL, filenames, XSS)
- Throughly understanding programming language is necessary.
- Be able to <span style="color:red">Trace & Debug</span> by observing stack memory.


_Note_: The designer has good intentions. But somehow mistakenly implement a bug due to not properly handle CWEs.


![width](assets/img/heartbleed.png)

---&twocol .class6 #id6 bg:white

## Methodology

*** =left

- There is <span style="color:red">NEVER</span> enough time.
 * Large Code-lines.
 * Time engagement.
- You do not want your <span style="color:red">clients</span> to check code for you.
- <span style="color:red">The good practice to avoid bugs</span> is to self-audit your coding styles.
- <span style="color:red">Do not</span> leave variable uninitialized that would lead to vulnerabilities.

=> Using <span style="color:red">Code Auditing Tools</span> will save you time : ).
*** =right

```{c}
int doAuth(char *name, char *pw)
   {
        int auth;
        if (userIsBanned(name))
          {
                auth = 0;
          }
        if (authenticateUser(name, pw))
          {
                auth = 1;
          }
        return auth;
   }
```

---&twocol .class7 #id7 bg:white

## Code Auditing Tools (CATs)

The strength of using CATs is:
 * Highlighted editors & Navigators are useful.
 * Basic auxiality functions are provided.
  - Tracking variables, functions
  - Generic search, auto-complete.

*** =left
<b>Syntax Aware Tools</b>
 * VI / VIM.
 * Notepad++ / Sublime / TextMate / Atom.
 * Eclipse / Visual Studio / Xcode.
 
<span style="color:red">Weakness</span>: Lack of comprehensive view for the entire projects. It often involves gnanular view.

*** =right
<b>Static Analysis Tools</b>
- Fortify
- Klocwork
- Coverity
- SonarQube
- CPPCheck (online)

<span style="color:red">Weakness</span>: Many false positives & often miss vulnerabilities without human interventions.

--- .class8 #id8 bg:white

## Implementation Bugs

Implementation Bug is an <span style="color:red">unintentionally written line-codes</span> that allow attacker to deviate the strange actions from legistimate design.

They are commonly caused by:
 * Failure to validate input. (XSS, SQL Injection)
 * Misuse an Application Programming Interface. (Facebook API)
 * Miscalculation & Inappropriate Error Handling Techniques. (Buffer Overflow)

<span style="float:right;">![width](assets/img/programbugs.jpg)
</span>

--- .class9 #id9 bg:white

## Memory Corruption

<q><span style="color:red">Memory Corruption</span> happens when the <span style="color:red">contents of a memory location</span> are <span style="color:red">unintentionally modified</span> due to programming error. When the corrupted memory contents are used later in the computer program, it leads to <span style="color:red">program crash</span> or strange behaviours.</q>

<span style="float:right;">source: Wikipedia</span>

--- .class9 #id9 bg:white

## Memory Corruption

- <span style="color:red">Program crash</span> and <span style="color:red">strange behaviours</span> = busticated & exploitable.
- Responsible for many infamous vulnerabilities (Buffer Overflows/Overwrite)
- Classic implementation bug.
- Been published since the 1980's, still on going today.

Example:

```{c}
int vuln_function(char *data)
  {
        char buf[128];
        /* make copy of data to buf */
        strcpy(buf, data)
        /* ... */
        return;
  }
```

--- .class10 #id10 bg:white

## How does strcpy work ?

- A <span style="color:red">string</span> is a sequence of characters/bytes.
 * <b>char *bufs = "Hello World"</b> can be represented by 48 65 6C 6C 6F 20 57 6F 72 6C 64 <span style="color:red">00</span> 
 * <b>char bufs[14] = "Hello World"</b> can be represented by 48 65 6C 6C 6F 20 57 6F 72 6C 64 <span style="color:red">00 00 00 00</span> 
  - If you specify bufs[11], string will not be terminated.
 
- A string in C is terminated by a NULL byte 0x0 (a equivalent ascii character '\0')
- Suppose we want to copy string from bufs to a char dest[6].
- A <b>strcpy(dest, bufs)</b> performs a string copy from buf to a destination string.
 * Bufs: 48 65 6C 6C 6F 20 57 6F 72 6C 64 00
 * Dest: 00 00 00 00 00 00 - Before Copy.
 * Dest: 48 65 6C 6C 6F 20 <span style="color:red">57 6F 72 6C 64 00</span> - After Copy.
- Because <span style="color:red">strcpy is unbounded</span>, if there is more data in the bufs than there is space available, it will overwrite the memory of the server's stack. If the memory is not available, program will crash ; ).

--- .class11 #id11 bg:white

## Memory Corruption

- So <span style="color:red">unbounded copy</span> is not good.
- <b>strcpy()</b> unlikely to be seen, except in <span style="color:red">CTF</span> where you would see tons of them. x)
- Just because a better version of strcpy exists, does not mean it is not susceptible to memory corruption.
- Let's try <span style="color:red">bounded</span> <b>strncpy(char *dest, char *src, size_t n)</b>
- Now there is a <span style="color:red">parameter n</span> to limit the length of copied data.

Example:

```{c}
int vuln_function(char *data)
  {
        char buf[128];
        /* make copy of data to buf */
        strncpy(buf, data, sizeof(buf))
        /* ... */
        return;
  }
```

--- .class12 #id12 bg:white

## What can go wrong ? 

If length of <b>data</b> > <b>buf</b>: 
 - <span style="color:red">Improper-NULL termination</span> (CWE-170).

Else if length of <b>data</b> < <b>buf</b>: 
 - <span style="color:red">Improper-NULL termination</span> (CWE-170).
 - <span style="color:red">Buffer Overflow</span> (CWE-121).

The <b>strncpy()</b> does not account for NULL termination. Please use it with <span style="color:red">cautious</span>.

--- .class13 #id13 bg:white

## Auditing Case: Improper Null Termination (CWE-170)  

In this scenario, we will demonstrate how to observe strange behaviors and debug a program that has a weakness CWE 170 using the following two methods:

- Manual. [Video #0:00-#2:43 ](https://youtu.be/2WJ1BmncUpw) 
- Automatic with [CPPcheck](http://cppcheck.sourceforge.net/) embedded in SonarQube.[Video #0:2.44-#5:54](https://youtu.be/2WJ1BmncUpw?t=2m44s) 


--- .class14 #id14 bg:white
## Countering Defected Softwares
- <span style="color:red">Security</span> starts with developers.
- <span style="color:red">Prevention</span> is the best medicine.
  * Make sure take advantages of static code analysis tools.
- Constantly find efficient approaches to create a <span style="color:red">standards-compliant software</span>.

source: [Early software detection compliant](http://goo.gl/W1l7tD) - By RogueWave

--- .class15 #id15 bg:white

## Acknowledgement

All the contents are derived from [Hack-Night Project](https://github.com/isislab/Hack-Night) and some information are <span style="color:red">slightly tailored</span> to convey speaker's experiences. Thank for listening : )


Please access our free materials at:
* Github: https://github.com/LalaNguyen/Hack-Zer0s

