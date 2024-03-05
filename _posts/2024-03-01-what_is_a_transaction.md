---
layout: distill
title: what is a transaction
description: a short explication of what is a transaction in the context of database management systems
tags: dbms sql transaction 
giscus_comments: true
date: 2024-03-01
featured: false

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Introduction
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Understanding Transactions
    subsections:
       - name: Consistency and Integrity
       - name: Definition
       - name: ACID
  - name: Operations
  - name: Deadlock
  - name: Conclusion
  - name: Further Information


# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

## Introduction

Imagine you want to transfer money from one bank account to another. To do this there are several steps involve: withdrawing the amount from the first account, depositing the money in the second account.
Since money unfortunately doesn't grow in transactions, both of these operations must be completed successfully, if the withdrawn isn't made the deposit can't be done other, the amount deposited must be the same as the amount withdrawn. If it is not possible to complete the deposit the money must return to the account where it was withdrawn from. So these operations must be treated as a single operation or as we can say a single **unit of work**.

From banking to baking, now let's imagine we want to bake chocolate chip cookies, there is a set of actions you have to perform, like, getting all the ingredients, mixing them together, molding the cookies, putting them in the oven and baking them. While you are doing this your mother is also in the kitchen baking a cake. If she were to come and mix some some ingredients of the cake in the mixing bowl while you are preparing your dough, then it would become too runny or too dry. Or imagine the disappointment on your face tasting some bland hard bread because your mother accidentally used the sugar out of your ingredients. So, these operations must be performed with some level of **isolation** to ensure each task's integrity and success. 

In both scenarios, the concept of **"transaction"** comes into play. A transaction, in its essence, is a sequence of operations or actions that are treated as a single unit of work. These operations must either all succeed together or fail together, ensuring **data integrity and consistency**. This concept is pivotal in various domains, including database management and software development. 

---

## Understanding transactions

We use databases to store information, but, for this information to keep it's usefulness it must be accessed and modified from time to time. To maintain the **consistency and integrity** of the data present this tasks should be performed systematically with a specific set of rules. In Database Management Systems this is called a **transaction**.

Before diving deeper into transactions, let's first clarify what we mean by **consistency and integrity**, as these are foundational to understanding the role and importance of transactions in database management.

# Consistency and Integrity

**Consistency** refers to the requirement that any transaction should bring the database from one valid state to another, ensuring that all data follows all rules and constraints (e.g., data types, triggers, constraints) of the database. For instance, in our cookie-baking analogy, consistency would be akin to following a recipe precisely. Just as using the right proportions of ingredients ensures the cookies turn out as expected (not too runny or too dry), in a database, consistency ensures that all data remains accurate and in the correct format throughout any transaction.

**Integrity**, on the other hand, involves maintaining the accuracy and reliability of the data over its entire lifecycle. This means that the data in the database is always accurate, and any changes made to it are done correctly. Going back to our baking analogy, integrity is similar to ensuring that the sugar intended for your cookies isn't mistakenly used in your mother's cake.

In summary, consistency and integrity in databases are like following a recipe and using the correct ingredients in baking. Just as each step and ingredient must be correctly followed and used to produce the desired cookie outcome, transactions in a database must adhere to rules that preserve **data consistency and integrity**, ensuring the database remains useful, accurate, and reliable.

# Definition

