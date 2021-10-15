# S3HttpClientApi
## Configuration
* AWS EC2 Instance	: t3a.micro
* Configuration: 2 vCPU, 1 GB RAM, Network Bandwidth - Up to 5 Gigabit
* Region: us-west-1
* Application : .Net Core 5
* File Size : 345 Byte
* S3 Bucket and EC2 instance are in same region
* Created VPC between S3 Bucket and EC2 instace
* 
## Isssue : S3 Response time is higher.
We created an application that will retrieve S3 data in parallel from the S3 bucket. While doing performance testing, we found that for a single file fetch the following is observed.
1 When we call the API in a loop with different Ids (multiple calls within 1-2 seconds) then the response time is around 20-30ms.
    * We observed that when the response time is 20-30ms, HttpSocket is still open (from Http Logs).
    * DNS lookup and socket handshaking steps were skipped when these requests were observed .
     
2 When we do the same test as #1 but with a delay of 3-10 seconds between each requests, the following results were seen
    1. 50ms-100ms mostly.
    2. 100+ms sometimes
    
3 When we call the API with an interval of more than >10 seconds, S3 will now take 100ms to 150ms to send response back.

We also monitored S3 logs and found that there is some difference between S3 logs and http logs in terms of processing time. our assumption is that S3 logs does not consider any network latency for results. Please check image/logs for more information.

## We tried the following solutions but could not see any improvements

    1. S3 Client with standard configuration.
    2. Implement DotNet HttpClient Factory and set it in S3 configuration.
    3. Implement Custom HttpClient Factory and set it in S3 configuration.

* Please check code for more information. Search for Case-1, Case-2 and Case-3 for above 3 solution.

## Logs
* We added HttpEventListener to check timing of each and every activity of http call.
* Please find complete logs in log folder.

    1. Result in 53ms - Without resuse of socket connection/HttpHandShake.
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
    1. S3 SDK reuses socket connection for a shorter interval and closes it. S3 requests that caches DNS lookup and bypasses Http Handshaking and resuses socket connection, performance is the best (20ms). 
    2. S3 SDK where it has to do handshaking again, the performance is not the best (ranges from 50ms to 150 ms)
    4. The most concering thing is that S3 response times are very random and erratic. It is not that we are just seeing some random spikes. Performance in general is mostly random and erratic  (ranges from 50ms to 150ms) mostly > 80ms. 



