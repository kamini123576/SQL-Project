# SQL-Project
This comprehensive project aims to analyze customer data provided by a bank to understand customer churn (customer loss). 
This project consists of Subjective Questions and the SQL code for those questions are given below:- 

-- 2.) Identify the top 5 customers with the highest Estimated Salary in the last quarter of the year. 

SELECT customerid, surname, estimatedsalary
FROM customerinfo
WHERE EXTRACT(QUARTER FROM BankDOJ) = 4
ORDER BY estimatedsalary DESC
LIMIT 5;


-- 3.) Calculate the average number of products used by customers who have a credit card. 
SELECT AVG(NumOfProducts) AS avg_products_with_credit_card
FROM bank_churn
WHERE HasCrCard = 1; 

-- 5.) Compare the average credit score of customers who have exited and those who remain.
SELECT Exited,
       AVG(CreditScore) AS avg_credit_score
FROM bank_churn
GROUP BY Exited;

-- 6.) Which gender has a higher average estimated salary, and how does it relate to the number of active accounts?
WITH ActiveAccounts AS (
    SELECT CustomerId,COUNT(*) AS Active_Accounts
    FROM Bank_Churn
    WHERE IsActiveMember = 1
    GROUP BY customerId
)
SELECT CASE WHEN c.GenderID = 1 THEN 'Male' ELSE 'Female' END AS Gender,
    COUNT(aa.CustomerId) AS Active_Accounts, AVG(c.EstimatedSalary) AS AvgSalary
FROM CustomerInfo c
LEFT JOIN Active_Accounts aa ON c.CustomerId = aa.CustomerId
GROUP BY Gender
ORDER BY AvgSalary DESC;

-- 7.) Segment the customers based on their credit score and identify the segment with the highest exit rate.
WITH credit_score_segments AS (
  SELECT
    customerid, isactivemember,
   CASE
      WHEN creditscore between 800 and 850 THEN 'Excellent'
      WHEN creditscore between 740 and 799 THEN 'Very Good'
      WHEN creditscore between 670 and 739 THEN 'Good'
      WHEN creditscore between 580 and 669 THEN 'Fair'
      ELSE 'Poor'
    END AS credit_score_segment
  FROM bank_churn
)
SELECT
  credit_score_segment,
  AVG(CASE WHEN isactivemember = 0 THEN 0 ELSE 1 END) AS exit_rate
FROM credit_score_segments
GROUP BY credit_score_segment
ORDER BY exit_rate DESC
LIMIT 1;

-- 8.) Find out which geographic region has the highest number of active customers with a tenure greater than 5 years.
SELECT 
    g.geographylocation, COUNT(b.customerId) AS active_customers
FROM
    geography g
        JOIN
    customerinfo c ON g.GeographyID = c.GeographyID
        JOIN
    bank_churn b ON c.CustomerId = b.CustomerId
WHERE
    b.tenure > 5
GROUP BY g.GeographyLocation
ORDER BY active_customers DESC
LIMIT 1;


-- 14.) How many different tables are given in the dataset, out of these tables which table only consists of categorical variables
 activecustomer , creditcard , customerinfo , gender , geography

-- 15.) Using SQL, write a query to find out the gender-wise average income of males and females in each geography id. Also, rank the gender according to the average value.
WITH geographic_avg_salary AS (
SELECT g.geographylocation,
    CASE
        WHEN c.genderid = 1 THEN 'Male'
        ELSE 'Female'
    END AS gender,
    AVG(c.estimatedsalary) AS avg_salary
FROM
    customerinfo c
JOIN geography g ON c.GeographyID = g.GeographyID
GROUP BY g.Geographylocation,c.GenderID
ORDER BY g.GeographyLocation)

SELECT *, RANK() OVER(PARTITION BY geographylocation ORDER BY avg_salary DESC) AS `rank`
FROM geographic_avg_salary;

-- 16.) Using SQL, write a query to find out the average tenure of the people who have exited in each age bracket (18-30, 30-50, 50+)
SELECT 
    CASE
        WHEN age BETWEEN 18 AND 30 THEN 'Adult'
        WHEN age BETWEEN 31 AND 50 THEN 'Middle-Aged'
        ELSE 'Old-Aged'
    END AS age_brackets,
    AVG(tenure) AS avg_tenure
FROM
    customerinfo c
        JOIN
    bank_churn b ON c.CustomerId = b.CustomerId
WHERE
    b.exited = 1
GROUP BY age_brackets
ORDER BY age_brackets;

-- 20.) According to the age buckets find the number of customers who have a credit card. Also retrieve those buckets that have lesser than average number of credit cards per bucket
WITH info AS (
SELECT 
    CASE
        WHEN c.Age BETWEEN 18 AND 30 THEN 'Adult'
        WHEN c.Age BETWEEN 31 AND 50 THEN 'Middle-Aged'
        ELSE 'Old-Aged'
    END AS age_brackets,
    count(c.CustomerId) AS HasCreditCard
FROM customerinfo c JOIN bank_churn b ON c.CustomerId=b.CustomerId
WHERE HasCrCard = 1
GROUP BY age_brackets)
SELECT *
FROM info
WHERE HasCreditCard < (SELECT AVG(HasCreditCard) FROM info);

-- 21.) Rank the Locations as per the number of people who have churned the bank and average balance of the customers.
SELECT g.GeographyLocation, COUNT(b.CustomerId) AS num_exited_people, AVG(b.CustomerId) AS avg_balance
FROM bank_churn b
JOIN customerinfo c ON b.CustomerId = c.CustomerId
JOIN geography g ON c.GeographyID = g.GeographyID
WHERE b.Exited = 1
GROUP BY g.GeographyLocation
ORDER BY Count(b.CustomerId)desc;

-- 23.) Without using “Join”, can we get the “ExitCategory” from ExitCustomers table to Bank_Churn table? If yes do this using SQL.
SELECT CustomerId,CreditScore,Tenure,Balance,NumOfProducts,HasCrCard,IsActiveMember,
    CASE
        WHEN Exited = 0 THEN 'Retain'
        ELSE 'Exit'
    END AS ExitCategory
FROM
    bank_churn;

-- 25.) Write the query to get the customer IDs, their last name, and whether they are active or not for the customers whose surname ends with “on”.
SELECT 
    c.customerid,
    c.surname,
    CASE
        WHEN b.isactivemember = 1 THEN 'Active'
        ELSE 'InActive'
    END AS activity_status
FROM
    customerinfo c
        JOIN
    bank_churn b ON c.customerid = b.customerid
WHERE
    c.surname REGEXP 'on$'
ORDER BY c.surname;



Subjective Question
-- 9.) Utilize SQL queries to segment customers based on demographics and account details.
SELECT 
    g.GeographyLocation,
    CASE
        WHEN EstimatedSalary < 50000 THEN 'Low'
        WHEN EstimatedSalary < 100000 THEN 'Medium'
        ELSE 'High'
    END AS income_segment,
    CASE
        WHEN c.genderid = 1 THEN 'Male'
        ELSE 'Female'
    END AS gender, age,
    COUNT(c.CustomerId) AS number_of_customers
FROM customerinfo c
        JOIN
    geography g ON c.geographyid = c.geographyid
GROUP BY income_segment , g.geographylocation , gender,age
ORDER BY g.GeographyLocation,age;

-- 14.) In the “Bank_Churn” table how can you modify the name of the “HasCrCard” column to “Has_creditcard”?
ALTER TABLE bank_churn
RENAME COLUMN HasCrCard TO Has_creditcard;
