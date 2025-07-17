---
layout: post
title: 'OMSCS Review #1: CS6200 Graduate Introduction to Operating Systems'
date: 2025-07-16 22:03 -0400
comments: true
categories: [OMSCS, Class Reviews]
tags: [OMSCS, GIOS, Gatech]
---

To kick off my OMSCS journey in the Spring 2025 semester, I decided to start with Graduate Introduction to Operating Systems

## <span style="color:red">Motivations</span>

From what I heard about this class from other online reviews, it appeared that this class was going to be a good introduction to the workload and academic rigor that is going to be expected in later courses in the program. Given that the Computing Systems specialization is what I'm going for, I figured that my prior experience with Linux, C and C++ will help me in the projects. Despite this, the projects were no easy task and they did take a significant amount of time spanning multiple weeks to complete.

## <span style="color:red">Grading</span> 

The grading scale for the Spring 2025 semester was as follows:

- **Project 1**: 15%
- **Project 3**: 15%
- **Project 4**: 15%
- **Midterm**: 25%
- **Final**: 25%

Notice how we skipped Project 2? Apparently it used to be offered in earlier iterations of the class as an extra credit assignment, but it seems like they removed it now. They still kept the project names the same just by convention, though.

The midterm came after the first project and covered the first half of the material taught in the class. The final was after Project 4. Although I ended up with a 62 on the final, which was below the average of 68.12, I had at least 100% on all the projects and was in the high 80s on the midterm, so I could have completely bombed the final and still came out with an A. Knowing this, I didn't waste too much time on preparing for the final and basically just winged it after skimming my notes

> Note: From an office hour session, it appears that after the curve, the A & B cutoff was 80-85, and the B & C cutoff was 59-65
{: .prompt-info}

In terms of averages, here's the class averages on all the graded material:

| Assignment | Average | Median  | STD Dev  |
| --------   | ------- | ------- |  ------- |
| Project 1  | 82.77   | 96.5    | 27.61    |
| Project 3  | 86.92   | 98      | 23.17    |
| Project 4  | 92.68   | 106     | 29.92    |
| Midterm    | 75.8    | 77      |          |
| Final      | 68.12   | 72      |          |

While the assignments were challenging, the class had a very active Slack and Piazza where the TAs and other students can provide help and guidance to each other so long as they are only discussing high-level ideas, which was a tremendous help. 

## <span style="color:red">Lectures</span>

The class features very high quality recorded lecture videos. Each lecture goes into depth about the topic discussed and provides visual examples to explane arcane concepts in operating system design using the metaphor of a "Toy Shop". 

Beyond the lectures, there were also practice exams for both the miderm and the final which featured questions similar to what would be asked on the exam, which was very useful for studying.

## <span style="color:red">Projects</span>

Each project ranges in difficulty, and each project was difficult to implement for a different reason. Along with each project, we are supposed to submit a writeup that is worth 10% of the project grade explaining the choices that were made, the high-level control flow of your code, and any code/resources you referenced on the internet

 I'll try to succinctly explain my experience with each project and how I rank its difficulty.

### <span style="color:lightcoral">Project 1: Getfile Protocol</span>

This project is split up into two parts. Essentially, we are implementing a bespoke transfer protocol called "GETFILE" which operates similar to HTTP GET requests. We construct a client-side library for sending these GETFILE requests and a server-side library that is used to serve these requests as they come in. In part 2, we extend our implementation by making our server and client multithreaded.

This project is often the most difficult for most students, as it would seem from the average scores. The general consensus appears to be that your Project 1 scores and midterm scores should be evaluated to determine whether or not you should drop the class, and it certainly is a difficult project. Personally, however, I found it fairly straightforward. The difficulty of the project essentially boiled down to familiarity with C and how to extract tokens using string parsing methods to validate requests. 

> If you come in to this class without knowing C and you aren't an extremely fast learner, you are beyond cooked.
{: .prompt-danger}

The second part was different in that you are now having to worry about the complexities of multithreading and ensuring mutual exclusion instead of having to worry about low-level string parsing. This part actually ended up being quite easy when I decided to wrap mutually exclusive inserts/access into common functions, which allowed me to reuse the same code in Project 3 as well.

**My final score on this project: 100%**

### <span style="color:lightcoral">Project 3: Interprocess Communication</span>

In this project we convert our web server into a proxy server (Part 1) that translates GETFILE requests into HTTP requests for another server on the internet, and a cache server (Part 2) that communicates with a proxy server via shared memory and utilizes an on-disk cache for downloaded files

This project delved into the intricaces of Linux IPC, including mechanisms such as shared memory and message queues. I decided to use POSIX message queues instead of the older System V queues just because the API seemed a bit more straightforward. This project also dealt with low level synchronization primitives such as semaphores.

Personally, I found this project harder than Project 1. Slack was my best friend for this project, as students posted diagrams and control-flow graphs that provided a clearer understanding of what needed to be done. I often ran into deadlocks attempting to implement a version of the reader/writer problem using semaphores, and it took me a painstakingly long time to figure out why as there wasn't really a solid way to debug the issue. But I ended up figuring it out and passing all the GradeScope tests

**My final grade: 100%**

### <span style="color:lightcoral">Project 4</span>

This final project was in C++ rather than C, and it had us implementing an RPC protocol service that will fetch, store, list, and get attributes for files on a remote server using gRPC and protobufs. In Part 2, we flesh out our previous RPC implementations into a full-blown rudimentary distributed file system (DFS), where we need to ensure consistency among all nodes in the file system and implement whole-file caching and a simple lock strategy on the server-side to manage client writes.

Many people say that this is the easiest project, and comparitively speaking I reluctantly agree. But that is not to say this project is "easy" by any stretch of the imagination. It still requires a decent amount of time to complete. Further, the project's complexity is increased by the fact that a large portion of my time was just spent sifting through the boilerplate code that was provided for this part to understand how it works and what exactly I needed to add, which isn't necessarily clear on the first glance. There ended up being extra credit tests on GradeScope for this part involving synchronization between multiple clients, and it isn't really clear on GradeScope which tests are extra credit and which aren't. 

**Final score: 110%**

## <span style="color:red">Final Thoughts</span>

This class was extremely challenging and demanded a lot of my time outside of work. Many weekends were spent simply in front of my laptop for hours trying to get my code to work. However, despite the difficult at the beginning, it seems to ease off near the end as you get more familiar with the format of the assignments and the requirements.

I can confidently say that this class delivered on its promises. I learned a lot about operating systems in far more depth than I ever covered in undergrad. This class, in a way, forces you to become a better programmer through the complexity of the assignments and teaches you valuable skills that will undoubtedly help you in a career in computer science.

My advice is to start early on the assignments and ask questions on Slack. Chances are someone else ran into the same issue and can provide better guidance.