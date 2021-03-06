#  RDBUnit: Unit testing for relational database queries

[![Build Status](https://travis-ci.org/dspinellis/rdbunit.svg?branch=master)](https://travis-ci.org/dspinellis/rdbunit)

**RDBUnit** is a unit testing framework for relational database queries.
It allows you to express in a simple way the setup prior to a query,
the query, and the expected results.
RDBUnit can test `SELECT` queries as well as queries that are used
for creating tables and views.
All types of queries can be either embedded into the test script, or
they can be included from an external file.
The tables for the input and the expected results can be created with
minimal ceremony: RDBUnit will automatically infer the types of the
tables' fields.

For complex relational OLAP queries *RDBUnit* can be combined particularly
effectively with the [simple-rolap](https://github.com/dspinellis/simple-rolap)
relational online analytical processing framework.
You can find a complete tutorial on using *RDBUnit* with *simple-rolap*
for mining Git repositories in a
[technical briefing](https://www.spinellis.gr/git-mine-briefing/)
presented at the 2017 International Conference on Software Engineering.

## Installation
* Clone this repository
* Run make install
```
git clone --depth=1 https://github.com/dspinellis/rdbunit.git
cd rdbunit
sudo make install
```

## Test specification
For every SQL query you want to test, create an *RDBUnit* file that
specifies the query's input, execution, and expected result.

### Simple example
The following example illustrates this concept.
```
BEGIN SETUP
sales:
month   revenue
March   130
April   50
END

BEGIN SELECT
SELECT MAX(revenue) as max_revenue FROM sales;
END

BEGIN RESULT
max_revenue
130
END
```

### Input file details
The input and output and output are specified as table contents.
The input starts with a line containing the words `BEGIN SETUP`,
while the results start with a line containing the words
`BEGIN RESULTS`.
The input and output are specified by first giving a table's name,
followed by a colon.
The name may be prefixed by the name of a database where the table
is to reside, followed by a dot.
The next row contains the table's fields, separated by spaces.
Then comes the table's data, which is terminated by a blank line,
or by the word `END`.
More than one table can be specified in the setup.
In the results the table name is not specified, if the tested
query is a selection statement, rather than a table or view creation.
The setup-query-results sequence can be specified multiple times within
a test file.

### Setup example
```
BEGIN SETUP
contacts:
name    registered      value   reg_date
John    true            12      '2015-03-02'
Mary    false           10      '2012-03-02'
END
```

### Results example (named table)
Named table results are used with `CREATE` queries.
```
BEGIN RESULT
leadership.nl_commits_leader_comments:
project_id      n
1               3
END
```

### Results example (unnamed set)
Unnamed set results are used with `SELECT` queries.
```
BEGIN RESULT
name    registered      value   reg_date        a
John    True            12      '2015-03-02'    Null
END
```

The query to test is either specified inline with a `BEGIN SELECT` (for
selection queries) or `BEGIN CREATE` (for creation queries) statement,
or by including a file through the corresponding `INCLUDE SELECT` or
`INCLUDE CREATE` statement.

### Inline example
```
BEGIN SELECT
SELECT *, NULL AS a FROM contacts WHERE registered;
END
```

### External query example
```
INCLUDE CREATE nl_commits_leader_comments.sql
```

## Unit testing
To run the tests run *RDBUnit* piping its output to one of the supported
databases (current *MySQL*, *PostgreSQL*, and *sqLite*).
A number of command-line flags allow you to tailor the operation of
*RDBUnit*.
When running, *RDBUnit* will report on its output something like
`ok 1 - recent_commit_projects.rdbu: test_stratsel.recent_commit_projects`
for each succeeding test case and
`not ok 1 - project_stars.rdbu: test_stratsel.project_stars` for each failing
test case.
A final line will list the number of succeeding and failing test cases.
By default *RDBUnit* creates and operates on temporary databases,
whose name is prefixed with the word `test_`.

### Execution example (SQLite)
```sh
$ rdbunit --database=sqlite recent_commit_projects.rdbu | sqlite3
ok 1 - recent_commit_projects.rdbu: test_stratsel.recent_commit_projects
```

### Execution example (MySQL)
```sh
$ rdbunit commits_comments.rdbu | mysql -u root -p -N
Enter password:
ok 1 - commits_comments.rdbu: test_leadership.nl_commits_leader_comments
ok 2 - commits_comments.rdbu: test_leadership.leader_commits_nl_comments
ok 3 - commits_comments.rdbu: test_leadership.commits_with_comments
1..3
```

### Execution example (PostgreSQL)
```sh
$ rdbunit --database=postgresql commits_comments.rdbu | psql -U ght -h 127.0.0.1 -t -q ghtorrent
 ok 1 - commits_comments.rdbu: test_leadership.nl_commits_leader_comments

 ok 2 - commits_comments.rdbu: test_leadership.leader_commits_nl_comments

 ok 3 - commits_comments.rdbu: test_leadership.commits_with_comments

 1..3
```
