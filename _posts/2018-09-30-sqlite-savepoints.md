---
title: "Android SQLite savepoints: nested transactions and workaround"
published: true
---
![Android SQLite](/assets/android_sqlite.webp)
*[Origin](https://www.blikoontech.com/tutorials/android-health-app-design-saving-data-in-sqlite-database)*

Hello! In this article I’ll show one small trick with SQLite savepoints and my story how I’ve found this trick.

Google [claims](https://developer.android.com/reference/android/database/sqlite/package-summary) that Android fully supports SQLite database engine. But the code that framework provides to us sometimes isn’t implemented well, so we cannot use some SQLite features and forced to search for workarounds.

One of the features that doesn’t work as expected is [savepoint](https://sqlite.org/lang_savepoint.html). In short, it’s a tag in SQLite’s operations journal that can mark some important point in operations history to fall back to it if something was wrong. It’s similar to transaction, but savepoints have one big advantage over transactions: you can have as much savepoints as you need. You can have only one transaction at the time, but when you need to undo only particular changes inside huge transaction, savepoint can be used. Actually, savepoints are transactions, but with names and with possibility to nest multiple transactions inside one.

Savepoints may be useful for step-by-step operations that involve database and where each step can be interrupted or something can go wrong there. But even in this case you can solve this problem by state machine in code, without using database in intermediate steps.

If you really need to use savepoints, here is the trick: always start SQL statement for rolling back to particular savepoint with semicolon. Like this:

```java
sqliteDb.execSql(";ROLLBACK TO savepointName;");
```

That’s it for useful tricks. If you want to read a story behind this line of code and how do we get this, keep reading.

At my previous job, we extensively used SQLite for offline work with data and we strictly relied on it. Unfortunately, our business logic was written in very rigid way, so we cannot easily mock database for testing. That’s why we decided to test it with real database. In order not to create clutter in the database and make tests stateless, we decided to use nested transactions, like in Postgres.

After quick search we found out that savepoints are good choice for us. I wrote some simple test case with savepoints, and… it crashed. Exception was strange: it has stated that I’m trying to roll back non-existent transaction. But… there was only one root transaction that should be rolled back in JUnit’s AfterClass method. I’ve spent two days debugging until I’ve found this code in
[android.database.DatabaseUtils#getSqlStatementType](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/core/java/android/database/DatabaseUtils.java;bpv=1;bpt=1;l=1566?q=DatabaseUtils&gsn=getSqlStatementType&gs=kythe%3A%2F%2Fandroid.googlesource.com%2Fplatform%2Fsuperproject%3Flang%3Djava%3Fpath%3Dandroid.database.DatabaseUtils%23794a1d841ce90d1e8ac8aff3bce9f7cea3ce5689e5ca43f027d9aa925bcb68c7) (it’s called inside `android.database.sqlite.SQLiteSession#executeSpecial`
for each SQL statement to find transactional operations). You can check the actual version of the code at the link, but in fact they're determining statement type by first 3 letters of the SQL statement string. And before Android P they haven't made any special treatment for `ROLLBACK TO` statement.

Because of this very clever code, when we try to rollback savepoint (that is done by `ROLLBACK TO` statement), this method decides that we are going to rollback transaction by ROLLBACK statement. It may make sense, because savepoints are not widely used and guys at Google might have easily forget about ROLLBACK TO statement, but it makes no excuses to this sophisticated approach to determine type of SQL statement by first 3 letters of SQL string. TBH, I bitterly smiled looking at that code.

More interesting things are coming. In search for workaround for this behavior, I’ve asked a question on StackOverflow, googled a lot until I found [this issue on Android Issue Tracker](https://issuetracker.google.com/issues/36957161). In short, this issue was opened in 2012, hanged in assigned state since then, had some comments (including one comment with possible workaround that I’ve mentioned above and patch for adding savepoint support to the framework) in 2013 until I wrote additional comment in 2017. After my comment, it was reassigned and fixed in few days with note that “fix will be available in Android P”. I understand that this issue had low priority, but… I haven’t seen such a big gap between opening an issue and resolving it before.

This story shows that Android framework is imperfect, especially its SQLite support. As a conclusion I would recommend you to use savepoints in Android app judiciously.

Thanks for reading! Hope it will be useful for you.
