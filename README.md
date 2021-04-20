/* What is the most popular genre? In which countries is it purchased? */
SELECT g.Name, SUM(l.Quantity) num_purchases, i.BillingCountry
FROM Genre g
JOIN Track t
ON g.GenreId = t.GenreId
JOIN InvoiceLine l
ON t.TrackId = l.TrackId
JOIN Invoice i
ON l.InvoiceId = i.InvoiceId
GROUP BY g.Name
ORDER BY num_purchases DESC;

SELECT g.GenreId, g.Name, SUM(l.Quantity) num_purchases, i.BillingCountry
FROM Genre g
JOIN Track t
ON g.GenreId = t.GenreId
JOIN InvoiceLine l
ON t.TrackId = l.TrackId
JOIN Invoice i
ON l.InvoiceId = i.InvoiceId
WHERE g.Name = 'Rock'
GROUP BY i.BillingCountry
ORDER BY num_purchases DESC;

/* Who is the top  performing sales support agent based on their total sales AND number of customers they support */
SELECT e.employeeId, e.FirstName, e.LastName, COUNT (DISTINCT c.CustomerId) num_customers_assit, ROUND(SUM(i.total),2) total_sales
FROM Employee e
JOIN Customer c
ON e.EmployeeId = c.SupportRepId
JOIN Invoice i
ON c.CustomerId = i.CustomerId
GROUP BY e.employeeId, e.FirstName, e.LastName
ORDER BY total_sales DESC, num_customers_assit DESC

/* Which country generates the highest sales revenue. What are the top 5 contributing states in this country and what percentage do they each contribute to the sales revenue for the country? */
SELECT i.BillingCountry, ROUND(SUM(l.UnitPrice*l.Quantity),2) sales_rev
FROM Invoice i
JOIN InvoiceLine l
ON i.InvoiceId = l.InvoiceId
GROUP BY i.BillingCountry
ORDER BY sales_rev DESC;

WITH t_c AS(
	SELECT i.BillingCountry, ROUND(SUM(l.UnitPrice*l.Quantity),2) sales_rev
	FROM Invoice i
	JOIN InvoiceLine l
	ON i.InvoiceId = l.InvoiceId
	WHERE i.BillingCountry = 'USA'
	GROUP BY i.BillingCountry
	ORDER BY sales_rev DESC
	)
SELECT i.BillingCountry, i.BillingState, t_c.sales_rev USA_sales_rev, SUM(l.UnitPrice*l.Quantity) state_sales_rev, ROUND(((SUM(l.UnitPrice*l.Quantity))/(t_c.sales_rev)*100),2) per_USA_sales_rev
FROM Invoice i
JOIN InvoiceLine l
ON i.InvoiceId = l.InvoiceId
JOIN t_c
ON i.Billingcountry = t_c.BillingCountry
GROUP BY i.BillingState, i.BillingCountry
ORDER BY per_USA_sales_rev  DESC
LIMIT 5;


/* How much revenue is generated every year and what is the percentage difference from the previous year? */
WITH prev_year AS (
SELECT CAST(STRFTIME('%Y',i.InvoiceDate) AS INT) pre_yr , SUM(i.Total) AS rev_pre_year
FROM invoice i
GROUP BY STRFTIME('%Y',i.InvoiceDate)
),
curr_year AS (
SELECT CAST(STRFTIME('%Y',i.InvoiceDate) AS INT) AS cur_yr, SUM(i.Total) AS rev_cur_year
FROM invoice i
GROUP BY STRFTIME('%Y',i.InvoiceDate)
)
SELECT curr_year.cur_yr AS 'Year', curr_year.rev_cur_year AS 'Revenue', ROUND((((curr_year.rev_cur_year - prev_year.rev_pre_year)/prev_year.rev_pre_year)*100),2) AS 'Percent Change'
FROM curr_year
JOIN prev_year
ON curr_year.cur_yr = prev_year.pre_yr +1
GROUP BY curr_year.cur_yr;
