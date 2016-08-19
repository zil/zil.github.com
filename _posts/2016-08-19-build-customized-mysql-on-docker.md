---
layout: post
title: Build A Customized MySQL Under Docker
description: ""
category: "docker"
tags: [docker,mysql]
---
{% include JB/setup %}

Recently I am testing a Hibernate application,starting a default mysql Docker container couldn't be simpler:  
`docker run -p 3306:3306 mysql:5.6`


## Here comes the problem:  

- the SQL Hibernate generated is __upper-case__  

- the Mysql container's table name is __case sensitive__

## Not very familiar with Hibernate,Changing Mysql behavior is a good way.  

- Through the Mysql documentation,found the mysqld(server) parameter `lower_case_table_names`.  

- On Unix/Linux OS,`lower_case_table_names` default value is 0,which is case sensitive.  
    Found a way to set it to 1 will fixes the problem.

## The Docker way
  
-  use Dockerfile to build image above another image,the core and cool feature within Docker.

- `CMD` it's a command in Dockerfile
	 to specify which program to exec after the container started

## Let's build a customized Mysql

-  save the below content to a file named `Dockerfile`

		FROM mysql:5.6
		EXPOSE 3306
		CMD ["mysqld", "--lower-case-table-names=1", "--character-set-server=utf8"]

- within the same directory as the above Dockerfile file:  
`docker build -t my-mysql .`

- run your customized my-mysql:  
`docker run --name mysql-test -p 3306:3306 my-mysql`
