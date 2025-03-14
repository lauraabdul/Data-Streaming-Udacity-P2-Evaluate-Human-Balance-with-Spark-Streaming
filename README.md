# STEDI Ecosystem

## Overview
You work for the data science team at STEDI, a small startup focused on assessing balance for seniors. STEDI has an application that collects data from seniors during a small exercise. The user logs in, selects the customer they are working with, starts a timer, and clicks a button with each step the senior takes. When the senior has reached 30 steps, their test is finished. The data transmitted enables the application to monitor seniors’ balance risk.

## Setting Up the Workspace
### Cloning the GitHub Repository
Clone the repository to set up your local workspace:
```sh
git clone git@github.com:udacity/nd029-c2-apache-spark-and-spark-streaming-starter.git
```

### Using Docker
Docker is required to run the project locally. Install Docker from: [Get Docker](https://docs.docker.com/get-docker/).

It is recommended to configure Docker to use up to 2 cores and 6 GB of memory.

### Start the Docker Workspace
From the root of the repository folder, run:
```sh
cd [repositoryfolder]
docker-compose up
```
Ensure all containers are running (you should see 9 processes):
```sh
docker ps
```

## STEDI Application
Log in to the STEDI application:
[STEDI Application](http://localhost:4567)

### Creating a Test Customer
1. Click **Create New Customer**
2. Enter test customer details and submit
3. Click **Start**, then add steps until reaching 30
4. Repeat this three times to receive a risk score

## Analyzing the Data
The STEDI data science team has configured real-time data sources using Kafka Connect. Redis is one of these sources, running in a Docker container on the default port (6379). Whenever customer data is saved to Redis, a payload is published to the Kafka topic `redis-server`.

### Connecting to Redis
```sh
docker exec -it evaluate-human-balance-with-spark-streaming_redis_1 redis-cli
```
View customers stored in Redis:
```sh
zrange customer 0 -1
```

### Monitoring Kafka Topics
To monitor the `redis-server` Kafka topic:
```sh
docker exec -it evaluate-human-balance-with-spark-streaming_kafka_1 kafka-console-consumer --bootstrap-server localhost:9092 --topic redis-server
```
Manually add a test customer in Redis:
```sh
zadd Customer 0 '{"customerName":"Sam Test","email":"sam.test@test.com","phone":"8015551212","birthDay":"2001-01-03"}'
```
You should see the payload appear in the Kafka consumer terminal.

## The Challenge
The development team has programmed business events to be published to Kafka. When a customer completes four or more assessments, a risk score is generated and transmitted to the `stedi-events` Kafka topic as a JSON object:
```json
{
  "customer": "Jason.Mitra@test.com",
  "score": 7.0,
  "riskDate": "2020-09-14T07:54:06.417Z"
}
```
Currently, the STEDI graph is not receiving any data. You need to generate a new payload in a Kafka topic and make it available to the STEDI application to consume.

### Creating a New Kafka Topic
Edit `docker-compose.yaml` and configure the new topic:
```sh
KAFKA_RISK_TOPIC: <new-topic-name>
```
Restart Docker containers:
```sh
CTRL+C
docker-compose up
```
Log in to STEDI and activate simulated user data using the toggle button. This creates 30 new customers and starts generating risk scores. Each customer receives risk scores after 4 minutes.

Monitor data generation:
```sh
docker logs -f nd029-c2-apache-spark-and-spark-streaming_stedi_1
```

## Writing Spark Streaming Scripts
Write three Spark Python scripts to process and join Kafka data:

1. **`sparkpyrediskafkastreamtoconsole.py`**: Subscribes to `redis-server`, decodes base64 payloads, and prints fields (email, birth date).
2. **`sparkpyeventskafkastreamtoconsole.py`**: Subscribes to `stedi-events`, deserializes JSON, and extracts email and risk score.
3. **`sparkpykafkajoin.py`**: Joins customer and risk dataframes on email, then outputs JSON to the new Kafka topic:
```json
{
  "customer": "Santosh.Fibonnaci@test.com",
  "score": "28.5",
  "email": "Santosh.Fibonnaci@test.com",
  "birthYear": "1963"
}
```
Submit the script:
```sh
submit-event-kafkajoin.sh
```
Once populated, the STEDI graph should display real-time data.

## Capturing Screenshots
Upload at least two screenshots of the working graph to the `screenshots` workspace folder.

