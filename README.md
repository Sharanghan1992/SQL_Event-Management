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
