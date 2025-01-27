# Module 1 Homework: Docker & SQL

## Question 1. Understanding docker first run 

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

### Answers 
The code to run docker by using bash is: 
```bash
docker run -it --entrypoint=bash python:3.12.8

``` 
To check the pip version: 
```bash
pip --version

``` 
The answer is **24.3.1**

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

### Answer
Service name and port 5432 inside the container: **db:5432**


##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.


### docker-compose.yml
```bash
docker-compose up
```
### Dockerfile

```bash
docker build -t taxi_ingest:v001 .
```

### ingest_data.py

```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker run -it \
--network=alxzoomcamp2025_default \
taxi_ingest:v001 \
  --user=root \
  --password=root \
  --host=pgdatabase \
  --port=5432 \
  --db=ny_taxi \
  --table_name=green_tripdata\
  --url=${URL}
```



## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles 

### Answer
```sql
SELECT 
	SUM(CASE WHEN trip_distance <= 1 THEN 1 ELSE 0 END) AS Col1,
	SUM(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 ELSE 0 END) AS Col2,
	SUM(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 ELSE 0 END) AS Col3,
	SUM(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 ELSE 0 END) AS Col4,
	SUM(CASE WHEN trip_distance > 10 THEN 1 ELSE 0 END) AS Col5
FROM
	public.green_tripdata
WHERE
    DATE(lpep_pickup_datetime) >='2019-10-01' AND DATE(lpep_dropoff_datetime) < '2019-11-01';

```

**104802	198924	109603	27678	35189**. 


## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

### Answer

```sql
SELECT 
	lpep_pickup_datetime,
	CASE WHEN DATE(lpep_pickup_datetime) = DATE(lpep_dropoff_datetime) THEN trip_distance ELSE 0 END AS trip_distance
FROM
	public.green_tripdata
ORDER BY trip_distance	DESC
LIMIT 1
```

**2019-10-11 20:34:21	 |  95.78**

## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.

### Answer

```sql
SELECT 
	SUM(t.total_amount),
	z."Zone"
FROM public.green_tripdata t
Inner JOIN public.zones z ON "PULocationID" = "LocationID"
WHERE DATE(lpep_pickup_datetime) = '2019-10-18'
GROUP BY z."Zone"
HAVING SUM(t.total_amount) > 13.000
ORDER BY SUM(t.total_amount) DESC

```
**East Harlem North, East Harlem South, Morningside Heights**


## Question 6. Largest tip

For the passengers picked up in Ocrober 2019 in the zone
name "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

### Answer 
 
```sql
SELECT 
	t.tip_amount,
	z1."Zone" as DropOff
FROM public.green_tripdata t
Inner JOIN (SELECT * FROM public.zones WHERE "Zone" = 'East Harlem North') z ON t."PULocationID" = z."LocationID"
Inner JOIN public.zones z1 ON t."DOLocationID" = z1."LocationID"
ORDER BY t.tip_amount DESC
LIMIT 1;
```

**"JFK Airport"**


## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](../../../01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for: 
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answer:
- terraform init, terraform apply -auto-aprove, terraform destroy


