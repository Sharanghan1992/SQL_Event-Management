# SQL_Event-Management
This project demonstrates advanced SQL querying techniques using an Event Management dataset. The database simulates a real-world events booking system with multiple entities such as venues, events, companies, account managers, company contacts, and payment types.

The project includes beginner to advanced SQL examples, showcasing filtering, grouping, window functions, common table expressions (CTEs), and complex joins. It is ideal for practicing MS SQL Server skills in query optimization and business analytics.

🗂 Dataset

The dataset (from Event Tables.xlsx) contains the following key tables:
Venue – Details about event venues, including seating capacity, hire cost, category, location, and postcodes.
Event – Records of events held at venues, their duration, quote status, and associated companies.
Company – Information about client companies booking events.
CompanyContact – Contacts associated with companies, including positions, phone numbers, and comments.
AccountManager – Account managers handling companies, including commission rates.
PaymentType – Payment type references.

🧰 Key SQL Features and Examples
1. Filtering and Search Patterns
Using BETWEEN, NOT BETWEEN, and wildcards (LIKE) to filter venues by state, capacity, or keywords.
Finding university venues by searching for 'uni' within names or addresses.
Detecting duplicates and handling NULL values.

2. Aggregations and Grouping
Counting venues, summing seating capacities, and calculating averages per state or category.
Using HAVING clauses to filter grouped data (e.g., suburbs with >10,000 total seats).

3. Joins
Inner, outer, and self-joins to match companies, venues, and events.
Matching companies and venues in the same postcode without relying on PK/FK relationships.
Finding companies without events and venues without bookings.

4. Window Functions
Ranking venues by seating capacity (DENSE_RANK, ROW_NUMBER).
Calculating percentages of total seats used per event category using SUM() OVER().
Ranking hotels within seating capacity groups by hire cost.

5. Common Table Expressions (CTEs)
Multi-step calculations for:
Top five venues by total days hired.
Percentage usage of seats within venue categories.
Account manager sales and commission percentages.

6. Union and Set Operations
Merging contact lists from companies and venues.
Combining company and venue suburb/state data.

7. Conditional Logic
Using CASE to classify account statuses and event risk levels (High/Intermediate/Low) based on multiple conditions.

🎯 Learning Outcomes
Practice real-world data filtering, grouping, and ranking scenarios.
Apply window functions and CTEs for advanced analytics.
Explore joins beyond primary/foreign key relationships.
Gain experience with business-oriented calculations like commission percentages and event risk scoring.

🛠 Technology Stack
Database: MS SQL Server
Dataset: Event Tables.xlsx (sample event and venue data)
SQL Features: Joins, Aggregations, Window Functions, CTEs, Unions, Conditional Logic


