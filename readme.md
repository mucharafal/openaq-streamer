# OpenAQ streamer

## Build

- Build docker image
```bash
make image
```

- Run docker
```bash
docker run --rm -p 8888:8888 -p 4040:4040 -v mountPoint:/home/jovyan/work balis/openaq-streamer
```
## Usage
1. Create an AWS SQS queue and subscribe to SNS topic `arn:aws:sns:us-east-1:470049585876:OPENAQ_NEW_MEASUREMENT`. 

2. To reduce the volume of data, go to the created SNS subscription and set up a filter policy, e.g.:
```js
{
  "country": [
    "PL"
  ]
}
```

2. Run spark structured streaming query. 
Python example:
```python
# read AWS credentials from a csv file (as downloaded when creating the keys)
# first row = header, second row = keys
# Alternatively, environment variables can be used via 'docker --env AWS_ACCESS_KEY=...'

import csv
with open('accessKeys.csv', newline='') as csvfile:
    awsreader = csv.reader(csvfile, delimiter=',', quotechar='|')
    for row in awsreader:
        ;
        
accessKey=row[0]
secretKey=row[1]

from pyspark.sql import SparkSession
spark = SparkSession\
    .builder\
    .config("spark.sql.streaming.schemaInference", True)\
    .getOrCreate()

stream = spark\
    .readStream\
    .format("sqs")\
    .option("queueUrl", "https://sqs.us-east-1.amazonaws.com/...")\  # insert your SQS queue URL
    .option("accessKey", accessKey)\
    .option("secretKey", secretKey)\
    .option("region", "us-east-1")\
    .load()

stream.select("city",  "parameter", "value", "date.local").writeStream\ 
    .format("console")\
    .outputMode("append")\
    .start()
```
