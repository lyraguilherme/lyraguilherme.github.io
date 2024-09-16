---
title: 'My Journey to CCIE'
date: 2024-02-24T14:07:09-03:00
draft: false
ShowToC: true
TocOpen: true
sitemap:
  changeFreq: daily
cover:
    image: /images/ccie/CCIE-66666.jpeg
    alt: 'Guilherme Lyra, CCIE #66666'
    caption: 'Guilherme Lyra, CCIE #66666'
---

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

## Module 1: Design (3 hours)

The objective of this module is to measure the ability to create, analyze, validate, and optimize network designs, which is the foundation for all deployment activities. Candidates will need to:

* Understand the capabilities of different technologies, solutions, and services.
* Translate customer requirements into solutions.
* Assess the readiness of the infrastructure to support the proposed solutions.

The module is scenario-based, with no access to any devices. Candidates will receive a set of documentation to review before answering the questions.

Examples of documentation include email threads, high-level designs, network topology diagrams, customer requirements and constraints, etc. Examples of questions include drag and drop, single-choice multiple-answer, multiple-choice, etc. During this module, backward navigation will be disabled. Therefore, candidates will not have full visibility of all questions in this module. The point values associated with each item are not displayed in this module.


## Module 2: Deploy, Operate and Optimize (5 hours)

In this module, candidates will deploy, operate, and optimize network technologies and solutions.

### Deploy ###
Candidates will build the network according to project specifications, customer requirements, and constraints. All steps necessary for a successful network implementation will be covered, including configuration, integration, and troubleshooting in the commissioning of technologies and solutions, as per the exam topics.

### Operate and Optimize ###
Candidates will operate and optimize network technologies and solutions. This includes monitoring network health and performance, configuring the network to improve service quality, reduce outages, mitigate disruptions, lower operational costs, and maintain high availability, reliability, and security, as well as diagnosing potential issues and adjusting configurations to align with business objectives and/or technical requirements. This module provides a setup very close to a real production network environment and will consist of practical items (device access) as well as web-based items. Whenever possible, a virtualized lab environment will be used in this module.

During this module, backward navigation will be enabled. Candidates will have full visibility of all questions in this module. Point values associated with each item are displayed in this module.

*Source: https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format*


--- 

## First steps ##

Download the CCIE Enterprise Infrastructure Learning Matrix spreadsheet from the link below. This spreadsheet was created and is maintained by Cisco itself. In it, you will find all the topics of the exam with some study materials recommendations.

> On the following link, at the bottom of the page, under Resources, there is a link to the CCIE Enterprise Infrastructure Learning Matrix:
 https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/expert/ccie-enterprise-infrastructure.html#~exams

Note that the first column of the spreadsheet (called "Level") is precisely for you to assign a rating from 1 to 5 to your knowledge in each of these topics. The spreadsheet comes pre-filled, but change it according to your level of knowledge. Be honest, as this is how you will know where you need to focus more or less on your studies. Periodically review this spreadsheet and update your level as you progress.

## General Tips ##

1. If English isn't your native language, make sure to learn it. You must be able to interpret all possible details of what is being asked, identify points of attention, traps, etc.

2. Don't believe in ready-made recipes. Studying dozens of hours a day without having a solid strategy and without knowing where you need to focus will be a waste of time.

3. The CCIE Enterprise Infrastructure Learning Matrix spreadsheet will be your greatest ally in this journey. Use it to find your weak points and know which content you need to study more or less. Go back and read what I wrote in the "first steps" item.

4. Start by studying a few hours a day and gradually increase your study time. Making a huge effort at the beginning and giving up halfway through the journey won't help. Read item 4 again.

5. At the beginning of your studies, focus on specific topics and create simple topologies to study. Avoid starting the journey by working with a very large topology. What's the point of working with 2 or 3 protocols at the same time if you don't understand any of them deeply? You'll need that in the final phase when you're close to scheduling the lab, but there's a long way to go until then.

6. While studying theory, you don't need to take the book and read from start to finish. By doing that, you'll probably forget most of the content. Focus on individual technologies/protocols.

7. Don't get frustrated when you need to read and reread the same subject countless times. This is part of the learning process.

8. I particularly like to consult the theory while I'm doing a specific configuration. Digital books help a lot with this.

9. Understand how a specific command or functionality will reflect at the protocol level. Make a habit of analyzing packet captures.

10. Still on packet captures, analyze all layers in depth. Understand what changes when performing or modifying a specific configuration, which bits or attributes are changed, how the information is propagated, etc.

11. Look for information on a particular subject in various different sources. When studying a specific topic, review Cisco's documentation and also books from different authors. Read item 10 again.

12. Study network design. There are usually several paths to achieve the same result. Why implement something in one way and not another? What will be the differences? Which one best meets your goal?

