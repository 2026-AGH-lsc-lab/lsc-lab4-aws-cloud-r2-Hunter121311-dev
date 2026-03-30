Step 3: Deploy Lambda (Zip)

[ec2-user@ip-172-31-70-137 lsc-lab4-aws-cloud-r2-Hunter121311-dev]$ aws lambda invoke --function-name lsc-knn-zip \
    --cli-binary-format raw-in-base64-out \
    --payload "$(python3 -c 'import json; q=json.load(open("loadtest/query.json")); print(json.dumps({"body": json.dumps(q)}))')" \
    /tmp/out.json
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
[ec2-user@ip-172-31-70-137 lsc-lab4-aws-cloud-r2-Hunter121311-dev]$ cat /tmp/out.json
{"statusCode": 200, "headers": {"Content-Type": "application/json", "X-Server-Time-Ms": "82.905", "X-Instance-Id": "2026/03/29/[$LATEST]19216e38226c49d88ddf13f0f9656c38", "X-Cold-Start": "false"}, "body": "{\"results\": [{\"index\": 35859, \"distance\": 12.001459121704102}, {\"index\": 24682, \"distance\": 12.059946060180664}, {\"index\": 35397, \"distance\": 12.487079620361328}, {\"index\": 20160, \"distance\": 12.489519119262695}, {\"index\": 30454, \"distance\": 12.499402046203613}], \"query_time_ms\": 82.905, \"instance_id\": \"2026/03/29/[$LATEST]19216e38226c49d88ddf13f0f9656c38\", \"cold_start\": false}"}[ec2-user@ip-172-31-70-137 lsc-lab4-aws-cloud-r2-Hunter121311-dev]$


Step 4: Deploy Lambda (Container)

[ec2-user@ip-172-31-70-137 lsc-lab4-aws-cloud-r2-Hunter121311-dev]$ aws lambda invoke --function-name lsc-knn-container \
    --cli-binary-format raw-in-base64-out \
    --payload "$(python3 -c 'import json; q=json.load(open("loadtest/query.json")); print(json.dumps({"body": json.dumps(q)}))')" \
    /tmp/out.json
cat /tmp/out.json
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
{"statusCode": 200, "headers": {"Content-Type": "application/json", "X-Server-Time-Ms": "84.173", "X-Instance-Id": "2026/03/29/[$LATEST]65cedb290b2d487e8c13583a2f98a33c", "X-Cold-Start": "true"}, "body": "{\"results\": [{\"index\": 35859, \"distance\": 12.001459121704102}, {\"index\": 24682, \"distance\": 12.059946060180664}, {\"index\": 35397, \"distance\": 12.487079620361328}, {\"index\": 20160, \"distance\": 12.489519119262695}, {\"index\": 30454, \"distance\": 12.499402046203613}], \"query_time_ms\": 84.173, \"instance_id\": \"2026/03/29/[$LATEST]65cedb290b2d487e8c13583a2f98a33c\", \"cold_start\": true}"}


Step 5: Deploy Fargate

 curl -X POST -H "Content-Type: application/json" -d @loadtest/query.json     http://lsc-knn-alb-1905191287.us-east-1.elb.amazonaws.com/search
{"instance_id":"ip-172-31-66-136.ec2.internal","query_time_ms":23.751,"results":[{"distance":12.001459121704102,"index":35859},{"distance":12.059946060180664,"index":24682},{"distance":12.487079620361328,"index":35397},{"distance":12.489519119262695,"index":20160},{"distance":12.499402046203613,"index":30454}]}

Step 6: Deploy EC2 App Instance - verivication
curl -X POST -H 'Content-Type: application/json' -d @loadtest/query.json http://3.209.82.211:8080/search
{"instance_id":"f1276dfd871a","query_time_ms":30.991,"results":[{"distance":12.001459121704102,"index":35859},{"distance":12.059946060180664,"index":24682},{"distance":12.487079620361328,"index":35397},{"distance":12.489519119262695,"index":20160},{"distance":12.499402046203613,"index":30454}]}

