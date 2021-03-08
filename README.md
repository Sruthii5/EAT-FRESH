# EAT-FRESH

EAT FRESH: A Smart Grocery Hub 
 
Introduction: 
The application is an online grocery hub that recommends products to its users based on their food preference and their selected store. A customer can search for a product using the autocomplete search feature, view products on sale, view the top reviewed products, select products that are currently trending on twitter or filter products by category and subcategory. 
 
 
The application has three distinct parts – client, server and the database. The client is an interactive UI that communicates with the server using a number of REST APIs with the client where all the business logic is stored. The client stores all user and transaction related data on a MYSQL DB instance and product review details on a Mongo DB while all the data is again imported to a NEO4J instance for analysis using knowledge graphs. 
  
  
 
Entity relationship model for database: 
 
Code structure overview: 
The root folder of the application is `\EAT-FRESH` which contains the 
`\EWA_Term_Project\frontend` and `\EAT-FRESH\backend` code. The entry point for backend is `..\backend\server.js` and for frontend is `..\frontend\src\App.js`. All API keys and credentials are stored in the `.env` file. 
  
To run the application locally, please install node and npm on your system and then install the required node modules for both frontend and backend as following: 
1.	Backend : go to path `\EAT-FRESH` and run `npm install` 
2.	Frontend: go to path `\EAT-FRESH\frontend` and run `npm install` 
Finally, to run the application go to path `\EAT-FRESH` and run `npm run dev` 
 
User Journey: 
1.	A user must first register to proceed with a transaction: 
  
Go to Your Account dropdown -> Sign In / Sign Up -> Register 
Fill up the information correctly, select user type and Food Preference and click Register. 
 
2.	Login after registration: 
  
Fill out the registration details and click Sign In 
 
3.	As soon as the user logs in, we can see that we only show his preferred food. This user selected Meat as food preference, so we show him only meat products: 
 
  
4.	User can go to their accounts page to change their preferences and it gets updated: 
  
 
After updating when we go home, we can see all the products this time. 
 
5.	User can use the autocomplete search bar to search for a product
  
6.	User can go to store selection as follows.
  
  
They can zoom in/ Zoom out and once they want to select a store to see products from, they can click on the store location icon and then `Select Store`. 
7.	Once a store is set, it shows products from only that store as seen here: 
  
8.	The Popular Deals shows products that are in sale, Best Seller shows the top-rated products and the Carousel shows the top trending products: 
   
9.	Clicking on a product takes a user to the product details page: 
  
  
10.	After adding items to cart by clicking `Add to cart` button from either home screen or product Details page, customers can now go to Cart by clicking the cart icon. 
  
The cart shows all the added products and their quantities and suggests recommended products using Market Basket Analysis. The Market Basket analysis is done by leveraging all previous transactions done by users. The code and implementation for this can be found inside ` EAT-FRESH/mbaScript.py` 
11.	After Adding a card/selecting an existing card for payment, users can either checkout using pickup or delivery: 
   
12.	Order Confirmation page: 
  
 
13.	On Orders page users can view the order information, cancel an order, reorder the item or review the item 
  
 
14.	Under account details, users can view their account information, update them, view and update stored address details and card detais: 
  
 
 
15.	As an Admin, we can change a customer to a Store_Manager and Recalculate our market basket analysis: 
  
 
16.	A store manager can view, add and remove products to a store: 
   
Mysql and Mongo DB: 

**DB Name: ewaDB 
Hostname: ewa-term-project-instance.ccstkwakl93l.us-east-2.rds.amazonaws.com Port: 3306 
Username: admin 
Password: #ewa_term_project# **
 
 
 
Knowledge Graphs and Analytics using Neo4j: 
We have imported out MySql and mongo DB data into Neo4j and then performed Knowledge Graph Searches and Analytics.  
 
How to run Neo4j Graph database: 
Step 1: Generate the CSV files from the database. 
Step 2.1 : Import csv files into neo4j: Go to Manage →Open Folder → Import Step 2.2:  load the data using Cypher query to create nodes and relationships. 
 
The CSV  files imported into Neo4 for performing analytics are the following: 
 
transactions.csv, shares.csv, users.csv, products.csv, orders.csv, reviews.csv, stores.csv, storeproducts.csv 
 
The csv files and import scripts have been attached under `data` folder. 
 
 
The top two neo4j analytics performed are: 
1.	PageRank  
2.	Betweenness 
 
 
1. PageRank: The Queries required to create the nodes and relationships for pagerank can be found under `../data/pagerank.txt`. 
Visualizing the Nodes and Relationships: 
 
 
// Compute nodePropertiesWritten, createMillis, computeMillis, writeMillis, ranIterations 
CALL gds.pageRank.write({ 
nodeQuery: 'MATCH (u:Customer)-[:WROTE]->(r:Review), (p:product)-[:HAS_REVIEWS]->(r:Review),(p)-
[:HAS_CATEGORY]->(:Category {category_name: $category}) WITH u, count(*) AS reviews WHERE reviews >= $cutoff 
RETURN id(u) AS id', relationshipQuery: 'MATCH (u1:Customer)-[:WROTE]->(r:Review), (p:product)-[:HAS_REVIEWS]->(r:Review),(p)[:HAS_CATEGORY]->(:Category {category_name: $category}) MATCH (u1)-[:FRIENDS_WITH]->(u2) RETURN id(u1) as source, id(u2) AS target', writeProperty: "dairyPagerank", validateRelationships: false, 
parameters: {category: "Dairy", cutoff: 1} 
}) 
YIELD nodePropertiesWritten, createMillis, computeMillis, writeMillis, ranIterations 
RETURN nodePropertiesWritten, createMillis, computeMillis, writeMillis, ranIterations 
 
 
 
 
 
 
 
