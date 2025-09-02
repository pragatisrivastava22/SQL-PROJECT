USE ZOMATO;

SELECT * FROM DATA;

-- It's observed that data is redundant.
-- The data can be normalized into four tables (Cities, Restaurants, Cuisines and Items).

## Table "Cities"
CREATE TABLE CITIES (
  CITY_ID INT AUTO_INCREMENT PRIMARY KEY,
  CITY_NAME VARCHAR(255) UNIQUE NOT NULL,
  AVG_PRICE_CITY DECIMAL(10, 4)
);

### Insert into Cities table
INSERT INTO CITIES (CITY_NAME, AVG_PRICE_CITY)

SELECT DISTINCT CITY, AVG_PRICE_CITY

FROM DATA

WHERE CITY IS NOT NULL;

SELECT * FROM CITIES;

/*This table serves as a lookup table for unique cities. It eliminates redundancy by storing city-specific information (like Avg_Price_City) just once, rather than repeating it for every restaurant located in that city.*/

## Table "Restaurants"
CREATE TABLE RESTAURANTS (
  RESTAURANT_ID INT AUTO_INCREMENT PRIMARY KEY,
  RESTAURANT_NAME VARCHAR(100),
  PLACE_NAME VARCHAR(100),
  DINING_RATING DECIMAL(3, 2),
  DINING_VOTES INT,
  DELIVERY_VOTES INT,
  DELIVERY_RATING DECIMAL(3, 2),
  AVG_RATING DECIMAL(3, 2),
  TOTAL_VOTES INT,
  RESTAURANT_POPULARITY INT,
  AVG_RATING_RESTAURANT DECIMAL(3, 2),
  AVG_PRICE_RESTAURANT DOUBLE,
  IS_HIGHLY_RATED INT,
  CITY_ID INT,
  FOREIGN KEY (CITY_ID) REFERENCES CITIES(CITY_ID)
);

### Insert into Restaurants table
INSERT INTO RESTAURANTS (
  RESTAURANT_NAME, PLACE_NAME, DINING_RATING, DINING_VOTES, DELIVERY_VOTES, DELIVERY_RATING,
  AVG_RATING, TOTAL_VOTES, RESTAURANT_POPULARITY, AVG_RATING_RESTAURANT, AVG_PRICE_RESTAURANT,
  IS_HIGHLY_RATED, CITY_ID
)
SELECT DISTINCT(RESTAURANT_NAME), PLACE_NAME, DINING_RATING, DINING_VOTES, DELIVERY_VOTES,
  DELIVERY_RATING, AVERAGE_RATING, TOTAL_VOTES, RESTAURANT_POPULARITY, AVG_RATING_RESTAURANT,
  AVG_PRICE_RESTAURANT, IS_HIGHLY_RATED, CITY_ID
FROM DATA
JOIN CITIES AS C ON CITY = C.CITY_NAME;

SELECT * FROM RESTAURANTS;

/*This is the core table for restaurant data. It is now more efficient as it no longer contains redundant city or cuisine information. Instead, it uses foreign keys (city_id) to link to the Cities and Cuisines tables, respectively. This structure ensures data integrity and simplifies updates.*/

## Table "Cuisines"
CREATE TABLE CUISINES (
  CUISINE_ID INT AUTO_INCREMENT PRIMARY KEY,
  CUISINE_NAME VARCHAR(100),
  AVG_RATING_CUISINE DECIMAL(3, 2),
  AVG_PRICE_CUISINE DOUBLE
);
### Insert into Cuisines table
INSERT INTO CUISINES (CUISINE_NAME, AVG_RATING_CUISINE, AVG_PRICE_CUISINE)
SELECT DISTINCT (CUISINE), AVG_RATING_CUISINE, AVG_PRICE_CUISINE
FROM DATA
WHERE CUISINE IS NOT NULL;

SELECT * FROM CUISINES;

/*Similar to the Cities table, this table centralizes all unique cuisine types. It holds aggregate data like Avg_Rating_Cuisine and Avg_Price_Cuisine, preventing these values from being duplicated across all menu items belonging to the same cuisine.*/

