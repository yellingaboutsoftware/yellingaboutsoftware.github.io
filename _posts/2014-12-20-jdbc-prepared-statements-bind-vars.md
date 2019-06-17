---
title: 1-based Indices in JDBC PreparedStatements
date: 2014-12-20 14:00:00
---
From [the JDBC docs for setting bind variables](http://docs.oracle.com/javase/7/docs/api/java/sql/PreparedStatement.html#setInt(int,%20int)):

>parameterIndex - the first parameter is 1, the second is 2, ...

Apparently someone forgot that we were writing software here? THE FIRST PARAMETER SHOULD ALWAYS BE 0! Fuck off.