# New-and-Deleted-Car-Listing-Analysis-in-SQL
# Comparison between 2 dates as in the script comparing inside my database table using azure studio to analyze the variation, that helps to now what is the trend cars and what is the risky cars in my stock based on the market vdirection including listing data of all car selleres in uae.

# SQL Query:

IF NOT EXISTS(
    SELECT * FROM INFORMATION_SCHEMA.COLUMNS 
    WHERE TABLE_NAME = 'dubicarsdatax' AND COLUMN_NAME = 'row_status'
)
BEGIN
    ALTER TABLE dubicarsdatax ADD row_status VARCHAR(10);
END;

;WITH Data_April01 AS (
    SELECT *, 'Existing' AS status
    FROM dubicarsdatax
    WHERE scrape_date = '2024-04-01'
),
Data_April02 AS (
    SELECT *, 'Existing' AS status
    FROM dubicarsdatax
    WHERE scrape_date = '2024-04-02'
)

,Comparison AS (
    SELECT 
        COALESCE(a.item_id, b.item_id) as item_id,
        CASE 
            WHEN a.item_id IS NOT NULL AND b.item_id IS NOT NULL THEN 'unchanged'
            WHEN a.item_id IS NOT NULL AND b.item_id IS NULL THEN 'deleted'
            WHEN a.item_id IS NULL AND b.item_id IS NOT NULL THEN 'new'
        END AS status
    FROM Data_April01 a
    FULL OUTER JOIN Data_April02 b 
        ON a.item_id = b.item_id AND a.car_model = b.car_model AND a.price = b.price -- Ensure all relevant columns are included in the join condition
)

UPDATE d
SET d.row_status = c.status
FROM dubicarsdatax d
INNER JOIN Comparison c ON d.item_id = c.item_id
WHERE d.scrape_date IN ('2024-04-01', '2024-04-02');

-- Optional: Select to view the result
SELECT * FROM dubicarsdatax
WHERE scrape_date IN ('2024-04-01', '2024-04-02');
