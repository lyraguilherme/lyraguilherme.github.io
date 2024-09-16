---
title: 'My Journey to CCIE'
date: 2024-02-24T14:07:09-03:00
draft: false
ShowToC: true
TocOpen: true
cover:
    image: /images/ccie/CCIE-66666.jpeg
    alt: 'Guilherme Lyra, CCIE #66666'
    caption: 'Guilherme Lyra, CCIE #66666'
---

## My journey

I officially started my career in IT in 2006, but my first steps with networks occurred a few years before that, between 2002 and 2003.

In 2006, I began as an intern in user support at a government agency. I started by formatting computers and quickly progressed to a point where I dealt with the entire network infrastructure including switches, firewalls, proxy, file servers, AD, etc.

After completing this internship, I went to another company, again as an intern, where I spent another 2 years before being officially hired. In total, I worked there for almost 10 years and had the opportunity to learn from exceptional professionals. While working there, I obtained the **CCNA** certification in **2008**, then the **CCDA** certification in **2013** and completed the **CCNP Routing & Switching** certification track in **2017**.

Right after finishing the last exam of the track and obtaining the CCNP R&S certification, I bought some books and started studying the content from INE and the now-defunct IPExpert. When I started studying, the exam was still the **CCIE Routing & Switching v5.0**. At some point, the exam was updated to version **CCIE Routing & Switching v5.1** and shortly after that, I changed jobs. It was February **2018** and I started working as a Technical Leader of a team composed of network and security engineers. Due to the high workload, unfortunately, I had to reduce the pace of my studies. I didn't completely stop studying, but I was already studying at a very slow pace.

Still at the same company, at the end of **2019**, I joined a newly created team that we called the Solutions Architecture team. I became responsible not only for the implementation but also for the Design of all the company's network projects. Also in 2019, I needed to renew my certifications and that's how I obtained the **Cisco CCDP** certification, which at the time was still a separate certification and later became a specialization, currently called **Cisco Certified Specialist - Enterprise Design**.

In **2020**, with the start of the pandemic and remote work, I was able to reorganize my routine in order to fully focus on studying for the **CCIE**. The exam was already in its third version since I had started studying, no longer called CCIE Routing & Switching but **CCIE Enterprise Infrastructure v1.0**.

So, in summary, I officially started studying for the **CCIE** in **2017**, after completing the **CCNP**, at a time when the exam was still the **CCIE R&S v5.0**. Due to routine issues, I slowed down my study pace and only managed to resume in **2020**. Basically, from the beginning of **2020** until December **2022** (the date I took the lab), my focus was exclusively on studying. In the next post, I will go into more detail about strategies, study hours, among others.

---

> All the information regarding the exam that you will find here is public and is available on Cisco's website.

---

## Understanding the CCIE exam

## Eligibility to the CCIE LAB

The first point to understand is that in order to schedule the CCIE LAB, you need to have passed the **350-401 ENCOR** exam within a period of **3 years**. If you are eligible, simply schedule the lab through the scheduling tool: https://ccie.cloudapps.cisco.com/CCIE/Schedule_Lab/CCIEOnline/CCIEOnline

Many people don't know but unlike exams like CCNA and CCNP, which are conducted at accredited training centers, the CCIE LAB is conducted in person at specific Cisco's testing facilities listed below:

![CCIE Lab Locations](/images/ccie/ccie_lab_locations.png)
*Source: https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/expert/ccie-lab-exam-locations.html*

Note in the table above that there are the so-called Mobile Labs. As the name suggests, these are temporary labs that occur in certain locations. At the time of writing this post (January 2024), I noticed that there are some dates when the lab will be in Brazil in April 2024. However, when checking through the scheduling tool, I did not find available slots. The downside of Mobile Labs is that slots tend to fill up quickly. You can check the dates of the Mobile Labs here: https://learningnetwork.cisco.com/s/mobile-lab-scheduler

For logistical reasons, I chose to take the lab at a fixed location and scheduled my exam at the infamous **Cisco's Building 5** in **Richardson, Texas**.

![Cisco Building 5, Richardson TX](/images/ccie/Richardson_Building_5.jpg)
*Cisco Building 5 - Richardson, Texas - December 2022*

By doing it this way, I didn't have any difficulty in finding availability for the LAB schedule. What needs to be considered here are the travel costs. Weigh all the items and decide what's most suitable for your case.

Continuing, the CCIE LAB lasts for 8 hours and is divided into 2 modules:

![CCIE Lab Modules](/images/ccie/labexam-modules.png)
*CCIE Lab Modules*

To pass, you need to achieve a minimum score in each of the modules as well as an overall minimum score (pass score). It's not enough to get the maximum score in one module and fall below the minimum in the other: in this condition, you won't pass.

The minimum score values are not disclosed. The important thing is to understand that this is the scoring format of the exam:

![CCIE Lab Modules](/images/ccie/labexam-scoreevaluation.png)
*CCIE Score Evaluation*

Another critical detail is that there is **no partial scoring**. For example: let's say in a certain task you configured everything correctly but used a different ACL name than what the question asked for. In this case, you won't get any points for that question. This is one of the factors that makes the exam so difficult: attention to details is essential.

### Module 1: Design (3 hours)

The objective of this module is to measure the ability to create, analyze, validate, and optimize network designs, which is the foundation for all deployment activities. Candidates will need to:

* Understand the capabilities of different technologies, solutions, and services.
* Translate customer requirements into solutions.
* Assess the readiness of the infrastructure to support the proposed solutions.

The module is scenario-based, with no access to any devices. Candidates will receive a set of documentation to review before answering the questions.

