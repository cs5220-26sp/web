---
title:      Syllabus
layout:     main
---

## Logistics

CS 5220, {{ site.data.logistics.semester }}: Applied High-Performance and Parallel Computing

Lecture Time: {{ site.data.logistics.time }}  
Lecture Location: {{ site.data.logistics.location }}

{% assign prof = site.data.logistics.prof -%}
Prof: [{{ prof.name }}]({{ prof.url }})  
E-mail: <{{ prof.email }}>  
{% if prof.office %}Office: {{ prof.office }}  {% endif %}
Office hours: 
{% for oh in prof.oh -%}
  {%- if oh.url -%}
    [{{ oh.when }}]({{ oh.url }})
  {%- else -%}
    {{ oh.when }}
  {%- endif -%}
  {%- if oh.where -%}, {{ oh.where }}{%- endif -%}
  {%- unless forloop.last %}; {%endunless %}
{% endfor %}

{% for ta in site.data.logistics.staff %}
TA: {% if ta.url %}[{{ ta.name }}]({{ ta.url }}){% else %}{{ ta.name }}{% endif %}  
Email: <{{ ta.email }}>  
Office hours: 
{% for oh in ta.oh -%}
  {%- if oh.url -%}
    [{{ oh.when }}]({{ oh.url }})
  {%- else -%}
    {{ oh.when }}
  {%- endif -%}
  {%- if oh.where -%}, {{ oh.where }}{%- endif -%}
  {%- unless forloop.last %}; {%endunless %}
{% endfor %}

{% endfor %}

### Welcome to CS 5220!

I'm excited to kick off another semester of CS 5220! 

The first two weeks of lectures provide the foundation for the course, covering performance basics and an overview of computer architecture and memory hierarchy. If you are not yet enrolled but plan to be, you are welcome to attend as long as seating is available.

Late enrollment after the first three weeks may be possible, but please note that deadlines, late penalties, and slip days are **not** affected by late enrollment. For example, if you enroll late and miss the HW1 deadline, **no extension will be granted**; however, according to course policy, the **lowest coding homework score will be dropped** at the end of the semester, so missing HW1 will not automatically affect your grade.

Please read the syllabus carefully to familiarize yourself with course policies and expectations. If you have any questions, feel free to post on [EdDiscussion][Ed], either publicly or privately. I look forward to a great semester and seeing all of you in class!

## Course Description

**Please note that the syllabus will continue to be updated until the beginning of the classes.**

CS 5220 is an introduction to performance tuning and parallelization, particularly for scientific codes. Topics include:

- Single-processor architecture, caches, and serial performance tuning
- Basics of parallel machine organization
- Distributed memory programming with MPI
- Shared memory programming with OpenMP
- GPU programming with CUDA
- Cerebras programming with CSL
- Parallel patterns: data partitioning, synchronization, and load balancing
- Parallel numerical algorithms
- Parallel graph algorithms
- Parallel algorithms in science and engineering

Students should be able to read and write serial programs written in C++ or related language and be familiar with computer architecture (e.g., CS3410 at Cornell). Because our examples will be drawn primarily from engineering and scientific computations, some prior exposure to numerical methods is useful, though not necessary.

Prior exposure to parallel programming is not required, and non-CS students from fields involving simulation and data-intensive applications are particularly welcome!

### Course work

The course work consists of:

- *Lecture*: Participation is expected! The slides will be
  posted before class and updated afterward as needed (e.g., to include notes and examples from class).
- *Homework*: Coding homework assignments require students to parallelize code, tune performance, and conduct performance experiments and analysis using different programming models. Each assignment is expected to take more than one day of work; students are strongly encouraged not to start the day before the deadline. You will have two weeks for each coding assignment. Homework is completed individually (see Collaboration below). The assignments are typically posted on Thursday and due the following Thursday.
- *Final Projects*: In addition to the assigned projects, students will complete a significant final project of their choice. We will suggest final projects, or students may propose their own. Group projects are encouraged. Final project proposals are due before spring break, and groups are expected to make steady progress in April and early May. The last two lectures are reserved for project presentations, and the final project report is due during the finals period instead of a final exam.

### Text

The class will be taught using lecture slides, which will be posted as the class progresses.

### Course Technology

This spring, we will distribute code via Git repositories and recommend using Git for code development and submissions, though this is not strictly required.  The assignment submissions and grading will generally be managed through Canvas. For discussion, we will use [Ed Discussion][Ed] which you can also access through the course Canvas site.

<!-- canvas:  -->

## Course policies

### Grading

Graded work will be weighted as follows:

- Class participation: 10%
- 6 homework: 60% (The lowest grade amongst the coding assignments will be dropped)
- Project: 30%

Your participation makes the class more interesting and useful
for all of us. There are several ways to participate:

- Coming to class
- Actively engaging and answering questions on [Ed Discussion][Ed]
- Helping correct and clarify class notes and assignments

It is also important for us to receive your feedback on how the class is going, both to improve the class this semester and to enhance it for future semesters. I will request non-anonymous comments around the midterm, and at the end of the semester, we will check with the college to see who has completed the course evaluation surveys (though we cannot determine whether your feedback is useful). Participation in these feedback activities counts toward your grade.

