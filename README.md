# Take home challenge
## Part 1: Architectural Challenge
### High Level Product workflow
```
Legend:
(system runs on AWS - EKS)
Dashboard: React.js website that customers use to interact with our API
API: Public API service that that is consumed by our dashboard, CLI and custom API clients
Storage: Service wrapper (a.k.a. core-api) for our DB, all internal services can only communicate with the DB via this service
DB: RDS postgres
MQ: RabbitMQ
Worker: Service that consumes scan requests from rabbitmq and creates scan jobs
ScanJob: Kubernetes job - runs a docker container, mounts customer repository source code, return result as JSON string via stdout
DataProcessing: Processes the scanJob result data
Notification: Notification service
GitProvider: E.g. github.com
```
1. Flow 1: User triggers scan
    - Customer logs in to the Dashboard
    - Customer selects a GitHub repository URL and clicks scan which will scan the repository source code
    - (1) API receives Dashboard request and sends request to the Storage service to create a Scan entity in the DB
    - (2) pushes the id of Scan entity into rabbitmq
    - (3) Worker consume the message from rabbitmq
    - (4) Worker calls the Storage service to request the Scan entity and its related data. Based on the source code content, a scan might require one or many jobs to run, a job can take from 5 seconds to 30 minutes to complete
    - (5) Worker schedules a job by calling the kubernetes API with job specification, including
        - Docker image
        - source code
    - (6) After all the jobs are completed, the Data processing service will summarize the data and calculate the the final Scan result from all jobs results
    - (7) The Scan result is saved into the DB via the Storage service
    - (8) The Notification services are triggered
    - (9) Notification service sends report to Slack and github 
2. Flow 2: User gets scan result
    - Customer login into Dashboard
    - Customer enters a GitHub repository URL and clicks on view result which will return the scan result for the repository
    - Dashboard sends a request to the API, and queries customer scan result via the Storage service and responds to the Dashboard

### Questions
1. Question 1

    How would you improve the current design to achieve better:
    ```    
    High availability
    Resilience
    Performance
    Cost efficiency
    ```
    You can list items based on priority, ranked from high to low.

    Answer:

        - Resilience
        - High availability
        - Performance
        - Cost efficiency

2. Question 2

    The number of scan requests can increase/decrease randomly in a day and on most weekends the system receives almost no requests at all.

    What strategy would you suggest to save cost while still maintaining the best possible performance and scan completion times?

    Answer:
    ```
    We can run batch jobs on Amazon EKS as a Kubernetes job on Fargate using StepFunctions.

    Amazon EKS provides flexibility to develop many container use cases like long running jobs, web application, micro-services architecture, on-demand job execution, batch processing, machine learning applications with seamless integration in conjunction with other AWS services.

    AWS Fargate provides serverless compute engine options for both Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes Service (EKS). Fargate makes it easy for you to focus on building your applications and not have to worry about managing the compute resources. Fargate removes the need to provision and manage servers, lets you specify and pay for resources per application, and improves security through application isolation by design

## Part 2: Technical Challenge
### Schedule batch jobs

Ignore this part because I am not clearly about requirements.