# Codecademy-Data-Science-Project-2
Solutions to Codecademy data science project #2; Exploring a sample database

-- BASIC CHALLENGE

-- Which tracks appeared in the most playlist? How many playlist did they appear in?
SELECT tracks.TrackId, tracks.Name AS 'Track Name', COUNT(playlist_track.TrackId) AS 'Number of playlist appeared in'
FROM tracks
LEFT JOIN playlist_track
ON tracks.TrackId = playlist_track.TrackId
GROUP BY playlist_track.TrackId
ORDER BY COUNT(playlist_track.TrackId) DESC;

-- Which track generated the most revenue? 
SELECT tracks.TrackId, tracks.Name AS 'Track Name', IFNULL(SUM(invoice_items.UnitPrice*invoice_items.Quantity),0) AS 'Revenue'
FROM tracks
LEFT JOIN invoice_items
ON tracks.TrackId = invoice_items.TrackId
GROUP BY invoice_items.TrackId
ORDER BY SUM(invoice_items.UnitPrice*invoice_items.Quantity) DESC;

-- Which Album?
SELECT albums.AlbumId, albums.Title AS 'Album', IFNULL(SUM(invoice_items.UnitPrice*invoice_items.Quantity),0) AS 'Revenue'
FROM tracks
LEFT JOIN albums
ON tracks.AlbumId = albums.AlbumId
LEFT JOIN invoice_items
ON tracks.TrackId = invoice_items.TrackId
GROUP BY albums.AlbumId
ORDER BY SUM(invoice_items.UnitPrice*invoice_items.Quantity) DESC;

-- Which Genre?
SELECT genres.GenreId, genres.Name AS 'Genre', IFNULL(SUM(invoice_items.UnitPrice*invoice_items.Quantity),0) AS 'Revenue'
FROM tracks
LEFT JOIN genres
ON tracks.GenreId = genres.GenreId
LEFT JOIN invoice_items
ON tracks.TrackId = invoice_items.TrackId
GROUP BY genres.GenreId
ORDER BY SUM(invoice_items.UnitPrice*invoice_items.Quantity) DESC;

--Which countries have the highest sales revenue? What percent of total revenue does each country make up?
SELECT invoices.BillingCountry, SUM(invoices.Total) AS 'Sales Revenue by Country', ROUND(((SUM(invoices.Total)/(SELECT SUM(invoices.Total)FROM invoices))*100),1) AS '% of Total Revenue'
FROM invoices
GROUP BY invoices.BillingCountry
ORDER BY SUM(invoices.Total) DESC;

--How many customers did each employee support, what is the average revenue for each sale and what is their total sale?
WITH total_customer_sales AS (
SELECT invoices.CustomerId, SUM(invoices.Total) AS 'Cus_Sum_Sales'
FROM invoices
GROUP BY invoices.CustomerId
)
SELECT employees.EmployeeId, employees.LastName, employees.FirstName, COUNT(customers.CustomerId) AS 'No. of customer per employee', SUM(total_customer_sales.Cus_Sum_Sales) AS 'Total sales per employee', ROUND((AVG(total_customer_sales.Cus_Sum_Sales)),2) AS 'Avg rev per sale'
FROM customers
LEFT JOIN employees
ON customers.SupportRepId = employees.EmployeeId
LEFT JOIN total_customer_sales
ON customers.CustomerId = total_customer_sales.CustomerId
GROUP BY employees.EmployeeId
ORDER BY COUNT(customers.CustomerId) DESC;

-- OR

SELECT employees.EmployeeId, customers.SupportRepId, COUNT(DISTINCT customers.CustomerId) AS 'No of customer per employee', ROUND((SUM(invoices.Total)),2) 'Total sales per employee', ROUND((SUM(invoices.Total)/COUNT(DISTINCT customers.CustomerId)),2) AS 'Avg rev per sale'
FROM customers
LEFT JOIN employees 
ON  customers.SupportRepId = employees.EmployeeId
LEFT JOIN invoices 
ON customers.CustomerId = invoices.CustomerId 
GROUP BY employees.EmployeeId
ORDER BY COUNT(customers.CustomerId) DESC;

-- INTERMEDIATE CHALLENGE