```SQL
SELECT * FROM Event

Select 'Event' =
			[Event Name]
			+ ' - ' +
			cast ([Quote Number] as varchar)
			+ ' - (' +
			[Quote Status]
			+ ')',
			[Days Hired] as 'Event Duration in Days',
			[Days Hired] + 3 as 'Staff Days Committed',
			[Venue Hire Start Date] as 'Event Start Date',
			DateAdd(day, [Days Hired], [Venue Hire Start Date]) as 'Event End Date',
			PaymentTypeID
From Event
Where (
			PaymentTypeID = 1
			OR
			PaymentTypeID = 2
			)
		AND
		[Quote Number] >= 125
		AND
		[Quote Number] <= 175
Order by	[Quote Status] desc,
			[Days Hired] desc,
			[Event Name] asc

-- Display venues and their states outside NSW using NOT BETWEEN and the PCode column
SELECT  *
FROM
	Venue
WHERE
	Pcode NOT BETWEEN 2000 AND (3000-1) -- 3000 being the next states first post code
	
-- Display the venues that have a seating capacity between 1000 and 3000 excluding the boundaries.
SELECT *
FROM
	Venue
WHERE
	[Seating Capacity] > 1000
	AND
	[Seating Capacity] < 3000

--OR
SELECT *
FROM
	Venue
WHERE
	[Seating Capacity] BETWEEN (1000+1) AND (3000 - 1)

-- how the venues in Brisbane, Melbourne and Perth.
SELECT *
FROM
	Venue
WHERE
	Suburb IN ('Brisbane', 'Melbourne','Perth')

--Display the venues that may be a university by looking for ‘uni’ anywhere in the address or venue name columns.
SELECT *
FROM
    Venue
WHERE
    Address LIKE '%Uni%'
    OR
    [Venue Category] LIKE '%Uni%'

--Find all company contact last names that start with O'.
SELECT * 
FROM CompanyContact
WHERE
    [Last name] LIKE'O''%'

--Select all company contacts who could be considered some type of manager. Hint: 'Manager' is
-- sometimes spelt 'mgr'. So the common letters are 'm', 'g' and 'r'. There could also be typos in the
-- word 'Manager', but lets assume that we have negotiated with the powers that be that we will
-- return anything with 'm', 'g' and 'r' in that order somewhere in the Position column.

SELECT *
FROM
	CompanyContact
WHERE
	[Position] LIKE '% M%g%r'

-- Show the company contacts that have a null value in their phone field.
SELECT *
FROM
	CompanyContact
WHERE
	Phone IS NULL

--Show the company contacts that have a comment recorded.
SELECT *
FROM
	CompanyContact
WHERE 
	Comments IS NOT NULL
	AND
	comments NOT LIKE ''

--OR

SELECT *
FROM 
	CompanyContact
WHERE 
	Comments IS NOT NULL
	AND
  	CONVERT(NVARCHAR, comments) <> '';


--OFFSET - Fetch
SELECT *
FROM
	venue
ORDER BY	
	[Seating Capacity]
	OFFSET 0 ROWS
	FETCH NEXT 2 ROWS ONLY

--In the Events database, show the 4th cheapest venue using OFFSET and FETCH.
SELECT *,
	RANK() OVER(ORDER BY [HIre Cost per Day] ASC ) as rank
FROM
	venue
ORDER BY
	[Hire Cost per Day]
	OFFSET 4 ROWS
	FETCH NEXT 1 ROWS ONLY

--Display the average days hired of accepted events.

SELECT
	AVG([Days Hired]) as avg_days_hired
FROM
	Event
WHERE
	[Quote Status] = 'Accepted'

-- Display the seating capacity of the largest hotel.
SELECT 
	MAX([Seating Capacity])
 FROM Venue
 WHERE
	[Venue Category] = 'Hotel'

SELECT 
	[Venue Category],
	COUNT(*)
FROM
	Venue
GROUP BY
	[Venue Category]

-- Show each of the states in the venue table and how many venues are in the state, how many seats
-- are in the state, and the largest and smallest state venue seating capacity.
SELECT 
	State,
	COUNT([Venue ID]) AS number_of_venues,
	SUM([Seating Capacity]) AS number_of_seats,
	MAX([Seating Capacity]) AS largets_seating_cap,
	MIN([Seating Capacity]) AS smallest_seating_cap
FROM Venue
GROUP BY
	State

-- Show how many events are hired for 1,2,3,4 and 5 days.

SELECT * FROM Event
SELECT
	[Days Hired],
	COUNT(*)
FROM
	Event
WHERE
	[Days Hired] < = 5
GROUP BY
	[Days Hired]

--For each of the venue categories, show the postcodes that have venues of the category. Count the
--venues and calculate the average cost per day per postcode per venue category.
SELECT
	[Venue Category],
	Pcode,
	COUNT([Venue ID]) AS number_of_venues,
	AVG([Hire Cost per Day]) AS avg_cost
FROM
	Venue
GROUP BY
	[Venue Category],
	Pcode

--Show the number of accepted events having the number of days hired less than 3. (Does this need
-- a HAVING clause and/or a WHERE clause?).
SELECT *
FROM
	Event

SELECT
	COUNT(*)
FROM
	Event
WHERE
	[Quote Status] = 'Accepted'
HAVING
	COUNT(*) < 3

-- Show the venue suburbs, the number of venues in the suburb, and the number of seats in the
-- suburb where the number of seats in the suburb is greater than 10,000 and the number of venues in the suburb is greater than 3.
SELECT * FROM Venue

SELECT
	Suburb,
	COUNT([Venue ID]) AS number_of_venues,
	SUM([Seating Capacity]) AS number_of_seats
FROM
	Venue
GROUP BY
	Suburb
HAVING
	SUM([Seating Capacity]) > 10000
	AND
	COUNT([Venue ID]) > 3

-- Are there any company name duplicates?
SELECT * FROm Company

SELECT
	[Company Name],
	COUNT([Company Name])
FROM
	Company
GROUP BY
	[Company Name]
ORDER BY
	COUNT([Company Name]) DESC


--WIndow Functions

SELECT
	[Venue Name],
	[Venue Category],
	[Seating Capacity],
	[State],
	SUM([Seating Capacity])OVER() AS total_seats,
	SUM([Seating Capacity])OVER( PARTITION BY [Venue Category]) AS total_seats_Venue_category,
	SUM([Seating Capacity])OVER( PARTITION BY [State] ORDER BY [State]) AS total_seats_state
FROM
	venue


SELECT
	[Venue Name],
	[Venue Category],
	[Seating Capacity],
	[State],
	SUM([Seating Capacity]) OVER(PARTITION BY [Venue Category]) AS total_seats,
	CAST
		(100. * [Seating Capacity] / 
		SUM([Seating Capacity]) OVER(PARTITION BY [Venue Category]) AS numeric(5,2))
FROM
	venue

--Row_Number()
SELECT 
	[Venue Name],
	[Venue Category],
	[Seating Capacity],
	row_number() over (Order by [Venue Name] desc) as RowNum
FROM 
	Venue
ORDER BY [Venue ID]

--Rank

SELECT *
FROM
(
	SELECT 
		[Venue Name],
		[Venue Category],
		[Seating Capacity],
		[State],
		DENSE_RANK() OVER(ORDER BY [Seating Capacity] DESC) AS rank
	FROM 
		Venue
) as rank_no
WHERE
	rank = 1



--Examine the following output showing hotels ranked by hire cost per day within seating capacity
-- and write the query to produce it:
-- “Grouped” (don’t do a GROUP BY clause – this the concept only) by seating capacity.
-- ordered by hire cost per day.
-- Ranked within seating capacity (1, 2, 3 etc.) by hire cost per day.

SELECT *
FROM Venue

SELECT
	[Venue Name],
	[Seating Capacity],
	[Hire Cost Per Day],
	DENSE_RANK() OVER(PARTITION BY [Seating Capacity]ORDER BY [Hire Cost per Day] ASC) as rank
FROM
	Venue
WHERE
	[Venue Category] = 'Hotel'

--Union
SELECT 
	Suburb,
	State
FROM 
	Company

UNION

SELECT 
	Suburb,
	State
FROM 
	Venue

-- Show the names and phone numbers for the account managers and the company contacts for a phone number list.

SELECT
	[First Name],
	[Last Name],
	[Mobile Phone No]
FROM AccountManager

UNION

SELECT 
	[First name],
	[Last name],
	Phone
 FROM CompanyContact

--JOINS

SELECT * FROM Venue
SELECT * FROM Event
SELECT * FROM AccountManager
SELECT * FROM Company
SELECT * FROM CompanyContact
SELECT * FROM PaymentType

-- Display the Company Names with accepted Events.
SELECT * FROM Company
SELECT * FROM Event

SELECT
	DISTINCT c.[company name]
FROM
	Company c
JOIN
	Event e
	ON
	e.[Company ID] = c.[Company ID]
WHERE
	e.[Quote Status] = 'Accepted'


--Display the venues that are hired for more than 3 days for any event.
SELECT * FROM Venue
SELECT * FROM Event

SELECT
	DISTINCT v.[Venue Name]
FROM
	Venue v
JOIN
	Event e
	ON
	e.[Venue ID] = v.[Venue ID]
WHERE
	e.[Days Hired] > 3


-- Show companies and the venues in the same postcode. Do this without using the primary key /
-- foreign key columns (because it won’t work). The query can be created without a where clause.
-- Remember the “Ambiguous column name” error. The PCode column name exists in both tables.

SELECT * FROM Company
SELECT * FROM Venue
SELECT * FROM Event

SELECT
	DISTINCT c.[Company Name],
	v.[Venue Name],
	c.Pcode as company_pcode,
	v.Pcode as venue_pcode
FROM
	Company c
JOIN
	Event e ON c.[Company ID] = c.[Company ID]
JOIN
	Venue v ON v.[Venue ID] = e.[Venue ID]
WHERE
	c.Pcode = v.Pcode

--Show all the companies and any company contacts for each company.
SELECT * FROM Company
SELECT * FROM CompanyContact

SELECT
	c.[Company Name],
	cc.[First Name],
	cc.[Last Name]
FROM
	Company c
FULL OUTER JOIN
	CompanyContact cc
	ON cc.[Company ID] = c.[Company ID]

--Show the venues and the events held at them including any venues without any events held at them.
SELECT * FROM Venue
SELECT * FROM Event

SELECT
	v.[Venue Name],
	e.[Event Name]
FROM
	Venue v
LEFT OUTER JOIN
	Event e
	ON e.[Venue ID] = v.[Venue ID]
--WHERE
	--e.[Event Name] IS NULL

--Show companies that have not had events.
SELECT * FROM Company
SELECT * FROM Event

SELECT
	DISTINCT c.[Company Name],
	e.[Event Name]
FROM
	Company c
LEFT JOIN
	Event e
	ON e.[Company ID] = c.[Company ID]
WHERE
	e.[Event Name] IS NULL

--Display the total sales and the total commission for each Account Manager. A row for each account manager is expected in the result.

SELECT * FROM AccountManager
SELECT * FROM Event -- # of days hired
SELECT * FROM Venue --Hire cost per day
SELECT * FROm Company

SELECT
	a.[First Name],
	a.[Last Name],
	SUM(e.[Days Hired] * v.[Hire Cost per Day]) AS total_sales,
	SUM(e.[Days Hired] * v.[Hire Cost per Day]) * (a.[Commission %]) AS mgr_commission
FROM
	AccountManager a
JOIN
	Company c
	ON c.[Acct Mgr ID] = a.[Acct Mgr ID]
JOIN
	Event e
	ON e.[Company ID] = c.[Company ID]
JOIN
	Venue v
	ON v.[Venue ID] = e.[Venue ID]
GROUP BY
	a.[First Name],
	a.[Last Name],
	a.[Commission %]
ORDER BY
	total_sales DESC

-- SELECT the top five venues with the highest usage in terms of total days hired displaying the
-- Suburb, Venue Name and the total Days Hired for each venue.

SELECT * FROM AccountManager
SELECT * FROM Event -- # of days hired
SELECT * FROM Venue --Hire cost per day
SELECT * FROm Company

WITH ranked_venues AS
(
	SELECT
		v.[Venue Name],
		v.Suburb,
		SUM(e.[Days Hired]) AS num_of_days_hired,
		DENSE_RANK() OVER(ORDEr BY SUM(e.[Days Hired]) DESC ) as rank
	FROM
		Event e
	JOIN
		Venue v ON e.[Venue ID] = v.[Venue ID]
	GROUP BY
		v.[Venue Name],
		v.Suburb
)
SELECT *
FROM
	ranked_venues
WHERE
	rank < = 5


--Show companies that have had events at venues in the same postcode.
SELECT
	c.[Company Name],
	c.[PCode] AS company_pcode,
	v.[Pcode] AS venue_pcode
FROM
	Event e
JOIN
	Company c ON e.[Company ID] = c.[Company ID]
JOIN
	Venue v ON v.[Venue ID] = e.[Venue ID]
WHERE
	c.[PCode] = v.[Pcode]

-- Calculate the percentage of the total sales and the percentage of the total commission on sales each account manager earns. 
-- One row per account manager is expected in the result set. 
-- Use the OVER function in your select clause. Don’t use GROUP BY.

;WITH acct_manager_sales AS
(
	SELECT
		a.[First Name] as first_name,
		a.[Last Name] AS last_name,
		SUM(v.[Hire Cost per Day] * e.[Days Hired]) AS total_sales,
		SUM(e.[Days Hired] * v.[Hire Cost per Day] * a.[Commission %]) AS mgr_commission
	FROM
		Event e
	JOIN
		Venue v
		ON v.[Venue ID] = e.[Venue ID]
	JOIN
		Company c
		ON c.[Company ID] = e.[Company ID]
	JOIN
		AccountManager a
		ON
		a.[Acct Mgr ID] = c.[Acct Mgr ID]
	GROUP BY
		a.[First Name],
		a.[Last Name]
)
SELECT DISTINCT
    first_name,
    last_name,
    total_sales,
    mgr_commission,
    CAST(total_sales * 100.0 / SUM(total_sales) OVER () AS DECIMAL(5,2))     AS pct_total_sales,
    CAST(mgr_commission * 100.0 / SUM(mgr_commission) OVER () AS DECIMAL(5,2)) AS pct_total_commission
FROM acct_manager_sales;

--Use a CASE in the SELECT clause and the Acct Established data to create the following result set. A
-- zero in the Acct Established column is assumed to be “No account” while a one is assumed to mean “Account Established.”
SELECT
	[Company Name],
	CASE
		WHEN [Acct Established] = 1 THEN 'Account Established'
		WHEN [Acct Established] = 0 THEN 'No Account'
	END AS Account
FROM
	Company

-- Show the risk associated with events.
--Rules:
--High Risk:
-- If a company has no account established ([Acct Established] = 0) and the number of days for the event is three or more and the event is in Sydney then the risk is seen as high.
--Intermediate Risk:
-- Intermediate risk is defined by the company not having an account with the event days hired being three or more days and the event being held in North Sydney, Melbourne, Brisbane or Perth.
--Low Risk:
--All other events are seen as low risk

SELECT
	c.[Company Name],
	e.[Event Name],
	CASE
		WHEN [Acct Established] = 1 THEN 'Account Established'
		WHEN [Acct Established] = 0 THEN 'No Account'
	END AS Account,
	SUM(e.[Days Hired]) AS number_of_days_hired,
	v.[Suburb],
	CASE
		WHEN [Acct Established] = 0 AND SUM(e.[Days Hired]) >= 3 AND v.[Suburb] = 'Sydney' THEN 'High Risk'
		WHEN [Acct Established] = 0 AND SUM(e.[Days Hired]) >= 3 AND v.[Suburb] IN ('North Sydney', 'Melbourne', 'Brisbane', 'Perth') THEN 'Intermediate Risk'
		ELSE 'Low Risk'
	END AS Risk
FROM
	Company c
INNER JOIN
	Event e ON e.[Company ID] = c.[Company ID]
INNER JOIN
	Venue v ON v.[Venue ID] = e.[Venue ID]
GROUP BY
	c.[Company Name],
	e.[Event name],
	c.[Acct Established],
	v.[Suburb]

-- Show the events and each event’s percentage usage of the total seats in the category.
-- Hint 1: Use the following query to find the total seats in a category and then use the query in the From clause joined to the venue table on [Venue Category]
SELECT * FROM AccountManager
SELECT * FROM Event -- # of days hired
SELECT * FROM Venue --Hire cost per day
SELECT * FROm Company

;WITH cat_seats AS (
	Select 
		v.[Venue ID],
		v.[Venue Category] as CatName,
		sum(v.[Seating Capacity]) as TotalCatSeats
	From 
		Venue v
	Group by 
		v.[Venue ID],
		v.[Venue Category]
)
SELECT
	e.[Event Name],
	CAST(v.[Seating Capacity] * 100/cat_seats.TotalCatSeats AS DECIMAL(5,2)) AS percentage
FROM
	cat_seats
INNER JOIN
	Event e ON e.[Venue ID] = cat_seats.[Venue ID]
INNER JOIN
	Venue v ON v.[Venue ID] = e.[Venue ID]


-- Show the events and each events percentage usage of the total seats in the category.(This is the
-- same exercise we did in the section ‘Queries in the FROM clause’).
-- Use the following query to find the total seats in a category IN A CTE and then use the CTE in the 
-- From clause joined to the venue table on [Venue Category]

-- Method 1
WITH cat_seats AS (
    SELECT 
        v.[Venue Category] AS CatName,
        SUM(v.[Seating Capacity]) AS TotalCatSeats
    FROM Venue v
    GROUP BY v.[Venue Category]
)
SELECT
    e.[Event Name],
   	cs.CatName,
    CAST(v.[Seating Capacity] * 100.0 / cs.TotalCatSeats AS DECIMAL(5,2)) AS PercentageUsage
FROM Event e
INNER JOIN Venue v 
    ON e.[Venue ID] = v.[Venue ID]
INNER JOIN cat_seats cs 
    ON v.[Venue Category] = cs.CatName
ORDER BY v.[Venue Category], e.[Event Name];


--Method 2
WITH cat_seats AS 
(
    SELECT 
        v.[Venue Category] AS CatName,
        SUM(v.[Seating Capacity]) AS TotalCatSeats
    FROM Venue v
    GROUP BY 
		v.[Venue Category]
),
event_seats AS 
(
	SELECT
		e.[Event Name] AS event_name,
		v.[Venue Category] AS cat_name,
		SUM(v.[Seating Capacity]) AS event_seats
	FROM
		Event e
	JOIN
		Venue v ON v.[Venue ID] = e.[Venue ID]
	 GROUP BY 
        e.[Event Name],
        v.[Venue Category]
)
SELECT
	es.event_name,
	cs.CatName,
	CAST(es.event_seats * 100.0/cs.TotalCatSeats AS DECIMAL(5,2)) AS PERCEn
FROM
	cat_seats cs
JOIN
	event_seats es ON cs.CatName = es.cat_name

--Using a CTE, select the top five venues with the highest usage in terms of total days hired
-- displaying the Suburb, Venue Name and the total Days Hired for each venue.

;WITH venue_hire AS
(
	SELECT

		v.[Venue Name] AS venue_name,
		v.[Suburb],
		SUM(e.[Days Hired]) AS number_of_days_hired,
		DENSE_RANK()OVER(ORDER BY SUM(e.[Days Hired])  DESC) AS rank
	FROM
		Venue v
	JOIN
		Event e ON e.[Venue ID] = v.[Venue ID]
	GROUP BY
		v.[Venue Name],
		v.[Suburb]
)
SELECT * FROM
	venue_hire
	WHERE
		rank <= 5
























