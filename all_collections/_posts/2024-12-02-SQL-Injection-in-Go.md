---
layout: post
title: SQL Injection in Go
date: 2024-12-02
categories: ["Go", "1-Day-Analysis"]
---

# Summary

SQL Injection is critical vulnerability allow attacker dump all data in database and steal all sensitive infomation of admin and customer. Additional, if database allow write permission, hacker can write malicious file and RCE system. 

In go lang, SQL injection also no exception, if developer not validate input or use query to database with string concatenate then sql injection will appear. Today, i will understand about sql injection in go lang, Where vulnerability will appear? And how to fix this?

# Struct of API App in golang

In the App golang often use struct such as: 
- Route: URL API and call to function controller
- Controller: This is function handle input from url API and data, status to response.
- Services: This is function handle logic of application, validate data.
- Repository: handle all database interacts including query, insert, delete, update, ...

This is struct be divided in to small file, have some developer will combine Controller vs Services to one file to easy manage. 

So to find vulnerability sql injection, I will focus to file Repository. At here, i alway check all query statement, if have query statement use string concatenate i will continue check at Services file. At services file if input not validate, maybe is vulnerability. To determine vulnerability is true or false, I will use sqlmap tool to check. 

# Analysis CVE-2024-45794



# Recommendation

