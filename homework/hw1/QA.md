## 下载链接

`bike_sharing.db`
```
wget https://15445.courses.cs.cmu.edu/fall2018/files/bike_sharing.tar.gz
```

## Q1 Count the number of cities.
```sql
sqlite> select count(distinct(city)) from station;
count(distinct(city))
---------------------
5
```
## Q2 Count the number of stations in each city.

Details: Print city name and number of stations. Sort by number of stations (increasing), and break ties by city name (increasing).

```sql
sqlite> select city,count(station_id) as count from station group by city order by count asc,city asc;
city        count
----------  ----------
Palo Alto   5
Mountain V  7
Redwood Ci  7
San Jose    16
San Franci  35
```

## Q3 Find the percentage of trips in each city. 

A trip belongs to a city as long as its start station or end station is in the city. 

For example, if a trip started from station A in city P and ended in station B in city Q, then the trip belongs to both city P and city Q. If P equals to Q, the trip is only counted once.

Details: 
- Print city name and ratio between the number of trips that belong to that city against the total number of trips (a decimal between 0-1, round to four decimal places using ROUND()). 
- Sort by ratio (decreasing), and break ties by city name (increasing).



```sql
-- trip 总数 :
select count(1) from trip
-- city 在 station,数量在 trip 必有级联.联系条件是 station 的 station_id 要等于 start_station_id 或者 end_station_id.这个条件已经符合了题目定义.
select * from station,trip where station_id = start_station_id or station_id = end_station_id
-- 得到每个 city 的 trip 总数.注意级联后 id 可能重复,所以作和时要去重.
select city,count(distinct(id)) from station,trip where station_id = start_station_id or station_id = end_station_id group by city
-- 然后把 trip 总数与其级联
select * from (select city,count(distinct(id)) from station,trip where station_id = start_station_id or station_id = end_station_id group by city),(select count(1) from trip)
-- 得到最终结果
select city, round(trip_count * 1.0 / trip_total, 4) as ratio
from (select city, count(distinct (id)) as trip_count
      from station
               inner join trip t on station.station_id = t.end_station_id or station.station_id = t.start_station_id
      group by city) as city_trip_count,
     (select count(1) as trip_total from trip) as total
order by ratio desc,
         city asc;
```

## Q4 For each city, find the most popular station in that city.

 "Popular" means that the station has the highest count of visits. As above, either starting a trip or finishing a trip at a station, the trip is counted as one "visit" to that station. The trip is only counted once if the start station and the end station are the same.

Details: For each station, print city name, most popular station name and its visit count. Sort by city name, ascending.

```sql
select city, station_name, max(station_trip_count) as visit_count
from (select city, station_name, count(distinct (id)) as station_trip_count
      from station
               inner join trip t on station.station_id = t.end_station_id or station_id = t.start_station_id
      group by station_name) group by city order by city asc;
```

## Q5: Find the top 10 days that have the highest average bike utilization.

找到平均自行车利用率最高的十天.

- For simplicity, we only consider trips that use bikes with id <= 100. 
- The average bike utilization on date D is calculated as the sum of the durations of all the trips that happened on date D divided by the total number of bikes with id <= 100, which is a constant. 
- 日期 D 的平均自行车利用率 = 日期 D 的所有车的所有行程时间之和/id<=100 的自行车总数(常数).
- If a trip overlaps with date D, but starts before date D or ends after date D, then only the interval that overlaps with date D (from 0:00 to 24:00) will be counted when calculating the average bike utilization of date D. 
- 如果一段行程与日期 D 重合了,但在日期 D 之前开始,或者在日期 D 之后结束,那么只有与日期 D 重合的部分用于日期 D 的平均自行车利用率的计算.
- And we only calculate the average bike utilization for the date that has been either a start or an end date of a trip.
- 只计算行程开始或结束日期的平均自行车利用率.
- You can assume that no trip has negative time (i.e., for all trips, start time <= end time).
- 可以假设没有行程存在负值时间.
- Details: 
  - For the dates with the top 10 average duration, print the date and the average bike duration on that date (in seconds, round to four decimal places using the ROUND() function). 
  - Sort by the average duration, decreasing. 
