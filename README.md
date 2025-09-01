use zomato;
select * from data;

-- Its observed that data is redundant. 
-- The data can be normalized into four tables (Cities, Restaurants, Cuisines and Items).

 # Table "Cities"
CREATE TABLE Cities (
    city_id INT AUTO_INCREMENT PRIMARY KEY,
    city_name VARCHAR(255) UNIQUE NOT NULL,
    Avg_Price_City DECIMAL(10, 4)
);
# Insert into Cities table 
INSERT INTO Cities (city_name, avg_price_city)
SELECT DISTINCT City, Avg_Price_City
FROM data
WHERE City IS NOT NULL;

select * from cities;
## Conclusion - This table serves as a lookup table for unique cities. It eliminates redundancy by storing city-specific information(like Avg_Price_City) just once, rather than repeating it for every restaurant located in that city.

# Table "Restaurants"
create table restaurants
(restaurant_id int auto_increment primary key,
restaurant_name varchar(100),
place_name varchar(100),
dining_rating decimal(3, 2),
dining_votes int,
delivery_votes int, 
delivery_rating decimal(3,2),
avg_rating decimal(3, 2),
total_votes int,
restaurant_popularity int,
avg_rating_restaurant decimal(3,2),
avg_price_restaurant double,
is_highly_rated int,
city_id int,
FOREIGN KEY (city_id) REFERENCES Cities(city_id));

# Insert into Restaurants table 

insert into restaurants (restaurant_name, place_name, dining_rating, dining_votes, delivery_votes, delivery_rating, avg_rating, total_votes, restaurant_popularity, 
avg_rating_restaurant, avg_price_restaurant, is_highly_rated, city_id)
select distinct(Restaurant_Name), Place_Name, Dining_Rating, Dining_Votes, Delivery_Votes, Delivery_Rating, Average_Rating, Total_Votes,
Restaurant_Popularity, Avg_Rating_Restaurant, Avg_Price_Restaurant, Is_Highly_Rated,
city_id from data
join cities AS c ON City = city_name;

select * from restaurants;

## Conclusion -  This is the core table for restaurant data. It is now more efficient as it no longer contains redundant city or cuisine information. Instead, it uses foreign keys (city_id) to link to the Cities and Cuisines tables, respectively. This structure ensures data integrity and simplifies updates.

# Table "Cuisines"
create table cuisines
(cuisine_id int auto_increment primary key,
cuisine_name varchar(100),
avg_rating_cuisine decimal(3,2),
avg_price_cuisine double);

insert into cuisines (cuisine_name, avg_rating_cuisine, avg_price_cuisine)
select distinct (cuisine), avg_rating_cuisine, Avg_Price_cuisine
from data 
where cuisine is not null;

select * from cuisines;
## Conclusion - Similar to the Cities table, this table centralizes all unique cuisine types. It holds aggregate data like Avg_Rating_Cuisine and Avg_Price_Cuisine, preventing these values from being duplicated across all menu items belonging to the same cuisine.

# Table "Items"
create table items
(item_id int auto_increment primary key,
item_name varchar(500),
best_seller varchar(50),
votes int,
price int,
price_per_vote double,
is_bestseller int,
restaurant_id int,
FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id),
cuisine_id int,
FOREIGN KEY (cuisine_id) REFERENCES cuisines(cuisine_id));
 
insert into items (item_name, best_seller, votes, price, price_per_vote, is_bestseller, restaurant_id, cuisine_id )
select item_name, best_seller, votes, prices, Price_per_Vote, is_bestseller, restaurant_id, cuisine_id
from data as d
join restaurants as rest on d.restaurant_name = rest.restaurant_name
join Cuisines as c on d.Cuisine = c.Cuisine_name;

select * from items;

## Conclusion - This table is dedicated to menu item details. By linking to the Restaurants table via restaurant_id and cuisine table via cuisine_id, it keeps the item-specific data separate from restaurant-specific data, thereby reducing the overall size and redundancy of the dataset. This makes it easier to manage and query item-level information.

