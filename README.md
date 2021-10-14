# S3HttpClientApi

## Isssue : S3 Response time is higher.
We created an application that will retrieve S3 data in parallel from the S3 bucket. While doing performance testing, we found that for a single file fetch.
1 When we call the  API in a loop with different Ids (without delay) then the response time is around 20-30ms.
    * We observed that when the response time is 20-30ms, HttpSocket is still open.
    * DNS lookup and socket handshaking steps were skipped when these requests were observed .
     
2 When we call the same API randomly within a delay of 10 seconds, results were be one of the following
    1. 30ms-80ms mostly.
    1. 100+ms sometimes
    
3 When we call the API with an interval of more then 10-20 seconds, S3 will take 100ms to 200ms to send response back.

We also checked with S3 logs and found that it sends response in 20ms to 80ms but in actual scenarios it takes more time. Please check image/logs for more information.

## We tried the following solutions but could not see any improvements

    1. S3 Client with standard configuration.
    2. Implement DotNet HttpClient Factory and set it in S3 configuration.
    3. Implement Custom HttpClient Factory and set it in S3 configuration.

* Please check code for more information. Search for Case-1, Case-2 and Case-3 for above 3 solution.

## Logs
* We added HttpEventListener to check timing of each and every activity of http call.
* Please find complete logs in log folder.

    1. Result in 53ms - With socket connection and HttpHandShake.
    ![Alt Log](Log/HttpLogs_53ms.jpg)

    2. Result in 20ms - Reuse socket connection
    ![Alt Log](Log/HttpLogs_20ms.jpg)

    3. Result in 108ms - With socket connection and HttpHandShake, S3 still takes time to retrieve data (inconsistent performance).
    ![Alt Log](Log/HttpLogs_108ms.jpg)

Http Logs (Time in ms)	S3 Logs (Time in ms)
   113	                  72
   102	                  88
   53	                     41
   123	                  67
   49	                     37
   23	                     19


## Conclusion
* From image 1,2 and 3, it is clear that
    1. S3 SDK reuses socket connection for a shorter interval and closes it. (53ms)
    2. S3 requests that caches DNS lookup and Http Handshaking bypass, performance is better. 
    3. If we fetch data in a loop without any delay then it reuses socket connection. (20ms)    
    4. S3 randomly take time to respond and hence fetch time is more then 100ms. 

## Configuration
* AWS EC2 Instance	: t3a.micro
* Configuration: 2 vCPU, 1 GB RAM, Network Bandwidth - Up to 5 Gigabit
* Region: us-west-1
* Application : .Net Core 5
* File Size : 345 Byte
* S3 Bucket and EC2 instance are in same region
* Created VPC between S3 Bucket and EC2 instnace

