---
layout: page
title: Real time database for flight data
img: assets/img/flight.webp
importance: 3
category: Cloud Computing and Big Data
---

## Introduction
Flight information plays a crucial role in the aviation industry, encompassing vast data that provides valuable insights into an aircraft's location and status during its journey. This includes departure and arrival times, flight numbers, aircraft types, routes, and current positions. Such data is essential for airlines, airports, air traffic control, and passengers alike, as it facilitates the smooth operation of flights, ensures safety oversight, and enables efficient travel planning.

To meet the growing need for real-time flight information, we have developed a distributed database system tailored to extract and analyze flight data efficiently. Our goal is to create intuitive and user-friendly web interfaces that make it easy for ordinary users and database managers to access and manage real-time flight information. Our system will provide a comprehensive and reliable data source, enabling aviation professionals to make informed decisions and enhance air travel's efficiency and safety.

## Planned Implementation
The original implementation of this project aimed to build a distributed database architecture that would store various attributes of flights, including flight numbers, airline details, departure and arrival times, current flight status, and geographical information such as departure and arrival airports and flight paths. 

Since airline data follows a fixed schema, we decided to structure this data into multiple tables such as flight, airline, departure, etc. inside a relational database like MySQL Furthermore, to optimize data management within the relational database, we planned to partition each table across multiple MySQL hosts. This partitioning strategy would distribute the data evenly among hosts, ensuring efficient data management.

The data partitioning strategy in the relational database was essential for efficient data management across multiple hosts. Each table needed to be replicated and partitioned across MySQL hosts. To achieve this, we would create databases in each host, containing partitioned tables with corresponding data.

To maintain fair distribution of partitions across hosts, we aimed to use Apache Zookeeper to track and manage mappings. There would be two sets of directories: one for maintaining available hosts and another for mapping table-partition-host relationships.

Finally, we needed a data router whose primary function was to communicate with Zookeeper and route data to the correct partition tables in the appropriate databases on the designated MySQL hosts. 

## Architecture

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/architecture.png" title="Architecture of the Real-time Flight Database" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Architecture of the Real-time Flight Database</figcaption>
</figure>

The above architecture diagram depicts the workflow of the web application and its interaction with the database. The users and DB manager interact with the web app through different interfaces. The user uses the front-end of the application whereas the DB manager will use curl commands to access the database. However, the flow of commands and data is the same in both scenarios. Any query or command to the database first passes through the data router. A hashing function determines which partition to access in the list of hosts based on the mapping information stored in Zookeeper. Then the actual host is accessed through a MySQL Client and the data is queried. 

The data in the distributed system is populated using the data ingestor. There is a web scraper that scrapes the FlightRadar24 website and publishes the data to a kafka topic. From this topic, the consumer reads the data and routes it to the correct partition of the corresponding table.
For querying the database, the spark cluster allows us to efficiently use MySQL by processing data in parallel across the multiple nodes of the database. The integration of Spark with MySQL allows us to seamlessly retrieve and process data from the relational database.


### Partitioning of Data
<div style="text-align: center;">
    <figure class="figure text-center" style="max-width: 60%; height: auto;">
        {% include figure.liquid loading="eager" path="assets/img/projects/airline/partitioning.png" title="Architecture of the Real-time Flight Database" class="img-fluid rounded z-depth-1" %}
        <figcaption class="figure-caption">Partitioning of Data</figcaption>
    </figure>
</div>

Partitioning of data involves creating databases across all the database nodes and then creating a table for the specific partition in each of the databases. This allows an isolation and consistent manner in which partitions can be stored across database nodes.

### Bookkeeping of nodes and self discovery
<div style="text-align: center;">
    <figure class="figure text-center" style="max-width: 65%; height: auto;">
        {% include figure.liquid loading="eager" path="assets/img/projects/airline/bookkeeping.png" title="Bookkeeping of nodes and self discovery" class="img-fluid rounded z-depth-1" %}
        <figcaption class="figure-caption">Bookkeeping of nodes and self discovery</figcaption>
    </figure>
</div>

In a distributed database, it is important to keep track and monitor which database nodes are available for data to be written and also to know how many partitions the data needs to be partitioned into. To do this we have used Zookeeper as depicted above. Once a database node comes up it registers itself as a zookeper child node under the parent hosts node. The data router sets up a children watch on the hosts znode and everytime a new node comes up or an existing node goes down, this is tracked and the data router can take care of the necessary operations, like opening a database connection or closing one.

