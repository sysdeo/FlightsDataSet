`Total delayed flights by day of week`
insert into nflights (day_of_week, nday)

select day_of_week, count(*)

from flights

where arrival_delay > 0

group by day_of_week

order by day_of_week


---
`Created table delay type to count delays by department,flight number, origin, destination, time of day

create table delay_type(

arrival_delay numeric,

air_system_delay numeric,

security_delay numeric,

airline_delay numeric,

late_aircraft numeric,

weather_delay numeric,

airline text,

flight_number numeric(4),

origin_airport text,

destination_airport text);

----
`Inserting into 1 of the main tables specific columns to further process delays over 0 minutes`

insert into delay_type(arrival_delay, air_system_delay, security_delay,airline_delay,late_aircraft,weather_delay,airline,flight_number, origin_airport,destination_airport)

select arrival_delay, air_system_delay, security_delay,airline_delay,late_aircraft_delay,weather_delay,airline,flight_number, origin_airport,destination_airport

from flights

where arrival_delay > 0

---
`Creating a table showing delays with over 0 for 6 sections of the day including:`
`Midnight, Dawn, Morning, Noon, Afternoon, Dusk`


CREATE TABLE six_way_delays (
    id SERIAL PRIMARY KEY,
    airline TEXT,
    flight_number TEXT,
    origin_airport TEXT,
    destination_airport TEXT,
    delay_minutes INT,
    time_of_day TEXT
    delay_count_at_destination INTEGER;
);

-- Populate the table with segmented data
INSERT INTO six_way_delays (airline, flight_number, origin_airport, destination_airport, delay_minutes, time_of_day)
SELECT 
    airline, 
    flight_number, 
    origin_airport, 
    destination_airport, 
    arrival_delay,
    CASE
        WHEN EXTRACT(HOUR FROM scheduled_departure) BETWEEN 0 AND 3 THEN 'Midnight to Dawn'
        WHEN EXTRACT(HOUR FROM scheduled_departure) BETWEEN 4 AND 7 THEN 'Dawn to Morning'
        WHEN EXTRACT(HOUR FROM scheduled_departure) BETWEEN 8 AND 11 THEN 'Morning to Noon'
        WHEN EXTRACT(HOUR FROM scheduled_departure) BETWEEN 12 AND 15 THEN 'Noon to Afternoon'
        WHEN EXTRACT(HOUR FROM scheduled_departure) BETWEEN 16 AND 19 THEN 'Afternoon to Dusk'
        WHEN EXTRACT(HOUR FROM scheduled_departure) BETWEEN 20 AND 23 THEN 'Dusk to Midnight'
    END
FROM 
    flights
WHERE 
    arrival_delay > 0;





----

` Populating the table with segmented data`

INSERT INTO six_way_delays (airline, flight_number, origin_airport, destination_airport, delay_minutes, time_of_day)

SELECT

airline,

flight_number,

origin_airport,

destination_airport,

arrival_delay,

CASE

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 0 AND 3 THEN 'Midnight to Dawn'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 4 AND 7 THEN 'Dawn to Morning'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 8 AND 11 THEN 'Morning to Noon'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 12 AND 15 THEN 'Noon to Afternoon'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 16 AND 19 THEN 'Afternoon to Dusk'

WHEN EXTRACT(HOUR FROM TIME '00:00' + ((scheduled_departure / 100) * interval '1 hour') + ((scheduled_departure % 100) * interval '1 minute')) BETWEEN 20 AND 23 THEN 'Dusk to Midnight'

END

FROM

flights

WHERE

arrival_delay > 0;`

----
``

UPDATE six_way_delays
SET delay_count_at_destination = subquery.delay_count
FROM (
    SELECT destination_airport, COUNT(*) as delay_count
    FROM six_way_delays
    GROUP BY destination_airport
) AS subquery
WHERE six_way_delays.destination_airport = subquery.destination_airport;

---

`rounding up numbers to limit to a singular decimal (eg: 26.1, 36.2)`

UPDATE seasonal_delays

SET spring = ROUND(CAST(spring AS numeric), 1);

----