## Table "Items"
CREATE TABLE ITEMS (
  ITEM_ID INT AUTO_INCREMENT PRIMARY KEY,
  ITEM_NAME VARCHAR(500),
  BEST_SELLER VARCHAR(50),
  VOTES INT,
  PRICE INT,
  PRICE_PER_VOTE DOUBLE,
  IS_BESTSELLER INT,
  RESTAURANT_ID INT,
  FOREIGN KEY (RESTAURANT_ID) REFERENCES RESTAURANTS(RESTAURANT_ID),
  CUISINE_ID INT,
  FOREIGN KEY (CUISINE_ID) REFERENCES CUISINES(CUISINE_ID)
);
### Insert into Items table
INSERT INTO ITEMS (ITEM_NAME, BEST_SELLER, VOTES, PRICE, PRICE_PER_VOTE, IS_BESTSELLER, RESTAURANT_ID, CUISINE_ID)
SELECT
  ITEM_NAME,
  BEST_SELLER,
  VOTES,
  PRICES,
  PRICE_PER_VOTE,
  IS_BESTSELLER,
  RESTAURANT_ID,
  CUISINE_ID
FROM DATA AS D
JOIN RESTAURANTS AS REST ON D.RESTAURANT_NAME = REST.RESTAURANT_NAME
JOIN CUISINES AS C ON D.CUISINE = C.CUISINE_NAME;

SELECT * FROM ITEMS;
/*This table is dedicated to menu item details. By linking to the Restaurants table via restaurant_id and cuisine table via cuisine_id, it keeps the item-specific data separate from restaurant-specific data, thereby reducing the overall size and redundancy of the dataset. This makes it easier to manage and query item-level information.*/

SELECT COUNT(ITEM_NAME) FROM DATA ORDER BY ITEM_NAME;

-- Items table has a lot of duplicate values.
-- The query displays the count of rows that are duplicate.
SELECT COUNT(*) AS TOTAL_GROUPS
FROM (
  SELECT
    COUNT(ITEM_NAME),
    VOTES,
    PRICE,
    PRICE_PER_VOTE,
    IS_BESTSELLER,
    RESTAURANT_ID,
    CUISINE_ID,
    COUNT(*) AS DUPLICATE_COUNT
  FROM ITEMS
  GROUP BY
    ITEM_NAME,
    VOTES,
    PRICE,
    PRICE_PER_VOTE,
    IS_BESTSELLER,
    RESTAURANT_ID,
    CUISINE_ID
  HAVING
    COUNT(*) > 1
) AS GROUPED_RESULTS;

SELECT COUNT(*) FROM ITEMS;

-- This query deletes the duplicate rows.
DELETE FROM ITEMS
WHERE ITEM_ID IN (
  SELECT ITEM_ID
  FROM (
    SELECT
      ITEM_ID,
      ROW_NUMBER() OVER (
        PARTITION BY ITEM_NAME, VOTES, PRICE, PRICE_PER_VOTE, IS_BESTSELLER, RESTAURANT_ID, CUISINE_ID
        ORDER BY ITEM_ID
      ) AS RN
    FROM ITEMS
  ) AS SUBQUERY
  WHERE RN > 1
);

/*This query is used to detect and count the number of duplicate groups within your Items table. It identifies rows with identical values across item_name, votes, price, price_per_vote, is_bestseller, restaurant_id, and cuisine_id. The final result, total_groups, represents the total number of these duplicate entries, highlighting a data quality issue that needs to be addressed, likely by removing the duplicates.*/

-- Retrieve all restaurants in a specific city:
# 1. Write a SQL query to list all restaurants located in 'Jaipur'.
SELECT R.RESTAURANT_NAME, C.CITY_NAME
FROM RESTAURANTS AS R
JOIN CITIES AS C ON C.CITY_ID = R.CITY_ID
WHERE C.CITY_NAME = 'JAIPUR';

/*So, the conclusion is that the output provides a comprehensive list of all restaurants specifically located in Jaipur, based on the data in your restaurants table.*/

