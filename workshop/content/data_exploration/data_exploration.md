+++
title = "Step by Step"
weight = 2
+++

1. Navigate to the S3 service and look at the subfolders under processed/instacart in the codingforexecs bucket. Notice that the data is in files, rather than in a database running on a server. It’s going to be fairly difficult to analyse these files without knowing their structure. We can solve this problem by using AWS Glue!

2. Navigate to AWS Glue and look at the orders crawler. Note the S3 locations it crawls through, and note how it’s discovered the tables and determined the columns for the data files in S3.

3. We now need to determine the top 8 ordered products by our customers across all departments as these are the products we’d like to provide to our most loyal customers (generous, I know!). To determine this let’s navigate to AWS Athena, and run a SQL query on the Datalake database that determines the top products in each department:

        SELECT
        "product_id"
        , "product_name"
        , "department"
        , "orders"
        , "ranking"
        FROM
        (
        SELECT
            "count"(1) "orders"
        , "op"."product_id"
        , "p"."product_name"
        , "d"."department"
        , "row_number"() OVER (PARTITION BY "d"."department" ORDER BY "count"(1) DESC) "ranking"
        FROM
            ((((orders o
        INNER JOIN users u ON ("u"."user_id" = "o"."user_id"))
        INNER JOIN order_products op ON ("op"."order_id" = "o"."order_id"))
        INNER JOIN products p ON ("p"."product_id" = "op"."product_id"))
        INNER JOIN departments d ON ("d"."department_id" = "p"."department_id"))
        GROUP BY "op"."product_id", "p"."product_name", "d"."department"
        ) 
        WHERE ("ranking" = 1)
        ORDER BY "orders" DESC

4. Great, so we have the top items across each department, but looking at the results, there’s probably a few departments that we want to remove. Baby food isn’t exactly the ideal surprise gift for a loyal customer! Let’s modify the SQL statement to remove the results for a number of departments, and limit the results to the top.

        SELECT
        "product_id"
        , "product_name"
        , "department"
        , "orders"
        , "ranking"
        FROM
        (
        SELECT
            "count"(1) "orders"
        , "op"."product_id"
        , "p"."product_name"
        , "d"."department"
        , "row_number"() OVER (PARTITION BY "d"."department" ORDER BY "count"(1) DESC) "ranking"
        FROM
            ((((orders o
        INNER JOIN users u ON ("u"."user_id" = "o"."user_id"))
        INNER JOIN order_products op ON ("op"."order_id" = "o"."order_id"))
        INNER JOIN products p ON ("p"."product_id" = "op"."product_id"))
        INNER JOIN departments d ON ("d"."department_id" = "p"."department_id"))
        GROUP BY "op"."product_id", "p"."product_name", "d"."department"
        ) 
        WHERE ("ranking" = 1) and department not in ('bulk', 'deli', 'babies', 'dairy eggs', 'dry goods pasta')
        ORDER BY "orders" DESC
        LIMIT 8

5. Excellent! Not sure I want recycled paper towels, dropped out of the sky as a loyalty gift, but let’s go with it. Rather than writing down the top 8 products, let’s save the search as a Saved query so we can run it on demand later.

6. Click on Save as and name the statement most_ordered_products-team#

## Added Challenge!

Now that you know how to find the most ordered products, can you write a SQL statement to find the least popular products? If you work it out, be sure to save the search like you did in step 6, to show participants after.
