---
layout: page
permalink: /teaching/
title: teaching
nav: true
nav_order: 6
---

### Courses
{% assign courses = site.teaching %}
{% for course in courses %}
- [{{ course.title }}]({{ course.url | relative_url }})
    - {{ course.semester}}
{% endfor %}
<!-- - [PES I/O Fundamentals of Deep Learning](../_teaching/rideshare.md) -->

### Guest Lectures

- Data analysis and model building with sensitive content for PESU CS: Data Analytics
    - Host Instructor: [Dr. Gowri Srinivasa](https://staff.pes.edu/nm1084/)
    - Fall 2023 / [Slides](https://docs.google.com/presentation/d/1sIcbwlwae_CSZv1HwvbtY0iQJbqN-sgF7U0N0lTiDOU/edit#slide=id.gc6f919934_0_0)

### Teaching assistantship
- Head TA for PESU CS UE18CS354 : Cloud Computing Laboratory
    - Primary Instructor: [Venkatesh Prasad](https://staff.pes.edu/nm1445/)
    - Spring 2021