Scenariusz A

zip
aws logs filter-log-events     --log-group-name "/aws/lambda/lsc-knn-zip"     --filter-pa
ttern "Init Duration"     --query 'events[*].message' --output text
REPORT RequestId: 7b3a9a1d-2d61-4db8-a568-93514ec85c00  Duration: 2.32 ms       Billed Duration: 568 ms Memory Size: 512 MB     Max Memory Used: 143 MB Init Duration: 565.67 ms
XRAY TraceId: 1-69c97e0c-37c1869e7aeed16d5525008e       SegmentId: f5b7315c7a95ceaf     Sampled: true

cat scenario-a-zip.txt
Summary:
  Success rate: 100.00%
  Total:        30049.0353 ms
  Slowest:      78.3664 ms
  Fastest:      12.3751 ms
  Average:      20.4700 ms
  Requests/sec: 0.9984

  Total data:   5.62 KiB
  Size/request: 192 B
  Size/sec:     191 B

Response time histogram:
  12.375 ms [1]  |■
  18.974 ms [18] |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  25.573 ms [6]  |■■■■■■■■■■
  32.173 ms [2]  |■■■
  38.772 ms [2]  |■■■
  45.371 ms [0]  |
  51.970 ms [0]  |
  58.569 ms [0]  |
  65.168 ms [0]  |
  71.767 ms [0]  |
  78.366 ms [1]  |■

Response time distribution:
  10.00% in 13.0683 ms
  25.00% in 13.7191 ms
  50.00% in 16.3892 ms
  75.00% in 21.5606 ms
  90.00% in 35.2758 ms
  95.00% in 35.8033 ms
  99.00% in 78.3664 ms
  99.90% in 78.3664 ms
  99.99% in 78.3664 ms


Details (average, fastest, slowest):
  DNS+dialup:   51.8177 ms, 51.8177 ms, 51.8177 ms
  DNS-lookup:   0.0747 ms, 0.0747 ms, 0.0747 ms

Status code distribution:
  [403] 30 responses

container
aws logs filter-log-events     --log-group-name "/aws/lambda/lsc-knn-container"     --filter-pattern "Init Duration"     --query 'events[*].message' --output text
REPORT RequestId: c9700c4a-2cd0-47ea-88d1-2e1cae14c53a  Duration: 86.19 ms      Billed Duration: 2243 ms        Memory Size: 512 MB     Max Memory Used: 142 MB Init Duration: 2156.57 ms
XRAY TraceId: 1-69c97f13-51864c547623d3f5173d2d02       SegmentId: 34afb2f61f3f35d0     Sampled: true

cat scenario-a-container.txt
Summary:
  Success rate: 100.00%
  Total:        30050.2572 ms
  Slowest:      76.2701 ms
  Fastest:      7.4432 ms
  Average:      18.1631 ms
  Requests/sec: 0.9983

  Total data:   5.62 KiB
  Size/request: 192 B
  Size/sec:     191 B

Response time histogram:
   7.443 ms [1]  |■■
  14.326 ms [11] |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  21.209 ms [12] |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  28.091 ms [4]  |■■■■■■■■■■
  34.974 ms [0]  |
  41.857 ms [1]  |■■
  48.739 ms [0]  |
  55.622 ms [0]  |
  62.505 ms [0]  |
  69.387 ms [0]  |
  76.270 ms [1]  |■■

Response time distribution:
  10.00% in 12.4212 ms
  25.00% in 13.2052 ms
  50.00% in 15.0161 ms
  75.00% in 16.7794 ms
  90.00% in 25.2142 ms
  95.00% in 35.4946 ms
  99.00% in 76.2701 ms
  99.90% in 76.2701 ms
  99.99% in 76.2701 ms


Details (average, fastest, slowest):
  DNS+dialup:   50.5369 ms, 50.5369 ms, 50.5369 ms
  DNS-lookup:   0.0785 ms, 0.0785 ms, 0.0785 ms

Status code distribution:
  [403] 30 responses