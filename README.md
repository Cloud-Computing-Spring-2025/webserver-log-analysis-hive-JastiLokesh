# Web-Server-Log-Analysis

## Project Overview
This project is designed to analyze web server logs using Apache Hive, providing valuable insights into website performance, user behavior, and error rates. The data is processed through Hive queries, and various analysis tasks are performed, including identifying frequent page requests, status code distributions, and the user agents most frequently accessing the site. The purpose of this project is to demonstrate the power of Hive for handling and analyzing large datasets, specifically web server logs.

## Implementation Approach

The analysis is performed using the following queries:

Count Total Web Requests: Counts the total number of requests received.

Analyze Status Codes: Identifies the frequency of HTTP status codes.

Identify Most Visited Pages: Extracts the top three most visited URLs.

Traffic Source Analysis: Identifies the most common user agents (browsers).

Detect Suspicious Activity: Finds IP addresses with more than three failed requests (404 or 500 errors).

Analyze Traffic Trends: Computes the number of requests per minute to observe traffic patterns.

## Steps to Execute
### Create table
```bash
CREATE EXTERNAL TABLE IF NOT EXISTS new_web_server_logs (
    ip STRING,
    timestamp1 STRING,  
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/web_logs/';
```

### Loading data
```bash
LOAD DATA INPATH '/user/hue/web_server_logs.csv' INTO TABLE web_server_logs;
```

###  Count Total Web Requests: Calculate the total number of requests processed.
```bash
SELECT CONCAT('Total Web Requests: ', COUNT(*)) AS total_requests
FROM web_server_logs;
```

###  Analyze Status Codes: Determine the frequency of HTTP status codes (e.g., 200, 404, 500).
```bash
SELECT status, COUNT(*) AS count
FROM web_server_logs
GROUP BY status
ORDER BY count DESC;
```

###  Identify Most Visited Pages: Extract the top 3 most visited URLs.
```bash
SELECT url, COUNT(*) AS count
FROM web_server_logs
GROUP BY url
ORDER BY count DESC
LIMIT 3;
```

### Traffic Source Analysis: Identify the most common user agents (browsers).
```bash
SELECT user_agent, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY user_agent
ORDER BY request_count DESC
LIMIT 3;
```

###  Detect Suspicious Activity: Identify IP addresses with more than 3 failed requests (status 404 or 500)
```bash
SELECT ip, COUNT(*) AS failed_requests
FROM web_server_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING COUNT(*) > 3;
```


###  Analyze Traffic Trends: Calculate the number of requests per minute to observe traffic patterns.
```bash
SELECT SUBSTRING(timestamp1, 1, 16) AS minute_time, COUNT(*) AS request_count
FROM web_server_logs
GROUP BY SUBSTRING(timestamp1, 1, 16)
ORDER BY minute_time;
```

### Implement Partitioning: Use partitioning by status code to optimize query performance.
```bash
CREATE TABLE web_logs_partitioned (
    ip STRING,
    timestamp STRING,
    url STRING,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```
## Challenges Faced

During the implementation of the **Hive-based Web Log Analysis**, several challenges were encountered:

### 1 **Data Loading Issues**
- Initial attempts to load the log data into Hive resulted in **empty tables** due to incorrect delimiter settings.
- **Solution:** Verified the delimiter format (`FIELDS TERMINATED BY ','`) and ensured the data file matched the table schema.

### 2 **Dynamic Partitioning Error**
- Encountered the error:  
  ```shell
  FAILED: SemanticException [Error 10096]: Dynamic partition strict mode requires at least one static partition column.
- **Solution:** Enabled non-strict dynamic partitioning using:
  ```shell
  SET hive.exec.dynamic.partition = true;
  SET hive.exec.dynamic.partition.mode = nonstrict;
  ```



### Sample Input Data (web_server_logs.csv)
```bash
ip,timestamp1,url,status,user_agent
192.168.1.1,2024-02-01 10:15:30,/home,200,Mozilla/5.0
192.168.1.2,2024-02-01 10:15:45,/products,200,Chrome/90.0
192.168.1.3,2024-02-01 10:16:10,/home,200,Safari/13.1
192.168.1.4,2024-02-01 10:16:30,/checkout,404,Mozilla/5.0
192.168.1.5,2024-02-01 10:16:50,/home,200,Chrome/90.0
192.168.1.10,2024-02-01 10:17:05,/products,500,Mozilla/5.0
192.168.1.15,2024-02-01 10:17:20,/checkout,500,Safari/13.1
```

### Expected Output Examples

**Total Web Requests:**
```bash 
Total Requests: 7  
```

**Status Code Analysis:**
```bash
200: 4  
404: 1  
500: 2  
```

**Identify Most Visited Pages**
**Most Visited Pages:**
```bash
/home: 3  
/products: 2  
/checkout: 2  
```

**Traffic Source Analysis:**
```bash
Mozilla/5.0: 3  
Chrome/90.0: 2  
Safari/13.1: 2  
```

**Suspicious IP Addresses:**
```bash
192.168.1.10: 1 failed request  
192.168.1.15: 1 failed request  
```

**Traffic Trend Over Time:**
```bash
2024-02-01 10:15: 2 requests  
2024-02-01 10:16: 3 requests  
2024-02-01 10:17: 2 requests
```
### Output Screenshots

**Creating table**

![Screenshot 2025-02-25 134730](https://github.com/user-attachments/assets/6c749995-295f-4f25-85ad-eec4dcc99dd9)


**Total Web Requests:**

![Screenshot (1)](https://github.com/user-attachments/assets/1864980e-e9ba-4900-8a4b-ed1ddf6df559)


**Status Code Analysis:**

![Screenshot (2)](https://github.com/user-attachments/assets/85ea6574-992f-4d14-83c8-3ebb13792df0)


**Identify Most Visited Pages**

![Screenshot (3)](https://github.com/user-attachments/assets/d969119c-b9ae-4f11-890c-0fb0eddd4032)


**Traffic Source Analysis:**

![Screenshot (4)](https://github.com/user-attachments/assets/efa0a36e-3cc1-4979-afc8-3729974d56c2)


**Suspicious IP Addresses:**

![Screenshot (5)](https://github.com/user-attachments/assets/b11fa195-db92-4381-8e40-90bf6556d629)


**Traffic Trend Over Time:**

![Screenshot (7)](https://github.com/user-attachments/assets/72ec0500-1977-477b-9911-db908c86ff0a)