### Spark SQL for structured data processing

<div style="text-align: center;">
    <figure class="figure text-center" style="max-width: 70%; height: auto;">
        {% include figure.liquid loading="eager" path="assets/img/projects/airline/spark.png" title="Spark SQL for structured data processing" class="img-fluid rounded z-depth-1" %}
        <figcaption class="figure-caption">Spark SQL for structured data processing</figcaption>
    </figure>
</div>

In our use case, we would like to solve for complex aggregation queries which might require data across multiple database nodes. While partitioning the data has its pros, this particularly proves a challenge for the use case we are trying to solve. 

To solve this problem, we use spark to create a logical view on top of the data across all the databases and use spark to fetch and aggregate this data. This provides 2 advantages:
1. Performance: A separate spark cluster takes care of the actual connection to all the nodes, executing the query and aggregating the results. This means that this cluster takes all the computation burden away from the data router and can be scaled independently.
2. Query Execution: Since spark creates a logical view on top of all the data, the query provided by the user can be used as is without any modifications and exposure to the underlying details.

## Functionalities
The use case for our project is to track data of passenger airline flights near real time. There are two categories of users who would like to use such a system:
1. End user: End users would be interested in seeing which flights are out of which airports, the location or altitude of the plane etc.
2. Database manager: This advanced user would be interested in setting up the ingestion of the data into the database so that the end user can see the data.

End User Functionalities
1. Flights currently en route - This functionality allows the user to find flights from which are currently in the air with their origin airport and destination airport.
2. Get flight by destination country: This functionality allows the user to find flights to which country the flights are headed to. The user can enter the country name and we would fetch which flights in the air are headed to that country.
3. Get flight by origin airport: This feature of the application allows users to search by the origin airport code, so they can see all the flights that have departed from that specific airport.
4. Get flight by destination airport: This feature of the application allows users to search by the destination airport code, so they can see all the flights that are arriving at that specific airport.
5. Get current flights in the northern hemisphere: This interesting feature filters flights which are currently flying in the northern hemisphere i.e having a latitude above zero.
6. Get current flights in the southern hemisphere: Similar to the previous feature, this feature filters flights which are currently flying in the southern hemisphere i.e having a latitude below zero.

Each of the features mentioned above, returns a list of flights. On each flight, we can click on it, to get further details of the flight such as the heading, call sign etc. Some of the key information shown are:
- Heading: The direction in which the longitudinal axis of an aircraft is pointed, usually expressed in degrees from North.
- Altitude: The height in feet of the aircraft in its current position.
- Ground Speed: Ground speed is the horizontal speed of an aircraft relative to the Earth's surface. It is measured in knots.
- Aircraft code: It is the unique code identifying the aircraft. 
- Destination airport gate: This field shows the gate of the destination airport the flight will dock at. This information is usually only available if the aircraft is near to its destination.

### Database manager Functionalities

The database manager is responsible for setting up the above functionality as well creating more functionalities. The database manager can directly interact with the underlying structure of the data powering the application - through SQL queries.

The DB manager is provided through a set of REST APIs to perform these operations. These APIs are:

#### Create Table API

The create table API is responsible for creating the tables and all the required blocks for data to be ingested into the database, as well as the metadata needed to maintain and query the data. Most importantly, it also describes the structure or the definition of the data being consumed.

```bash
curl --location 'http://<host>/database/tables' \
--header 'Content-Type: application/json' \
--data '{
   "createTableQuery": "CREATE TABLE flight_data (id INT AUTO_INCREMENT PRIMARY KEY,heading INT);",
   "partitionKey": "id",
   "tableName": "flight_data"
}'
```
<figcaption class="figure-caption text-center">Create Table API</figcaption>

This API takes the following 3 arguments:
- Create Table Query: This is the actual DDL query that the DB manager crafts.
Considering this is a real time system and an ingestion consumer is set up, it is important for the fields in this table to match the fields sent in the event over the kafka topic.
- Partition Key: This field dictates how the data stored in the database must be partitioned. This field is crucial and must be chosen correctly as the data access patterns change based on the partition key and the data we are trying to query. 
- Table Name: Name of the table to be created. It must match the name of the table in the create table query. This field is used for metadata purposes.

#### Scan Data API

The scan data API, provides a database operation to query all the data or in other words, data across partitions. Such an API is useful for data aggregation queries or summary type queries.