### Slip Days and Late Policy

You may use up to **9 slip days total over the semester**, with **at most 3 slip days applied to any single assignment**. For each day an assignment is submitted past its due date (and after any available slip days are used), a **late penalty of 1.5% of the assignment’s total value per day** will be applied. **NO credit will be given for submissions more than 7 days past the due date**, regardless of available slip days.

### Exception (Strict Deadlines)

The late policy and slip days **do not apply** to the **Final Project Poster** and **Final Project Report**. Deadlines for these deliverables are **strict**. 

### Extension Policy

You will have two weeks to complete each coding assignment and more than a month to complete the final project. **NO extensions will be granted** for reasons such as midterms, conference or journal paper deadlines, conference travel, similar professional obligations, or minor illness. You should plan accordingly and use slip days when necessary. Emails requesting extensions for these or similar reasons will be ignored.

If you need additional accommodation, you must request it in writing and in advance, including a clear rationale and a concrete plan for when you will be able to submit the work. Granting additional accommodation is not guaranteed.

### Collaboration

An assignment is an academic document, like a journal article.
When you turn it in, you are claiming everything in it is your
original work, *unless you cite a source for it*.

You are welcome to discuss homework and projects among yourselves in
general terms. However, you should not look at code or writeups from
other students, or allow other students to see your code or writeup,
even if the general solution was worked out together. Unless we
explicitly allow it on an assignment, we will not credit code or
writeups that are shared between students (or teams, in the case of
projects).

If you get an idea from a classmate, the TA, a book or other published
source, or elsewhere, please provide an appropriate citation. This is
not only critical to maintaining academic integrity, but it is also an
important way for you to give credit to those who have helped you out.
When in doubt, cite! Code or writeups with appropriate citations will
never be considered a violation of academic integrity in this class
(though you will not receive credit for code or writeups that were
shared when you should have done them yourself).

### Use of Generative AI

You may use generative AI tools if you wish, but we ask that **you
explicitly state where and how you have used these tools**. If you do
use generative AI, you should make sure that you understand the
generated code or text. You are ultimately the responsible party for
your submissions, and incorrect or nonsensical results will be graded
the same whether they are produced by a human or an AI system.

### Perlmutter (Course Cluster)

**NERSC Perlmutter** is the primary machine on which homework performance will be graded.

Be aware that Perlmutter may be unavailable for **24–48 hours every 2–3 weeks** due to planned maintenance. You must plan accordingly and **not wait until the last minute** to start your homework. **NO extensions will be granted for planned Perlmutter outages.** You are expected to regularly check Perlmutter’s status and planned outages (see link on the course site).

For **unplanned Perlmutter outages**, the course staff will consider extensions based on  
(a) **when** the outage occurs and  
(b) **how long** it lasts.

For example:
- If an unplanned outage occurs within **3 days of the homework release** and lasts **less than 48 hours**, **no extension** will be granted.  
- If an unplanned outage occurs within **3 days of the homework due date**, an extension will be granted **proportional to the outage duration** (e.g., a 1-day extension for a 1-day outage).

The course staff receives outage notifications directly from NERSC. Therefore, **do not email the staff about unplanned outages**. We are already aware of them and will communicate any decisions regarding extensions as soon as possible. Please be patient.


### Accommodations for Students with Disabilities

Students with Disabilities: Your access to this course is important. Please request your accommodation letter early in the semester, or as soon as you become registered with SDS, so that we have adequate time to arrange your approved academic accommodations.

### Inclusivity Statement

Cornell University (as an institution) and I (as a human being and instructor of this course) are committed to full inclusion in education for all persons. Services and reasonable accommodations are available to individuals with temporary or permanent disabilities, to students with DACA or undocumented status, to students facing mental health or other personal challenges, and to students with other kinds of learning challenges. Please feel free to let me know if there are circumstances affecting your ability to participate fully in this course.

Some resources that may be helpful include:

- **Office of Student Disability Services:** https://sds.cornell.edu  
- **Cornell Health CAPS (Counseling & Psychological Services):** https://health.cornell.edu/services/counseling-psychiatry  
- **Undocumented/DACA Student Support:** See the list of campus resources at  
  https://dos.cornell.edu/undocumented-daca-support/undergraduate-admissions-financial-aid


### Academic Integrity

We expect academic integrity from everyone. School is stressful,
and you may feel pressure from your coursework or other factors,
but that is no reason for dishonesty!  If you feel you can't complete
the work on the own, come talk to the professor, the TA, or your advisor,
and we can help you figure out what to do.

For more information, see Cornell's [Code of Academic Integrity][coai].

### Emergency Procedures

In the event of a major campus emergency, course requirements, deadlines, and
grading percentages are subject to changes that may be necessitated by a
revised semester calendar or other circumstances. Any such announcements will
be posted to [Ed Discussion][Ed] and [the course home page](index.html).

[Ed]: {{ site.data.logistics.discussion }}
[canvas]: https://canvas.cornell.edu/courses/85162/assignments/syllabus
[coai]: https://deanoffaculty.cornell.edu/faculty-and-academic-affairs/academic-integrity/code-of-academic-integrity/