select count(item_name) from data order by item_name;

-- Items table has a lot of duplicate values. 
-- The query displays the count of rows that are duplicate.
select count(*) as total_groups
from (select count(item_name), votes, price, price_per_vote, is_bestseller, restaurant_id, cuisine_id,
count(*) as duplicate_count
from Items
group by item_name, votes, price, price_per_vote, is_bestseller, restaurant_id, cuisine_id
having count(*) > 1) as grouped_results;
    
select count(*) from items;

-- This query deletes the duplicate rows.
delete from Items
where item_id in (select item_id
from (select item_id, 
row_number() over 
(partition by item_name, votes, price, price_per_vote, is_bestseller, restaurant_id, cuisine_id
order by item_id) as rn
from Items) as subquery
where rn > 1);
## This query is used to detect and count the number of duplicate groups within your Items table. It identifies rows with identical values across item_name, votes, price, price_per_vote, is_bestseller, restaurant_id, and cuisine_id. The final result, total_groups, represents the total number of these duplicate entries, highlighting a data quality issue that needs to be addressed, likely by removing the duplicates.

-- Retrieve all restaurants in a specific city:
# 1. Write a SQL query to list all restaurants located in 'Jaipur'.
  
select r.restaurant_name,c.city_name from restaurants as r
join cities as c on c.city_id = r.city_id
where c.city_name=' Jaipur';

## So, the conclusion is that the output provides a comprehensive list of all restaurants specifically located in Jaipur,based on the data in your restaurants table.

-- find the average rating for each cuisine type:
# 2. Write a SQL query to calculate the average rating for each cuisine.

select cuisine_name, avg_rating_cuisine from cuisines;
## It's designed to retrieve the pre-calculated average rating for each cuisine.

-- Identify the top 5 highly-rated restaurants:
# 3. Write a SQL query to find the top 5 restaurants with the highest Average_Rating.
select Restaurant_Name, 
avg_rating_restaurant as top_restaurants 
from restaurants
order by top_restaurants desc
limit 5; 

-- Count the number of best-selling items for each restaurant:
# 4. Write a SQL query to count how many BESTSELLER items each restaurant has.
select r.restaurant_name,
count(i.best_seller) as best_selling_item
from items as i
join restaurants as r on r.restaurant_id = i.restaurant_id
group by restaurant_name
order by best_selling_item;

-- Calculate the total votes for each item, ordered by popularity:
# 5. Write a SQL query to find the total Votes for each Item_Name, ordered in descending order of votes.
select item_name,
r.total_votes as total_votes
from items as i
join restaurants as r on r.restaurant_id = i.restaurant_id 
order by total_votes desc;

-- Find restaurants with both high dining and delivery ratings:
# 6. Write a SQL query to list Restaurant_Name that have a Dining_Rating above 4.0 and Delivery_Rating above 4.0.
select distinct(restaurant_name) 
from restaurants
where dining_rating < 4.0 and delivery_rating < 4.0;

-- Determine the average price of dishes for each city:
# 7. Write a SQL query to calculate the average Prices of items in each City.
select city_name,
avg_price_city as average_prices
from cities
order by average_prices;

-- List all items that are marked as 'MUST TRY' and their prices:
# 8. Write a SQL query to find all Item_Name and their Prices where Best_Seller is 'MUST TRY'.
select item_name, price
from items 
where best_seller = 'must try';

# 9. Write a SQL query to compare the average Votes for items that Is_Bestseller (1) versus those that are not (0).
select is_bestseller, 
avg(votes) as avg_votes
from items
group by  is_bestseller;

# 10. Write a SQL query to rank City based on their Avg_Rating_City in descending order?"
select city_name,
avg_rating_city as avg_rating_city
from cities
order by avg_rating_city desc;