13. In the final stretch of studies for the Lab, I consider that approximately 3 months before your lab date, at this point, you should be able to study at least 8 hours a day. At this stage, you should focus on configuring and troubleshooting complex topologies. Remember that the exam will last 8 hours. Create a strategy to maintain concentration and focus under stressful and tired conditions.


## Books ##

Some of the books I've used and consider quite relevant:

* CCIE Routing & Switching v5.0, Volume 1, 5th Edition [Kocharians & Paluch]
* CCIE Routing & Switching v5.0, Volume 2, 5th Edition [Kocharians & Paluch]
* Definitive MPLS Network Designs [Guichard]
* IP Multicast, Volume I [Loveless, Blair, Durai]
* IPv6 Fundamentals, 2nd Edition [Graziani]
* MPLS Fundamentals [De Ghein]
* Optimal Routing Design [White, Slice, Rentana]
* Routing TCP/IP, Volume 1, 2nd Edition [Doyle & Carrol]
* Routing TCP/IP, Volume 2, 2nd Edition [Doyle]
* Troubleshooting BGP [Edgeworth, Jain]

## Routing & Switching ##

* Remember that traditional protocols still account for a large part of the exam.
* For my R&S studies, I used the old materials from INE, both the videos and the workbook from the old version of the exam (CCIE R&S v5.0). In my opinion, their content is unbeatable for the R&S part. I still use the workbook topologies today to study or simulate something.
* For R&S, I also used old materials from the now-defunct IPExpert.
* For R&S labs, I used EVE-NG and Cisco CML.
* Try to study using the same software versions that will be used in the exam. This information can be found here:
 https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format --> in the Equipment and Software list, you will find a PDF with the images and respective versions used in the exam: CCIE Enterprise Infrastructure (v1.1) Equipment and Software List.
* Do not ask me to send any images to you. If you do not have access to the images, you can use them legally through Cisco CML.


## SD-WAN ##

* Start with the Configuration Guides and Design Guides from Cisco.
* There is a lot of content in Cisco presentations available at ciscolive.com.
* I usually recommend studying all the material in English, but if speak Portuguese and are starting to study about SD-WAN, I recommend watching this presentation I did at the invitation of the Cisco Community: https://www.youtube.com/watch?v=xtTHjDv1r-M
* Review the topics covered in the CCIE Enterprise Infrastructure Learning Matrix carefully. Don't waste time studying features that won't be tested in the lab.
* I recommend the content from INE (updated for CCIE Enterprise v1.0). Within the INE platform, after completing a certain module, there are usually some labs they provide to reinforce the content.
* For labs, if you have available hardware, you can use VMware to virtualize SD-WAN components. Remember to use the same software versions that are tested in the exam. This information can be found here: https://learningnetwork.cisco.com/s/article/ccie-practical-exam-format --> in the Equipment and Software list, you will find a PDF with the images and respective versions used in the exam: https://learningcontent.cisco.com/documents/marketing/exam-topics/CCIEEIv1.1-equipment_and_SW.pdf
The issue with running SD-WAN running "on-premises" is that you will need a robust machine. If you don't have one at home or work, seek an alternative to rent a laboratory online that has these images and sufficient resources. An option is to rent some sessions in Cisco's own environment, called **CCIE Enterprise Infrastructure Practice Labs**: https://learningnetwork.cisco.com/s/article/ccie-enterprise-infrastructure-practice-labs

## DNA Center, SD-Access e ISE ##

* In my case, I already had experience with these solutions in production environments. I was responsible for implementing SD-Access projects, and that helped a lot during the exam.
* For labs, there's no way around it: you'll need access to a DNA Center and Catalyst 9000 series switches. In my case, I used several sessions in the CCIE Practice Labs (CCIE Enterprise Infrastructure Practice Labs). Lab access needs to be scheduled in advance. Additionally, the fee is charged in dollars and if I'm not mistaken, there's a limit of 4 hours per session. I spent a lot of money here, but it's definitely worth it.

## Automation ##

* **Python:** a machine running Linux (can be a VM) and with connectivity to a topology with routers running IOS XE. If you have no knowledge of Python, start with the basics before interacting with equipment.
* **EEM Applets and Guestshell:** I suggest using a virtual router running IOS XE. It can be CSR 1000v or Catalyst 8000v.
* **vManage and DNA Center APIs:** 
    * **vManage:** You can virtualize it "in-house" or use a lab from the Cisco **DevNet Sandbox**.
    * **DNA Center:** Specifically regarding APIs, I recommend using a Cisco DevNet Sandbox, where you can use the tool without having to pay.
    * The **DevNet Sandbox** labs are usually read-only, so if you want to manipulate any configuration, I suggest scheduling a session in the **CCIE Enterprise Infrastructure Practice Labs**: https://learningnetwork.cisco.com/s/article/ccie-enterprise-infrastructure-practice-labs