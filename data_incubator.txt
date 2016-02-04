# create table
CREATE TABLE hits(
    id int NOT NULL AUTO_INCREMENT,
    time datetime,
    user_id int,
    category int,
    PRIMARY KEY(id)
);

#load csv file into table called hits
LOAD DATA INFILE 'hits.csv' 
    INTO TABLE hits
    FIELDS TERMINATED BY ',' 
    LINES TERMINATED BY '\n' 
    IGNORE 1 LINES
(time,
user_id,
category);

/*
# find the average mean-visit-length when there is at least 2 visits
# too complicated
SELECT minmaxdate.user_id, 
       LAST.time - FIRST.time, 
FROM    (
        SELECT user_id, 
               MIN(time) firstdate, 
               MAX(time) lastdate 
        FROM   hits
        GROUP  BY user_id
        ) minmaxdate 
       INNER JOIN hits FIRST 
         ON FIRST.user_id = minmaxdate.user_id 
            AND FIRST.time = minmaxdate.firstdate 
       INNER JOIN hits LAST 
         ON FIRST.user_id = minmaxdate.user_id 
            AND LAST.time = minmaxdate.lastdate 
*/

# find the average visit length of each user
SELECT 
    user_id,
    TIME_TO_SEC(TIMEDIFF(MAX(time), MIN(time))) AS time_difference
FROM hits
GROUP BY user_id;



# Average number of seconds between first and last visit (among those who visited more than once)
# rounded to 10 sig figs
SELECT CAST(AVG(time_difference) AS decimal(10,7))
FROM (
    SELECT 
    TIME_TO_SEC(TIMEDIFF(MAX(time), MIN(time))) AS time_difference
    FROM hits
    GROUP BY user_id
)
hits
WHERE time_difference != 0;

/*
# find the average mean-visit-length when there is at least 2 visits
# rounded to 10 sig figs, but did not work
SELECT ROUND(AVG(time_difference), 10 - FLOOR(LOG(10, AVG(time_difference))) - 1)
FROM (
    SELECT 
    TIME_TO_SEC(TIMEDIFF(MAX(time), MIN(time))) AS time_difference
    FROM hits
    GROUP BY user_id
)
hits
WHERE time_difference != 0;
*/

result = 792.3397257

# Average number of seconds between consecutive visits by a user
SELECT CAST(AVG(time_difference) AS decimal(10,7))
FROM (
    SELECT TIME_TO_SEC(TIMEDIFF(MAX(time), MIN(time)))
            /
            (COUNT(DISTINCT(time)) -1) AS time_difference
    FROM hits
    GROUP BY user_id
    )
hits
WHERE time_difference != 0;

result = 163.7227846;

# average number of visits per user
SELECT CAST(AVG(time_count) AS decimal(10,9)) AS averge_time
FROM (
    SELECT 
    COUNT(time) AS time_count
    FROM hits
    GROUP BY user_id
)
hits;

result = 4.770513906

# Average number of visits per user given they visited more than once
SELECT CAST(AVG(time_count) AS decimal(10,9)) AS averge_time
FROM (
    SELECT 
    COUNT(time) AS time_count
    FROM hits
    GROUP BY user_id
)
hits
WHERE time_count > 1;

result =  6.960358729

# Average number of categories visited per user
SELECT CAST(AVG(category_count) AS decimal(10,9))
FROM (
    SELECT 
    COUNT(DISTINCT category) AS category_count
    FROM hits
    GROUP BY user_id
)
hits;

result = 1.723885900

# Average number of categories visited per user given they visitied more than once
SELECT CAST(AVG(category_count) AS decimal(10,9))
FROM (
    SELECT 
    COUNT(DISTINCT category) AS category_count,
    COUNT(time) AS time_count
    FROM hits
    GROUP BY user_id
)
hits
WHERE time_count > 1;

result = 2.144305459

# Average number of categories visited per user given they visitied more than one category
SELECT CAST(AVG(category_count) AS decimal(10,9))
FROM (
    SELECT 
    COUNT(DISTINCT category) AS category_count,
    COUNT(time) AS time_count
    FROM hits
    GROUP BY user_id
)
hits
WHERE category_count > 1;

result = 2.831528356

# Probability of immediately visiting a page of the same category (given that the user visits again) evenly averaged over all categories.
SELECT CAST(AVG(probability) AS decimal(10,7))
FROM (
    SELECT COUNT(MAX(category) = MIN(category))
            /
            (COUNT(DISTINCT(time)) -1) AS probability, 
            COUNT(time) AS time_count
    FROM hits
    GROUP BY user_id
    )
hits
WHERE time_count > 1;


# for testing smaller data sets
LOAD DATA INFILE 'test.csv' 
    INTO TABLE test
    FIELDS TERMINATED BY ',' 
    LINES TERMINATED BY '\n' 
(time,
user_id,
category);

SELECT same_category / (Count(DISTINCT(category)))
FROM
  (SELECT Count(time) - 1 AS same_category,
        time, user_id,
          category

   FROM test
   GROUP BY category, user_id
   HAVING Count(*) > 1) test
;

#find the number of distinct categories
SELECT count(distinct(category))
FROM hits

#total number of consecutive repeat-category visits by user
(select sum(sub_total) from (select count(time) -1 as sub_total from hits group by user_id) hits )
#result = 3713828

# Probability of immediately visiting a page of the same category (given that the user visits again) evenly averaged over all categories
SELECT sum(probability) AS real_total from
  (
    SELECT user_id, (sum(same) / 3713828) AS probability
   FROM
     (
      SELECT *, count(time)-1 AS same
      FROM hits
      GROUP BY user_id, category
      HAVING count(*) >1
     ) hits
   GROUP BY user_id
  ) hits

(select sum(sub_total) from (select count(time) -1 as sub_total from test group by user_id) test ) as total

 select *, ROW_NUMBER()
             AS probability from hits where user_id = 421158;

      SELECT *, count(time)-1 AS same
      FROM hits
      where user_id = 421158
      GROUP BY user_id, category, time
      HAVING count(*) >1



SELECT SUM(CASE WHEN y.category=y.next_category THEN @var+1 ELSE @var END) consecIds
    FROM
    (SELECT t.time, t.category, next_id, n.category next_category
      FROM
    (
      SELECT t.time, t.category,

      (
        SELECT time
          FROM test
         WHERE time > t.time
         ORDER BY time
         LIMIT 1
      ) time1
        FROM test t,(SELECT @var:=0)x
    ) t LEFT JOIN test n

         ON t.next_id = n.time 
         GROUP BY t.category,n.category)y


        SELECT time
          FROM test
         WHERE time > t.time
         ORDER BY time
         LIMIT 1