-- Find the average rating for each cuisine type:
# 2. Write a SQL query to calculate the average rating for each cuisine.
SELECT CUISINE_NAME, AVG_RATING_CUISINE
FROM CUISINES;

/*It's designed to retrieve the pre-calculated average rating for each cuisine.*/

-- Identify the top 5 highly-rated restaurants:
# 3. Write a SQL query to find the top 5 restaurants with the highest Average_Rating.
SELECT RESTAURANT_NAME,
  AVG_RATING_RESTAURANT AS TOP_RESTAURANTS
FROM RESTAURANTS
ORDER BY TOP_RESTAURANTS DESC
LIMIT 5;

/*This query's output directly presents the top five restaurants based on their average rating, allowing for a quick and clear identification of the best-performing establishments in your dataset.*/

-- Count the number of best-selling items for each restaurant:
# 4. Write a SQL query to count how many BESTSELLER items each restaurant has.
SELECT R.RESTAURANT_NAME,
  COUNT(I.BEST_SELLER) AS BEST_SELLING_ITEM
FROM ITEMS AS I
JOIN RESTAURANTS AS R ON R.RESTAURANT_ID = I.RESTAURANT_ID
GROUP BY RESTAURANT_NAME
ORDER BY BEST_SELLING_ITEM;

/*This query summarizes the best-selling item count for each restaurant, highlighting which establishments have the most popular items on their menu.*/

-- Calculate the total votes for each item, ordered by popularity:
# 5. Write a SQL query to find the total Votes for each Item_Name, ordered in descending order of votes.
SELECT ITEM_NAME,
  R.TOTAL_VOTES AS TOTAL_VOTES
FROM ITEMS AS I
JOIN RESTAURANTS AS R ON R.RESTAURANT_ID = I.RESTAURANT_ID
ORDER BY TOTAL_VOTES DESC;

/*This query ranks menu items by their total votes, showcasing the most popular dishes across all restaurants.*/

-- Find restaurants with both high dining and delivery ratings:
# 6. Write a SQL query to list Restaurant_Name that have a Dining_Rating above 4.0 and Delivery_Rating above 4.0.
SELECT DISTINCT (RESTAURANT_NAME)
FROM RESTAURANTS
WHERE DINING_RATING > 4.0 AND DELIVERY_RATING > 4.0;

/*This query ranks menu items by their total votes, showcasing the most popular dishes across all restaurants.*/

-- Determine the average price of dishes for each city:
# 7. Write a SQL query to calculate the average Prices of items in each City.
SELECT CITY_NAME,
  AVG_PRICE_CITY AS AVERAGE_PRICES
FROM CITIES
ORDER BY AVERAGE_PRICES;

/*This query provides a ranked list of cities by their average item prices, making it easy to identify the most and least expensive dining locations.*/

-- List all items that are marked as 'MUST TRY' and their prices:
# 8. Write a SQL query to find all Item_Name and their Prices where Best_Seller is 'MUST TRY'.
SELECT ITEM_NAME, PRICE
FROM ITEMS
WHERE BEST_SELLER = 'MUST TRY';
/*This query successfully isolates and presents all items designated as 'MUST TRY' and their corresponding prices, offering a quick and targeted view of highly recommended dishes.*/

# 9. Write a SQL query to compare the average votes for items that are a best-seller (1) versus those that are not (0).
SELECT IS_BESTSELLER,
  AVG(VOTES) AS AVG_VOTES
FROM ITEMS
GROUP BY IS_BESTSELLER;
/*This query provides a clear comparison of 'item popularity' by calculating the average number of votes for best-selling items versus non-best-selling items. The output helps to quantify whether the "Bestseller" label is an accurate reflection of an item's higher vote count.*/

# 10. Write a SQL query to rank cities based on their average rating in descending order.
SELECT CITY_NAME,
  AVG_RATING_CITY AS AVG_RATING_CITY
FROM CITIES
ORDER BY AVG_RATING_CITY DESC;
/*This query provides a definitive 'ranking of cities' based on their average rating, allowing for a clear comparison of dining quality across different locations.*/