```bash
curl --location 'http://<host>/database/scan' \
--header 'Content-Type: application/json' \
--data '{
   "tableName": "flight_data",
   "selectQuery": "SELECT DISTINCT callsign FROM flight_data"
}'
```
<figcaption class="figure-caption text-center">Scan Data API</figcaption>

This API takes the following 2 arguments:
- Table Name: Name of the table to query from 
- Select Query: The query to be executed to fetch data from the database

#### Query Data API
The query data API, provides a database operation to query data from a specific partition. This API provides the least amount of latency and is useful when we need to fetch data  

```bash
curl --location 'http://<host>/database/query' \
--header 'Content-Type: application/json' \
--data '{
   "partitionKey": "DAL170",
   "tableName": "flight_data",
   "query": "SELECT * FROM flight_data WHERE callsign='\''DAL170'\''"
}'
```
<figcaption class="figure-caption text-center">Query Data API</figcaption>

This API takes the following 3 arguments:
- Partition Key: The key we are actually querying against. This helps the database find the right partition
- Table Name: The table to query from
- Query: The query to execute. It is important that the query here is for a specific entity in the database which is identified by the partition key mentioned above, else the query will give an empty result.

APIs used for implementing current set of end user functionalities:

| Functionality | API |
| --- | --- |
| Flights currently en route | Scan Data API |
| Flights by destination country | Scan Data API |
| Flights by origin airport | Scan Data API |
| Flights by destination airport | Scan Data API |
| Flights in the northern hemisphere | Scan Data API |
| Flights in the southern hemisphere | Scan Data API |
| ***Flight details used in all above functionalities*** | Query Data API |

## Screen captures of the application

#### Flights currently en route
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/flights_enroute.png" title="Flights currently en route" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Flights currently en route</figcaption>
</figure>

#### Flights by destination country
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/destination.png" title="Flights by destination country" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Flights by destination country</figcaption>
</figure>

#### Flights by origin airport
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/origin_airport.png" title="Flights by origin airport" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Flights by origin airport</figcaption>
</figure>

#### Flights by destination airport
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/destination_airport.png" title="Flights by destination airport" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Flights by destination airport</figcaption>
</figure>

#### Flights in the northern hemisphere
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/northern_hemisphere.png" title="Flights in the northern hemisphere" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Flights in the northern hemisphere</figcaption>
</figure>

#### Flights in the southern hemisphere
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/airline/southern_hemisphere.png" title="Flights in the southern hemisphere" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Flights in the southern hemisphere</figcaption>
</figure>

## Challenges and Conclusion

During our project, some of the challenges we ran into included finding a reliable flight data source for fetching and publishing information. This includes aircraft code, altitude, heading, etc. Many online sources were not accessible without a subscription. Once we locked in a source, FlightRadarAPI, a selenium-based script, was used to parse through the website, condense, and convert this into a format that applies to our project. We also discovered a minor amount of data loss due to a time delay from IP-based throttling. This was resolved with a retry mechanism where we wait for 5 seconds and then refresh the web page. There was also a challenge testing the code due to a lack of resource allocation, requiring multiple team members to assist in connecting the backend database manager and the frontend analyses. Due to resource allocation issues, aesthetic output was used to test the frontend systems and ensure it could take any result from the backend manager. Once this was complete, a different system was used to connect the frontend and backend sections by creating a pocket web server for the backend manager. This allowed the frontend website to talk to the backend web server using RESTful API calls.

In conclusion, we have successfully developed a real-time flight database system that can be used by both end-users and database managers. The system is designed to provide real-time flight information, including flight numbers, airline details, departure and arrival times, current flight status, and geographical information. The system is built on a distributed database architecture that stores flight data in multiple tables across multiple MySQL hosts. The data is partitioned across hosts to ensure efficient data management, and Apache Zookeeper is used to track and manage mappings. A data router communicates with Zookeeper to route data to the correct partition tables in the appropriate databases on the designated MySQL hosts. The system also includes a data ingestor that scrapes the FlightRadar24 website and publishes the data to a Kafka topic. A consumer reads the data from the topic and routes it to the correct partition of the corresponding table. The system also includes a Spark cluster that allows us to efficiently use MySQL by processing data in parallel across multiple nodes of the database. The integration of Spark with MySQL allows us to seamlessly retrieve and process data from the relational database. The system provides a comprehensive and reliable data source for aviation professionals to make informed decisions and enhance air travel's efficiency and safety.