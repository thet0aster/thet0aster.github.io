---
layout: post
title: 'OMSCS Review #3: CS6035 Introduction to Information Security'
categories:
- OMSCS
- Class Reviews
tags:
- OMSCS
- IIS
- Gatech
image:
  path: https://cloudfront-us-east-1.images.arcpublishing.com/ajc/CPLKGQDZU3MT46H6NGLNYEEIMU.jpg
  alt: Georgia Institute of Technology
date: 2025-12-29 22:08 -0500
---
In the Summer 2025 semester of my OMSCS journey, I decided to take IIS. This is a survey class covering various topics in computer security

## <span style="color:red">Grading</span> 

The grading scale for the class was as follows:

| Assignment          | Release Date | Due Date | Weight |
|---------------------|---------------|:--------:|:------:|
|  Man in the Middle  |     Jan 9     |  Jan 18  |   10%  |
|   Machine Learning  |     Jan 17    |   Feb 1  |   12%  |
| Binary Exploitation |     Jan 31    |  Feb 15  |   14%  |
|   RSA Cryptography  |     Feb 14    |   Mar 1  |   14%  |
|     API Security    |     Feb 28    |   Mar 8  |   8%   |
|     Web Security    |     Mar 7     |  Mar 15  |   8%   |
|      Log4Shell      |     Mar 14    |  Mar 29  |   12%  |
|  Database Security  |     Mar 28    |  Apr 12  |   12%  |
|   Malware Analysis  |     Apr 11    |  Apr 19  |   10%  |

In addition, we ended up having the following extra credit opportunities:

| Assignment                       | Weight |
|----------------------------------|:------:|
|         Extra Credit Exam        |   2%   |
|   Malware Analysis Extra Credit  |   2%   |
| Binary Exploitation Extra Credit |   2%   |

The final class grade distribution was as follows:

|  Statistics:  | Man in The Middle | Machine Learning | Binary Exploitation | API Security | Web Security | Log4Shell | Malware Analysis | Cryptography | Database Security |
|:-------------:|:-----------------:|:----------------:|:-------------------:|:------------:|:------------:|:---------:|:----------------:|:------------:|:-----------------:|
|       A       |        72%        |        69%       |         38%         |      92%     |      79%     |    80%    |        66%       |      81%     |        25%        |
|       B       |         8%        |        8%        |          7%         |      3%      |      7%      |    13%    |        29%       |      8%      |        15%        |
|       C       |         3%        |        9%        |          8%         |      0%      |      1%      |     0%    |        2%        |      5%      |        11%        |
|       D       |         3%        |        2%        |          6%         |      0%      |      1%      |     2%    |        1%        |      1%      |        21%        |
|       F       |         8%        |        6%        |         33%         |      3%      |      4%      |     3%    |        0%        |      2%      |        24%        |
| No Submission |         6%        |        6%        |          8%         |      2%      |      8%      |     2%    |        2%        |      3%      |         4%        |

## <span style="color:red">Projects</span> 
We had 9 projects in this class, with a week to complete each project as it was assigned. The condensed summer timeline with only 1 week between projects made this class quite stressful, unlike what other reviews of this class seemed to portray.

### <span style="color:lightcoral">Man in the Middle</span>
This is the first project of the class and involved using Wireshark to perform a MITM attack. This project was fairly simple, but had some parts that were a bit tricky and took a couple hours to figure out. I ended up with 100% on this project

### <span style="color:lightcoral">Machine Learning</span>
This project is an application of machine learning to cybersecurity. You will learn about numerous concepts in data science and machine learning, including random forests and k-means clustering. You will also be given the CLAMP dataset to run some features and try to tune some hyperparameters to achieve a certain level of performance, with the overall goal being to demonstrate that machine learning techiques can be used to detect malware.

### <span style="color:lightcoral">Binary Exploitation</span>
This project was the most exciting project in my opinion, as I have extensive background in this topic, but for others in the class it was the hardest module of them all (as you can clearly see by the grade distribution). This project operates as a typical CTF for binary exploitation using pwntools and gdd. You will learn about static and dynamic analysis, buffer overflows, ROP chains, etc.

This project took me an entire weekend, but your mileage may vary. Many others spent days trying to attempt to get all the flags on this project and were unsuccessful. There was also extra credit flags for this project, but I didn't end up completing any of them. Overall grade for this was 100%

### <span style="color:lightcoral">API Security</span>

This project involves attempting to break into weak API endpoints and trying to steal files and credentials. You will learn how to use curl, postman, JWT tokens, and the Swagger API. Overall grade was 100%

### <span style="color:lightcoral">Web Security</span>
This project involved web exploitation, where you attempt to inject JavaScript/HTML payloads via XSS, CSRF, etc into a custom website to attempt to gain unauthorized access.


### <span style="color:lightcoral">Log4Shell</span>
This was also an extremely interesting project as it is based on Log4Shell, a vulnerability in Log4J that allows for arbitrary code execution via rerouting requests to arbitrary LDAP and JNDI servrs, which caused a massive response globally across countries, governments, and businesses to patch affected systems in 2021. Overall grade is 100%.

### <span style="color:lightcoral">Malware Analysis</span>
This project involved analyzing reports from JOE Sandbox about historical malwares and finding out from these reports how to classify the type of malware based on the behavior they exhibit. This project was relatively straightforward. I got 44/50 on phase I and 50/50 on phase 2. This project also had some extra credit, which I ended up doing and getting some points.

### <span style="color:lightcoral">Cryptography</span>
Involved with RSA encryption system and gives you a good exposure to cryptographic theory and number theory. This project is relatively straightforward if you have a math background. 100%.

### <span style="color:lightcoral">Database Security</span>
This project involved SQL injection and stuff. I don't really remember, because I ended up winging it. I realized I would have an A in the class regardless of whether I do this project or not, so I got some basic flags after spending 5 minutes looking at the project and just submitted what I had. Final grade for this was 27.5%