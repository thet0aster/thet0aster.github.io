---
layout: post
title: 'OMSCS Review #2: CS6290 High Performance Computer Architecture'
categories:
- OMSCS
- Class Reviews
tags:
- OMSCS
- HPCA
- Gatech
image:
  path: https://cloudfront-us-east-1.images.arcpublishing.com/ajc/CPLKGQDZU3MT46H6NGLNYEEIMU.jpg
  alt: Georgia Institute of Technology
date: 2025-12-13 21:37 -0500
---
In the Fall 2025 semester of my OMSCS journey, I decided to take High Performance Computer Architecture, where we delved into a variety of topics relating to computer organization and architecture

## <span style="color:red">Motivations</span>

In the prior Summer 2025 semester, I finished taking IIS (Introduction to Information Security), which was an entirely project-based course and I learned a lot of practical information on various aspects of computer security. After taking GIOS the previous semester, I had a choice to continue in the fall semester and jump directly to AOS (Advanced Operating Systems), since it appears that AOS picks up from where GIOS left off. However, I decided to go with HPCA as I read reviews that the class was beneficial in getting a more comprehensive understanding of the underlying hardware implementations of things like memory and paging which would be valuable for AOS. So I decided to take HPCA this semester and have registered to take AOS in the coming spring semester. Although I finished IIS before HPCA, I have not yet had time to write my review for IIS, so instead that post will be my third installment in this series.

## <span style="color:red">Grading</span> 

The grading scale for the class was as follows:

- **Project 0**: 5%
- **Project 1**: 10%
- **Project 2**: 15%
- **Project 3**: 20%
- **Midterm**: 20%
- **Final**: 30%

Each project ramps up in difficulty and the amount of contribution to your grade. The midterm covers all the topics from the first half of the course and the final is cumulative.

There is no guaranteed curve in terms of letter-grade cutoffs for this class, so pretty much you have to get a 90% or higher for an A and an 80% for a B, and so on. However, there might be a curve depending on the average grades of the class at the end of the semester if the class average is lower than usual or results in too few A's, but don't count on it. Historically, there hasn't been a curve for the cutoff for A and B since 2017.

I ended up with a final grade of 93.33% in the class. My project grades were 93.5%, 100%, 99%, and 99% respectively. My midterm grade was 91% and my final exam grade was 86%.

## <span style="color:red">Lectures</span>

Like GIOS, the lectures for this class are extremely high quality. Professor Prvulovic breaks down each topic in depth and intersperses the lectures with quizzes to test your knowledge on concepts that were introduced. Along with the lectures, we are provided a PDF of lecture notes which provide a concise summary of the topics discussed in each module.

Communication in the course was facilitated through EdStem, where students and TAs could communicate, ask questions and post review materials. The head TA for this course was Nolan Capehart and he provided invaluable resources to students and additional resources for studying on Ed, which was very helpful. Beyond this, there are also practice problems and sample exams available on Canvas to aid in studying for exams.

## <span style="color:red">Projects</span>

As mentioned previouly, there are 4 projects in this course that ramp up in difficulty. All the projects involve a software known as SESC, or the Superscaler Simulator, which is an open source multiprocessor simulator. The class provided an Ubuntu VM with the software set up or other installation methods such as a Docker container were also provided.

The projects were fairly straightforward with clear instructions. They typically involved running a particular benchmark simulation in SESC with modified configurations such as cache size, cache associativity, cache eviction policy, thread count, etc, and analyzing the results. Project 0 and Project 1 were individual projects, while Projects 2 and 3 could be completed in teams of two.

### <span style="color:lightcoral">Project 0</span>

Not much to say about this project, its main purpose was to familiarize yourself with SESC and running some basic simulations. In this project, we used the "lu" simulation packaged with SESC, which is an implementation of LU decomposition running on the simulated hardware while varying the matrix size. We analyze the generated simulation reports and notice the changes in dynamic instructions executed, simulation time, and branch predictor accuracy. We were also tasked to compile a "Hello world" program on our simulated architecture and notice how the dynamic instruction count changes based on the string that is printed.