// Compute percentiles 
MATCH (u:Customer) WHERE exists(u.dairyPagerank) return avg(u.dairyPagerank) as ave, apoc.math.round(percentileDisc(u.dairyPagerank, 0.5), 2) AS `50%`, apoc.math.round(percentileDisc(u.dairyPagerank, 0.75), 2) AS `75%`, apoc.math.round(percentileDisc(u.dairyPagerank, 0.90), 2) AS `90%`, apoc.math.round(percentileDisc(u.dairyPagerank, 0.95), 2) AS `95%`, apoc.math.round(percentileDisc(u.dairyPagerank, 0.99), 2) AS `99%`, apoc.math.round(percentileDisc(u.dairyPagerank, 1.0), 2) AS `100%` 
 
 
 
// Check for page rank greater than 90%(1.86) 
MATCH (u:Customer) 
WHERE exists(u.dairyPagerank) 
MATCH (u:Customer) 
WHERE u.dairyPagerank > 0.87 
WITH u ORDER BY u.dairyPagerank DESC 
LIMIT 10 
RETURN u.name AS name, 
       u.dairyPagerank AS pageRank,         size((u)-[:WROTE]->()) AS totalReviews, 
 
 
Now, taking 75%, we get 4 results. As we can see our percentile change slows down after 50% and remains the same once it crosses 90%, which means we do not have many users in that percentile group: 
 
  
 
Let’s us now see pagerank > 50% and take the top 10 users from that group: 
 
 
// Check for page rank greater than 0.87 
MATCH (u:Customer) 
WHERE exists(u.dairyPagerank) 
MATCH (u:Customer) 
WHERE u.dairyPagerank > 0.87 
WITH u ORDER BY u.dairyPagerank DESC 
LIMIT 10 
 
RETURN u.name AS name, 
       u.dairyPagerank AS pageRank,  
 
It can be inferred from the above result that the customer called Bat Man has the highest dairypagerank. Which means among all the customers who wrote reviews for the subcategory Dairy, Bat Man was the most influential among them all. Using this the admin can target similar people (top 10 customers sorted based on dairyPageRank) and offer them deals to keep them attracted towards the website. 
 
 
2. Betweenness:  
 
We create the graph as follows: 
 
 
CALL gds.graph.create('myGraph3', 'Customer', 'FRIENDS_WITH') 
 
 
  
 
 
CALL gds.betweenness.write.estimate('myGraph3', { writeProperty: 'betweenness' }) 
YIELD nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory 
RETURN nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory 
 
 
  
 
 
CALL gds.betweenness.write.estimate('myGraph3', { writeProperty: 'betweenness', concurrency: 1 }) 
YIELD nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory 
RETURN  nodeCount, relationshipCount, bytesMin, bytesMax, requiredMemory 
 
 
  
 
 
CALL gds.betweenness.stream('myGraph3') 
YIELD nodeId, score 
RETURN gds.util.asNode(nodeId).name AS name, score 
ORDER BY score DESC 
 
 
  
 
 
CALL gds.betweenness.stats('myGraph3') 
YIELD minimumScore, maximumScore, scoreSum 
RETURN minimumScore, maximumScore, scoreSum 
 
 
  
 
 
CALL gds.betweenness.stream('myGraph1') 
YIELD nodeId, score 
Where exists(gds.util.asNode(nodeId).dairyPagerank) RETURN gds.util.asNode(nodeId).name AS name, score, gds.util.asNode(nodeId).dairyPagerank as pageRank ORDER BY score Desc LIMIT 10 
 
 
  
 
Since gds.alpha did not support our versions of Neo4J, we followed the instructions present in the website: 
https://neo4j.com/docs/graph-data-science/current/algorithms/betweennesscentrality/  
 
We faced several issues during the execution. While creating myGraph it only allowed us to use one node and one relation. Hence betweenness is calculated for customers as node and HAS_FRIEND as the relation. From the results obtained it can be seen that the customer Arunabh Saikia has the best betweenness even though he doesn’y have the best page rank. 
 
 
Scenarios we found useful for analytics using neo4j Knowledge Graphs: 
 
1.	Stores with most products are useful in order to keep track of the count of all products  available in the store. 

2.	For the development of store and understanding people’s approach for delivery methods at different times, this analytic is useful. It provides stores which had the most number pickups out of the order placed for the store. 
 
3.	This helps to find the customer with the most number of transactions. 
 
 
4.	This analytic investigates those users who bought a product after a friend shared the product to them. This type of analytics can become useful for advertising and promotions for Grocery Hub. 
 
 
5.	This provides the most sold products with less rating (rating <=3).  
  
 
6.	Provided top 3 users who has most reviews and shares 
  
 
Market Basket Analysis: Market Basket Analysis is one of the key techniques used by retailers to uncover associations between items bought together. It works by looking for combinations of items that occur together frequently in transactions. To put it another way, it allows us to identify relationships between the items that people buy together most frequently and then suggest products to users based on what they have in cart.   	Our implementation for market basket analysis is using the aproiri algorithm and the code implementation can be found in `../EAT-FRESH/mbaScript.py`. 
 
How It Works: When a user adds a product to their cart, we check in our market basket table which stores associations between products that are bought together. We only recommend products above a certain confidence level (The confidence level simply says how much likely a user is to by a product when they add  another product based on previous behavior). An admin can recalculate the associations from their account. On recalculating, it will fetch all the transactions from the database. 
 
Trending Deals: The code for twitter matches can be found on ‘../TwitterMatches.ipynb’ 
