---
layout: post
title: "Easy Excel Exporting from Mysql"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

From time to time,you may need **export some data in excel format**,I has been doing that by using Java or DB tool.

Recently I use `mysql` cmd line a lot: **redirect from a sql file and dump the result to a file**.

`mysql -uYour_User -pYour_Pass -hYour_Host -DYour_DB < you_sql_file > result`

__How does it related to excel exporting ?__

# It lies on the `result` file

* The result file's first row is headings,the rest is one record per row with field separated by space[^1].

* You could just copy the content and paste it to a empty excel sheet.

* Save the sheet and you are done,no Java or DB tool !


[^1]: if you sql not ends with `\G`,