-- Do longer or shorter length albums tend to generate more revenue?
WITH album_items AS (
SELECT tracks.AlbumId, COUNT(tracks.TrackId) AS 'No_of_Tracks_in_Album', (SUM(tracks.Milliseconds))/(1000*60) AS 'Album_Length_mins'
FROM tracks
GROUP BY tracks.AlbumId
)
SELECT tracks.AlbumId, albums.Title AS 'Album', album_items.No_of_Tracks_in_Album, album_items.Album_Length_mins, SUM(invoice_items.UnitPrice*invoice_items.Quantity) AS 'Album_Revenue'
FROM tracks
LEFT JOIN albums
ON tracks.AlbumId = albums.AlbumId 
LEFT JOIN album_items
ON tracks.AlbumId = album_items.AlbumId
LEFT JOIN invoice_items
ON tracks.TrackId = invoice_items.TrackId
GROUP BY tracks.AlbumId
ORDER BY Album_Revenue DESC;

-- OR

WITH songs_revenue AS (
SELECT invoice_items.TrackId, SUM(invoice_items.UnitPrice*invoice_items.Quantity) AS 'Song_Sales'
FROM invoice_items
GROUP BY invoice_items.TrackId
)
SELECT tracks.AlbumId, albums.Title AS 'Album', COUNT(tracks.TrackId) AS 'No_of_Tracks_in_Album', (SUM(tracks.Milliseconds))/(1000*60) AS 'Album_Length_mins', SUM(songs_revenue.Song_Sales) AS 'Album_Revenue'
FROM tracks
LEFT JOIN albums
ON tracks.AlbumId = albums.AlbumId 
LEFT JOIN songs_revenue
ON tracks.TrackId = songs_revenue.TrackId
GROUP BY tracks.AlbumId
ORDER BY Album_Revenue DESC;

-- Is the number of times a track appears in any playlist a good indicator of sales?
-- No, some tracks that appeared in the highest number of playlists were not ordered and tracks that appeared in the smallest number of playlists had more sales.
WITH tracks_apperance AS (
SELECT playlist_track.TrackId, COUNT(playlist_track.PlaylistId) AS 'Total_apperance_in_playlists'
FROM playlist_track
GROUP BY playlist_track.TrackId
)
SELECT tracks_apperance.TrackId, tracks_apperance.Total_apperance_in_playlists, SUM(invoice_items.UnitPrice) AS 'Track_Rev', (SUM(invoice_items.UnitPrice))/(tracks_apperance.Total_apperance_in_playlists) AS 'Avg_Track_Rev'
FROM playlist_track
LEFT JOIN invoice_items
ON playlist_track.TrackId = invoice_items.TrackId
LEFT JOIN tracks_apperance
ON playlist_track.TrackId = tracks_apperance.TrackId
GROUP BY playlist_track.TrackId
ORDER BY Avg_Track_Rev DESC;

-- ADVANCED CHALLENGE

-- How much revenue is generated every year and what is the percent change from the previous year?
WITH previous_year AS (
SELECT CAST(strftime('%Y',invoices.InvoiceDate) AS INT) AS 'Pre_year', SUM(invoices.Total) AS 'Rev_pre_year'
FROM invoices
GROUP BY strftime('%Y',invoices.InvoiceDate)
), 
current_year AS (
SELECT CAST(strftime('%Y',invoices.InvoiceDate) AS INT) AS 'Cur_year', SUM(invoices.Total) AS 'Rev_cur_year'
FROM invoices
GROUP BY strftime('%Y',invoices.InvoiceDate)
)
SELECT current_year.Cur_year AS 'Year', current_year.Rev_cur_year AS 'Revenue', ROUND((((current_year.Rev_cur_year - previous_year.Rev_pre_year)/previous_year.Rev_pre_year)*100),2) AS 'Percent Change'
FROM current_year
JOIN previous_year
ON current_year.Cur_year = previous_year.Pre_year +1
GROUP BY current_year.Cur_year;

-- OR

-- LAG funtion returs the value of a previous role in a column.
SELECT CAST(strftime('%Y', InvoiceDate) AS INT) AS 'Year', SUM(Total) AS 'Revenue', LAG(SUM(Total)) OVER() AS 'Pre_Rev', ROUND((SUM(Total)- LAG(SUM(Total)) OVER())/LAG(SUM(Total)) OVER () * 100, 2) AS 'Percent Change'
FROM invoices
GROUP BY strftime('%Y', InvoiceDate);