Examples of documentation include email threads, high-level designs, network topology diagrams, customer requirements and constraints, etc. Examples of questions include drag and drop, single-choice multiple-answer, multiple-choice, etc. During this module, backward navigation will be disabled. Therefore, candidates will not have full visibility of all questions in this module. The point values associated with each item are not displayed in this module.


### Module 2: Deploy, Operate and Optimize (5 hours)

In this module, candidates will deploy, operate, and optimize network technologies and solutions.

#### Deploy ####
Candidates will build the network according to project specifications, customer requirements, and constraints. All steps necessary for a successful network implementation will be covered, including configuration, integration, and troubleshooting in the commissioning of technologies and solutions, as per the exam topics.

#### Operate and Optimize ####
Candidates will operate and optimize network technologies and solutions. This includes monitoring network health and performance, configuring the network to improve service quality, reduce outages, mitigate disruptions, lower operational costs, and maintain high availability, reliability, and security, as well as diagnosing potential issues and adjusting configurations to align with business objectives and/or technical requirements. This module provides a setup very close to a real production network environment and will consist of practical items (device access) as well as web-based items. Whenever possible, a virtualized lab environment will be used in this module.

During this module, backward navigation will be enabled. Candidates will have full visibility of all questions in this module. Point values associated with each item are displayed in this module.

*Source: https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format*


--- 

## Study Tips

### First Steps

Start by downloading the **CCIE Enterprise Infrastructure Learning Matrix** spreadsheet from the link below. Created and maintained by Cisco, this spreadsheet lists all exam topics with recommended study materials.

> [Cisco CCIE Enterprise Learning Matrix](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/expert/ccie-enterprise-infrastructure.html#~exams)

The first column in the spreadsheet, labeled **"Level"**, is for rating your knowledge from 1 to 5 on each topic. Although it comes pre-filled, adjust it honestly to reflect your actual knowledge. This way, you'll know where to focus your studies. Update it regularly as you progress.

### General Tips

1. **Master English**  
   If English isn't your native language, prioritize learning it. Understanding all details and avoiding exam traps is critical.

2. **Avoid Mindless Study**  
   Studying hours without strategy or focus is counterproductive. Develop a plan and use the **Learning Matrix** to find weak spots.

3. **Start Small**  
   Begin with a few hours of study each day, gradually increasing your time. Burnout is a real risk if you push too hard at the start.

4. **Focus on Specific Topics**  
   At the beginning, create simple topologies. Avoid working with large topologies that use multiple protocols until you have a solid understanding of each individually.

5. **Selective Reading**  
   Don’t try to read books from cover to cover. Instead, focus on individual technologies and protocols.

6. **Embrace Repetition**  
   Re-reading topics is normal. Accept that learning requires revisiting concepts multiple times.

7. **Hands-on Configuration**  
   Refer back to theory while configuring networks. This approach helps solidify learning.

8. **Analyze Packet Captures**  
   Learn how commands affect protocols by analyzing packet captures. Examine layers in depth and understand how configurations alter specific bits or attributes.

9. **Study Design**  
   Explore different design methods. There are many ways to reach the same result—understand the pros and cons of each to make informed decisions.

10. **Final Study Phase**  
    Approximately three months before the lab exam, aim for at least 8 hours of daily study, focusing on configuring and troubleshooting complex topologies. Train yourself to maintain focus during long, stressful exam conditions.

### Recommended Books

Here are some books that I personally used and consider highly relevant:

- **CCIE Routing & Switching v5.0, Volume 1, 5th Edition** [Kocharians & Paluch]
- **CCIE Routing & Switching v5.0, Volume 2, 5th Edition** [Kocharians & Paluch]
- **Definitive MPLS Network Designs** [Guichard]
- **IP Multicast, Volume I** [Loveless, Blair, Durai]
- **IPv6 Fundamentals, 2nd Edition** [Graziani]
- **MPLS Fundamentals** [De Ghein]
- **Optimal Routing Design** [White, Slice, Rentana]
- **Routing TCP/IP, Volume 1, 2nd Edition** [Doyle & Carrol]
- **Routing TCP/IP, Volume 2, 2nd Edition** [Doyle]
- **Troubleshooting BGP** [Edgeworth, Jain]

### Routing & Switching

- Traditional protocols still make up a significant portion of the exam.
- I recommend INE’s older materials for Routing & Switching (CCIE R&S v5.0), including both videos and workbooks. They provide unbeatable content.
- I also used EVE-NG and Cisco CML for labs.
- Make sure to study using the software versions that will be used in the exam. You can find this information in the **CCIE Enterprise Infrastructure (v1.1) Equipment and Software List** on Cisco's website: [Cisco CCIE Exam Format and Equipment](https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format)

### SD-WAN

- Start with Cisco's **Configuration Guides** and **Design Guides**.
- Cisco Live presentations are another valuable resource (available on **ciscolive.com**).
- For lab practice, if you have the hardware, you can use VMware to virtualize SD-WAN components. If not, consider renting lab sessions from Cisco's **CCIE Enterprise Infrastructure Practice Labs**: [Cisco CCIE Practice Labs](https://learningnetwork.cisco.com/s/article/ccie-enterprise-infrastructure-practice-labs)

### DNA Center / SD-Access / ISE

- If you lack access to these in production environments, you’ll need lab sessions using Cisco's **CCIE Practice Labs**. These can be scheduled in advance and usually last four hours per session. It’s a worthwhile investment.

### Automation

- **Python:** Use a Linux machine (physical or virtual) to connect to routers running IOS XE. Start with basic Python skills if you’re new to the language.
- **EEM Applets and Guestshell:** Practice on virtual routers like CSR 1000v or Catalyst 8000v.
- **APIs (vManage and DNA Center):** Utilize **Cisco DevNet Sandbox** for free access, but for full lab control, schedule sessions in the **CCIE Practice Labs**.
