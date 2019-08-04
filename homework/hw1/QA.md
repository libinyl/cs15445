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


```sql
-- dates, avg_duration
-- dates: 所有在 trip 中出现过的 start_time,end_time,去重。union 会自动去重。
-- avg_duration: sum(当天所有行程的所有有效时间)/count(bike_id<=100)
-- 当天所有行程的有效时间：v_end_time-v_start_time
-- 有效结束时间：v_end_time = min(end_time, dates+1 day)
-- 有效开始时间：v_start_time = max(start_time, dates)
with date_tbl as (select date(start_time) as date_col
                  from trip
                  union
                  select date(end_time) as date_col
                  from trip)
    select
     date_col,
     round(sum(strftime('%s', min(datetime(end_time), datetime(date_col, '+1 day'))) -
               strftime('%s', max(datetime(start_time), datetime(date_col)))) * 1.0 /
           (select count(1) from trip where bike_id <= 100), 4)
         as avg_duration
    from
     trip,
     date_tbl
    where
     trip.bike_id <= 100
         and datetime(trip.start_time) < datetime(date_tbl.date_col, '+1 day')
         and datetime(trip.end_time) > datetime(date_tbl.date_col)
    group by
     date_col
    order by
     avg_duration desc limit
     10;

```

## Q6 OVERLAPPING TRIPS

One of the possible data-entry errors is to record a bike as being used in two different trips, at the same time. Thus, we want to spot pairs of overlapping intervals (start time, end time). To keep the output manageable, we ask you to do this check for bikes with id between 100 and 200 (both inclusive). Note: Assume that no trip has negative time, i.e., for all trips, start time <= end time.
Details: For each conflict (a pair of conflict trips), print the bike id, former trip id, former start time, former end time, latter trip id, latter start time, latter end time. Sort by bike id (increasing), break ties with former trip id (increasing) and then latter trip id (increasing).

Hint: (1) Report each conflict pair only once, so that former trip id < latter trip id. (2) We give you the (otherwise tricky) condition for conflicts: start1 < end2 AND end1 > start2

## Q7 MULTI CITY BIKES

- Find all the bikes that have been to more than one city. 
- A bike has been to a city as long as the start station or end station in one of its trips is in that city.
 
- Details: 
  - For each bike that has been to more than one city, print the bike id and the number of cities it has been to.
  - Sort by the number of cities (decreasing), then bike id (increasing).

## Q8 BIKE_POPULARITY_BY_WEATHER

- Find what is the average number of trips made per day on each type of weather day. 
- The type of weather on a day is specified by weather.events, such as 'Rain', 'Fog' and so on. 
- For simplicity, we consider all days that does not have a weather event (weather.events = '\N') as a single type of weather. 
- Here a trip belongs to a date only if its start time is on that date. 
- We use the weather at the starting position of that trip as its weather type as well. 
- There are also 'Rain' and 'rain' in weather.events. For simplicity, we consider them as different types of weathers.
- When counting the total number of days for a weather, we consider a weather happened on a date as long as it happened in at least one region on that date.

- Details: 
  - Print the name of the weather and the average number of trips made per day on that type of weather (round to four decimal places using ROUND()). 
  - Sort by the average number of trips (decreasing), then weather name (increasing).

## Q9 TEMPERATURE_SHORTER_TRIPS
- A short trip is a trip whose duration is <= 60 seconds. 
- Compute the average temperature that a short trip starts versus the average temperature that a non-short trip starts. 
- We use weather.mean_temp on the date of the start time as the Temperature measurement.

- Details: 
  - Print the average temperature that a short trip starts and the average temperature that a non-short trip starts. (on the same row, and both round to four decimal places using ROUND()) Please refer to the updated note before Q1 when calculating the duration of a trip.

## Q10 RIDING_IN_STORM

- For each zip code that has experienced 'Rain-Thunderstorm' weather, find the station that has the most number of trips in that zip code under the storm weather. 
- For simplicity, we only consider the start time of a trip when deciding the station and the weather for that trip.

- Details: Print the zip code that has experienced the 'Rain-Thunderstorm' weather, the name of the station that has the most number of trips under the strom weather in that zip code, and the total number of trips that station has under the storm weather. 
- Sort by the zip code (increasing). You do not need to print the zip code that has experienced 'Rain-Thunderstorm' weather but no trip happens on any storm day in that zip code.