### <span style="color:lightcoral">Project 1</span>

This project involved modifying the branch predictor configuration in sesc with the raytrace benchmark, seeing how the predictor accuracy changed by using a not-taken, hybrid, or an oracle predictor, as well as how pipeline depth affects the simulation's latency using the respective predictor. The final part of this project involved making changes to SESC's source code to stratify branches into fixed size buckets of the number of times that branch was completed and the predictor accuracy of each bucket. This involved making some minor code changes to classify predictions and mispredictions at a branch and counting the number of completions.

### <span style="color:lightcoral">Project 2</span>

This project was the first project that we could complete in a group of two. Luckily, there was a post on Ed that made it easy to connect with other students in the class and reach out to them to form groups for Projects 2 and 3, and I was able to reach out to another student in the class to work on the projects together. This project specifically involved analyzing the performance of caching on an out-of-order processor and how cache parameters such as size and associativity affect outcomes like miss rate and cycle count. The second part of this project involved programming a new replacement policy, NXLRU, to replace the default LRU replacement policy in SESC. This policy chooses the second least recently used line in LRU order to replace in the cache on a miss. Next we were instructed to add code to stratify the types of misses into conflict, capacity, and compulsory misses running the FMM simulation

For this project, we are also provided a shared spreadsheet for students to check that their implementations were correct. Students could run another chosen simulation, such as LU, on their code and record their results in the shared spreadsheet. This way, you can know if your implementation was correct if you are getting similar results using the chosen shared simulation as the other class without sharing the FMM simulation results which is what we are submitting for a grade. This was helpful in double checking that my code was working as intended

### <span style="color:lightcoral">Project 3</span>

Project 3 picks up from where Project 2 leaves off and explores multi-core behavior and cache coherence. Like with Project 2, we can work on this with a partner and we are provided a shared spreadsheet to check your implementation against the class's results. We once again are classifying misses like in Project 2, but are now concerned with compulsory, replacement, and coherence misses (misses that are a result of coherence traffic generated due to multiple cores/caches). This project was the hardest of all three, and it took me and my partner a week to finish all parts and get the correct implementation that gave us matching shared results compared to the rest of the class.

### <span style="color:lightcoral">Exams</span>

As previously metioned, there is a midterm and a final exam for this class weighted at 20% and 30% respectively. All the exams were open note/closed internet. The midterm covered topics from the first half of the course, ending at VLIW, and the final covered mainly the second half, but was also cumulative. The exams covered a wide variety of topics discussed in the lectures, from Tomasulo's algorithm, Reorder Buffer, branch prediction and pipelining. The final exam delved into more advanced concepts on caches such as cache coherence involving the MSI protocol and its variations (MESI, MOSI, MOESI) and snooping or directory-based coherence, as well as fault tolerance, storage systems, memory consistency and multi-core performance considerations. The exams were comprehensive but fair. Doing well on the exams is crucial as they comprise 50% of your total grade in the course. The midterm was calculation heavy and was on a time crunch, but I was able to finish the entire exam with a minute left on the clock. The final exam, while cumulative, seemed easier than the midterm and focuses more on conceptual understanding rather than on calculations. We are also given 3 hours for the final and are not constrained by time, and I was able to finish the exam with an hour to spare.

## <span style="color:red">Final Thoughts</span>

This course definitely had its moments and demanded a fair bit of time outside of work in order to complete. The resources provided such as the sample exams and the material posted by the TAs on Ed were very useful in preparing for the exams, although I feel the projects weren't the greatest compared to previous classes.

While the class definitely delivered on providing a comprehensive understanding of computer architecture, you are basically in the dark regarding your grade for the majority of the class. All of the projects but one were left without a grade posted in canvas until the very final couple weeks of the course, so that is something to keep in mind especially if you take the midterm and are unsure if you need to drop the class or not. Beyond this, there were periods of time where TAs wouldn't respond to any posts on Ed and many students expressed frustration about being kept in the dark regarding grades or not responding to questions posted

My advice is to take good notes for the exams and print them out. Also printing out all the quizzes and having them to refer to during the exams was also very helpful