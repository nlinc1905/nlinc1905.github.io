---
layout: post
title:  "R Package to Query Oracle DBs"
date:   2017-03-24 22:35:22 -0500
categories: R, SQL, Oracle
---
I am a lazy person when it comes to doing things on the computer, so I try to automate everything I can.  Something I frequently find myself doing at work is using the RODBC package to run Oracle SQL queries in R.  This requires storing credentials (which poses a security risk), typing a line of code to read a SQL query, another line to connect to the database, and another line to execute the query.  So I built a package that wraps all of these steps into 1 function, and eliminates the need to store user credentials in an R script.  The package can be downloaded here: <https://github.com/nlinc1905/oracleQueries>.