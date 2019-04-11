
# SQL Subqueries

## Introduction

SQL queries can get complex. For example, you might have been a little thrown off from the many to many join in the last lab. There you had to join four tables. This is just the tip of the iceberg. Depending on how your database is set up, you might have to join subset views of multiple tables. When queries get complex like this, it is often useful to use the concept of subqueries to help break the problem into smaller, more digestible tasks.
M
## Objectives

You will be able to:

* Write subqueries to decompose complex queries

## Our Customer Relation Managment Database Schema

As a handy reference, here's the schema for the CRM database you'll continue to practice with.

<img src="images/Database-Schema.png" width="600">


```python
import sqlite3
import pandas as pd
```


```python
conn = sqlite3.Connection('data.sqlite')
c = conn.cursor()
```

## All of the Employees From the United States

Let's start with a query of employees from the United States. Using your current knowledge, you could solve this using a join.


```python
c.execute("""select lastName, firstName, officeCode
                    from employees
                    join offices
                    using(officeCode)
                    where country = "USA";
          """
         )
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lastName</th>
      <th>firstName</th>
      <th>officeCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Bow</td>
      <td>Anthony</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jennings</td>
      <td>Leslie</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Murphy</td>
      <td>Diane</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Patterson</td>
      <td>Mary</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Thompson</td>
      <td>Leslie</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Firrelli</td>
      <td>Julie</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Patterson</td>
      <td>Steve</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Tseng</td>
      <td>Foon Yue</td>
      <td>3</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Vanauf</td>
      <td>George</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



Another approach would be to use a subquery. Here's what it would look like.


```python
c.execute("""select lastName, firstName, officeCode
                    from employees
                    where officeCode in (select officeCode from offices where country = "USA");
          """
         )
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lastName</th>
      <th>firstName</th>
      <th>officeCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Murphy</td>
      <td>Diane</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Patterson</td>
      <td>Mary</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bow</td>
      <td>Anthony</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Jennings</td>
      <td>Leslie</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Thompson</td>
      <td>Leslie</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Firrelli</td>
      <td>Julie</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Patterson</td>
      <td>Steve</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Tseng</td>
      <td>Foon Yue</td>
      <td>3</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Vanauf</td>
      <td>George</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



There it is! A query within a query! This can be very helpful and also allow you to break down problems into constituent parts. Often queries can be formulated in multiple ways as with the above example. Other times, using a subquery might be essential. For example, what if you wanted to find all of the employees from offices with at least 5 employees?  

Think for a minute about how you might write such a query.  


Now that you've had a minute to think it over, you might see some of the challenges with this query. On the one hand, we are looking to filter based on an aggregate condition: the number of employees per office. You know how to do this using a groupby and having clause, but the data we wish to retrieve is not aggregate data: we only wish to filter based on the aggregate, not retrieve aggregate data. As such, this is a natural place to use a subquery.


```python
c.execute("""select lastName, firstName, officeCode
                    from employees
                    where officeCode in (select officeCode 
                                                from offices 
                                                join employees
                                                using(officeCode)
                                                group by 1
                                                having count(employeeNumber) >= 5
                                         );
          """
         )
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lastName</th>
      <th>firstName</th>
      <th>officeCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Murphy</td>
      <td>Diane</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Patterson</td>
      <td>Mary</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Firrelli</td>
      <td>Jeff</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bondur</td>
      <td>Gerard</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bow</td>
      <td>Anthony</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Jennings</td>
      <td>Leslie</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Thompson</td>
      <td>Leslie</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Bondur</td>
      <td>Loui</td>
      <td>4</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Hernandez</td>
      <td>Gerard</td>
      <td>4</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Castillo</td>
      <td>Pamela</td>
      <td>4</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Gerard</td>
      <td>Martin</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



You can chain queries like this in many fashions. For example, maybe you're also a statistical geek and want to find the average of individual customers average payments:

(It might be more interesting to investigate the standard deviation of customer's average payments, but standard deviation is not natively supported in sqlite as it is in other sql versions like postgreSQL.)


```python
c.execute("""select AVG(avgPayment)
                    from (select AVG(amount) as avgPayment from payments);
          """
         )
df = pd.DataFrame(c.fetchall())
df.columns = [x[0] for x in c.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AVG(avgPayment)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>32431.645531</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

In this lesson, you were briefly introduced to the powerful concept of subqueries and how you can use them to write more complex queries. In the upcoming lab you'll really start to strengthen your SQL and data wrangling skills by using all of the SQL techniques introduced thus far.
