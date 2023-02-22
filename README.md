SQL Test using BigQuery
    
Dataset: https://drive.google.com/drive/folders/1UCTW4kUwoAu0Efj45IO1j8b0s6L3qLMO?usp=sharing

Question No. 1 -->  For each trip, get the start station’s name, longitude and latitude

     SELECT t1.trip_id, t2.name first_station_name, t2.longitude, t2.latitude FROM `galuh-378404.citibikedata.citibike_trips` t1
     left join `galuh-378404.citibikedata.citibike_stations` t2 on (
     t1.start_station_id = t2.station_id)
     
Question No. 2 --> Find out how many total trips that start from every station, sort from the highest to lowest total trips. (Show only the station's name and the total of trips)
    
    SELECT t2.name first_station_name, count(t1.trip_id) total_trips  FROM `galuh-378404.citibikedata.citibike_trips` t1
    inner join `galuh-378404.citibikedata.citibike_stations` t2 on (
    t1.start_station_id = t2.station_id) group by first_station_name
    order by total_trips desc

Question No. 3 --> Find out how many total trips per GENDER that start the trip from station id 236

    SELECT t1.gender, t2.start_station_id, count(t2.trip_id) total_trips FROM `galuh-378404.citibikedata.citibike_users` t1
    inner join `galuh-378404.citibikedata.citibike_trips` t2 on (
    t1.userid = t2.userid
    )
    group by gender, start_station_id having start_station_id = 236
    
Question No. 4 --> Find out which start stations have more than 600 total trips duration
    
    SELECT t1.name, sum(t2.tripduration) total_duration FROM `galuh-378404.citibikedata.citibike_stations` t1
    inner join `galuh-378404.citibikedata.citibike_trips` t2 on (
    t1.station_id = t2.start_station_id
    )
    group by name having total_duration > 600
    
Question No. 5 --> Find the top 3 longest trip duration in terms of every trip per start station. Do not use UNION as we do not want to query the table multiple times
    
    WITH cte AS
    (SELECT ID_trip, ID_start_stations, start_station_name, trip_duration, 
    ROW_NUMBER() OVER(PARTITION BY ID_trip, start_station_name ORDER BY trip_duration DESC) AS rn 
    FROM (select t1.trip_id ID_trip, t1.start_station_id ID_start_stations, t2.name start_station_name, t1.tripduration trip_duration from `galuh-378404.citibikedata.citibike_trips` t1 
    inner join `galuh-378404.citibikedata.citibike_stations` t2 on t1.start_station_id =t2.station_id group by ID_trip, ID_start_stations, 
    start_station_name, trip_duration)), cte2 AS
    (SELECT cte.ID_trip , cte.ID_start_stations, cte.start_station_name, cte.trip_duration, 
    ROW_NUMBER() OVER(PARTITION BY cte.start_station_name ORDER BY cte.trip_duration desc) AS rn2
    FROM cte
    WHERE rn = 1
    )
    SELECT ID_trip, ID_start_stations, start_station_name, trip_duration
    FROM cte2
    WHERE rn2 <= 3
