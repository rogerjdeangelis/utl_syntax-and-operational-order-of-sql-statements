# utl_syntax-and-operational-order-of-sql-statements
Syntax and Operational Order of SQL statements.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Syntax and Operational Order of SQL statements

    Unlike datastep language the sequence of operations is much
    different in SQL.

    There is no sense of order in SQL, the 'order by' was a last addition to SQL and
    happens after all other SQL processing. This is why it is so difficult
    to get order statistics from SQL. However costly changes have
    been made to SQL to get some order stats.

    Probably not totally accurate for 'proc sql' but is still helpful

    Group by is 'evaluated' before the select clause.

    ORDER OF SQL OPERATIONS

    From      Where    Group    Having   Select    Distinct   Union  Order
    FLORIDIAN WARRIORS GATHER   HAPPY    SEAWEED   DRIED      UNDER  OWNINGS

    PROGRAMMING ORDER  (SIMPLE QUERY)

    Select   From      Group by   Having  Order
    SOME     FRENCH    GROW       HAIRY   ORANGES


    Inspired by
    https://goo.gl/Nq3kzA
    https://dzone.com/articles/a-beginners-guide-to-the-true-order-of-sqlnbspoper?edition=254681&utm_source=Daily%20Digest&utm_medium=email&utm_campaign=dd%202016-12-14

    Example
    proc sql;
    SELECT distinct(substr(name,1,2)), count(*) FROM sashelp.class WHERE name eqt 'J' GRPUP BY substr(name,1,2)  having count(*)>1
    ;quit;

    ANSI SQL

    The logical order of operations is the following (for simplicity” I’m leaving out vendor specific
    things like CONNECT BY, MODEL, MATCH_RECOGNIZE, PIVOT, UNPIVOT and all the others):

    FROM: This is actually the first thing that happens, logically. Before anything else, we’re loading all
      the rows from all the tables and join them. Before you scream and get mad: Again, this is what happens
      first logically, not actually. The optimiser will very probably not do this operation first, that would
      be silly, but access some index based on the WHERE clause. But again, log
      ically, this happens first. Also: all the JOIN clauses are actually part of this FROM clause. JOIN is
      an operator in relational algebra. Just like + and - are operators in arithmetics.
      It is not an independent clause, like SELECT or FROM

    WHERE: Once we have loaded all the rows from the tables above, we can now throw them away again using WHERE

    GROUP BY: If you want, you can take the rows that remain after WHERE and put them in groups or buckets,
      where each group contains the same value for the GROUP BY expression (and all the other rows are put in a
      list for that group). In Java, you would get something like: Map<String, List<Row>>. If you do specify a
      GROUP BY clause, then your actual rows contain only the group columns
      , no longer the remaining columns, which are now in that list. Those columns in the list are only visible to
      aggregate functions that can operate upon that list. See below.
      aggregations: This is important to understand. No matter where you put your aggregate function syntactically
      (i.e. in the SELECT clause, or in the ORDER BY clause), this here is the step where aggregate functions are
      calculated. Right after GROUP BY. (remember: logically. Clever databases may have calculated them before, actually).
      This explains why you cannot put an aggregate func
      tion in the WHERE clause, because its value cannot be accessed yet. The WHERE clause logically happens before
      the aggregation step. Aggregate functions can access columns that you have put in “this list” for each group, above.
      After aggregation, “this list” will disappear and no longer be available. If you don’t have a GROUP BY clause,
      there will just be one big group without any k
      ey, containing all the rows.

    HAVING: … but now you can access aggregation function values. For instance, you can check that count(*) > 1
      in the HAVING clause. Because HAVING is after GROUP BY (or implies GROUP BY), we can no longer access columns
      or expressions that were not GROUP BY columns.
      WINDOW: If you’re using the awesome window function feature, this is the step where they’re all calculated.
      Only now. And the cool thing is, because we have already calculated (logically!) all the aggregate functions,
      we can nest aggregate functions in window functions. It’s thus perfectly fine to write things like sum(count(*))
      OVER () or row_number() OVER (ORDER BY count(*)). Win
      dow functions being logically calculated only now also explains why you can put them only in the SELECT or
      ORDER BY clauses. They’re not available to the WHERE clause, which happened before. Note that PostgreSQL and Sybase
      SQL Anywhere have an actual WINDOW clause!

    SELECT: Finally. We can now use all the rows that are produced from the above clauses and create new rows / tuples
      from them using SELECT. We can access all the window functions that we’ve calculated, all the aggregate functions
      that we’ve calculated, all the grouping columns that we’ve specified, or if we didn’t group/aggregate, we can use
      all the columns from our FROM clause. Rem
      ember: Even if it looks like we’re aggregating stuff inside of SELECT, this has happened long ago, and the sweet
      sweet count(*) function is nothing more than a reference to the result.

    DISTINCT: Yes! DISTINCT happens after SELECT, even if it is put before your SELECT column list, syntax-wise. But
      think about it. It makes perfect sense. How else can we remove distinct rows, if we don’t know all the rows (and their columns) yet?

    UNION, INTERSECT, EXCEPT: This is a no-brainer. A UNION is an operator that connects two subqueries. Everything
      we’ve talked about thus far was a subquery. The output of a union is a new query containing the same row types (i.e. same columns)
      as the first subquery. Usually. Because in wacko Oracle, the penultimate subquery
      is the right one to define the column name. Oracle database
      , the syntactic troll

    ORDER BY: It makes total sense to postpone the decision of ordering a result until the end, because all other
      operations might use hashmaps, internally, so any intermediate order might be lost again. So we can now order
      the result. Normally, you can access a lot of rows from the ORDER BY clause, including rows (or expressions)
      that you did not SELECT. But when you specified DISTINC
      T, before, you can no longer order by rows / expressions that were not selected. Why? Because the ordering would be quite undefined.