[GeeksForGeeks](https://www.geeksforgeeks.org/transaction-in-dbms/) defines a transaction as
```a set of logically related operations. It is the result of a request made by the user to access the contents of the database and perform operations on it. It consists of various operations and has various states in its completion journey. It also has some specific properties that must be followed to keep the database consistent.
```

# ACID 

These proprieties mentioned in the definition followed by transactions are usually referred to has **ACID properties**, they are **Atomicity**, **Consistency**, **Isolation** and **Durability**. We already mentioned Consistency.

**Atomicity** ensures that a transaction is treated as a single unit of work, meaning, that it either completes in its entirety or it doesn't happen at all. There is no in-between state. If any part of the transaction fails, the entire transaction is rolled back, and the database state is left unchanged as if the transaction never occurred. Baking cookies is not a good example for this, because, if you realize midway that your oven is broken, you can not return to a state where all the ingredients are separated and in their original container, but, I'm sure you get the idea.

**Isolation** ensures that transactions are securely isolated from each other. This means the operations of one transaction are hidden from other transactions until it's completed. This property prevents transactions from interfering with each other, ensuring data integrity. Meaning, that the other transaction in your kitchen - your mother - can not interfere with your cookie baking. Like if both of you have separate workstations separate from each other. 

**Durability** guarantees that once a transaction has been committed, it will remain so, even in the event of a power loss, crash, or error. This means the changes made by the transaction are permanently recorded in the database. Once the cookies are baked and taken out of the oven, they don't magically disappear if the power goes out - only if you eat them, but that is another story.

By adhering to these ACID properties, databases ensure that transactions are processed reliably, maintaining the integrity and consistency of the data.

# Operations

We mentioned before that a user can make different types of operations to access the contents of the database.

The information in the database can be read - **Read(X)** - that is, the value is read from the database and stored in a buffer for displaying or other action. Like checking your account ballance online or your pantry to see if you have all the ingredients needed for the cookies. You are not taking anything out or using it.

During a write operation we write - **Write(X)** the value from the memory buffer to the database. It must be preceded by a read operation during which the are brought to the buffer and some operations are performed on it - according to what the user requested. Then the modified value is written in the database. Think of checking you balance (Read) before you withdraw (Write) money from you bank account or you check what you have (Read) before putting all of your ingredients together (Write) to bake your cookies.

**Commit** is an operation that ensures that the integrity of the database is maintained. Operations are only made permanent after all of the work performed by the current transaction is completed, that is, the changes done by the transaction are made permanent in the database. There may be interruptions on transactions (like an error or a power failure), and this way we ensure that the consistency of the data is maintained. Imagine a Commit like you with or apron and the oven gloves with a batch of cookies ready for baking.

**Rollback** is intimately connected to the transaction, it's what happens when a transaction is interrupted, all the operations are undone and the database returns to the original state. The cookie example is not a good one, but, it's like if you decide that the batch isn't good, and there is a operation that can return all the ingredients to the pantry. 

# Deadlock

When a transaction needs to read or modify data, to ensure that no other transaction makes conflicting changes, it acquires **locks** to certain resources. When two or more transactions hold locks on resources the others need to complete their operations, and none can proceed until the other releases its locks. Each transaction is waiting for the other to finish, but neither can without the resources held by the other. This situation is called a **deadlock**.

Deadlocks can significantly hinder database performance, leading to stalled transactions and system inefficiencies. To manage deadlocks, Database Management Systems (DBMS) employ various strategies, including **deadlock detection algorithms** that identify and break deadlocks by aborting one or more transactions to free up resources. Another approach is **deadlock prevention**, which involves designing the database and transactions in a way that avoids the conditions leading to deadlocks.

Imagine you and your mother are both trying to bake in a kitchen with only one oven and one oven mint. If you both are ready to bake and you hold the mint and your mother the oven, neither of you can proceed creating "kitchen deadlock." In databases, similar scenarios require careful management to ensure smooth operation.

# Conclusion

Transactions are fundamental to maintaining the **consistency, integrity, and reliability** of data in database systems. By understanding and implementing the ACID properties, databases can ensure that transactions are processed in a manner that preserves data integrity, even in the face of errors or system failures. Operations like **Read, Write, Commit, and Rollback** play crucial roles in transaction management, while mechanisms to handle deadlocks ensure that databases remain efficient and responsive.

Just as following a recipe step by step leads to delicious cookies, adhering to transaction principles ensures that databases function effectively, supporting a wide range of applications from financial services to online shopping. Understanding transactions and their management is essential for anyone working with database systems, providing the foundation for building robust and reliable software solutions.


## Further information

[Geeks for Geeks](https://www.geeksforgeeks.org/transaction-in-dbms/)

[Tutorialspoint](https://www.tutorialspoint.com/dbms/dbms_transaction.htm)

[Javapoint](https://www.javatpoint.com/dbms-transaction-processing-concept)