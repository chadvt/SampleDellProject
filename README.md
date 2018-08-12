# SampleDellProject
Example of large inventory management solution for retail

>Scenario Summary: I am a warehousing company. On the first and third Friday of every month, our sales spike due to members’ paydays. 
> 
> During these times, people checking our produce inventory within the stores (and online) goes up significantly so we want to make sure that we are able to accurately display inventory so we are not selling produce that we do not have enough of. 
Anecdotal evidence from cashiers and customers claim inventory lookups are slow during busy peak times. 
>
>The current strategy is to add more resources to the database cluster but I want to assure the business that we are ready to scale at a reasonable cost.

Assumptions:
•	All warehouse stores have network connectivity to database resources, cloud, and local data centers
•	The business will want to provide fault tolerance leveraging on-premise servers and cloud resources as backup
•	 Provide 99.99% uptime with <5 second response time for a reasonable cost

 # Tools to Use:

•	  [Vagrant](http://vagrant.io/) - Using standardized OS images to test continuous build's will free up resources (developers and testers) - no need for dedicated environments. Vagrant image/build will include local software load balancer, database, inventory app, and API

•	 [Docker](https://docs.docker.com/toolbox/toolbox_install_windows/)/[Kubernetes](https://kubernetes.io/) - Containerization will reduce vendor lock-in while allowing for on-prem hardware scaling and cloud resource. Leveraging application based caching solutions (redis for example) while running in multiple regions and local data centers, will ensure rapid response as well as the 99.99% uptime

•	 Secure Cloud Services ([AWS](https://aws.amazon.com/)/[Azure](https://azure.microsoft.com/en-us/)/[GCP](https://cloud.google.com/)) - Containerized approach leveraging a cloud strategy will allow for relevant region specific resources expand and contract as needed while keeping the costs at a minimum

•	 Cloud ready datastore - Leveraging key value index stores (example - [Cassandra](http://cassandra.apache.org/), [Redis](https://redis.io/), etc) will be leveraged for rapid updates with quick response times. 

•	 BI functionality to be provided with [ODS Snowflake DB](https://www.snowflake.com/product/)

•	 [Consul](http://www.consul.io) - Manage both on-prem and cloud infrastructure - pushing continuous builds with a minimum of impact

Proposed Workflow: The workflow I envision is a webapp running onsite at the warehouses as well as customer mobile devices. The workflow would be - Mobile device webapp-->API-->Datastore. Running containerized API apps, you will be able to run in the cloud or on-prem without performance penalties

# Deployment 
Cloud Approach: AWS S3 hosting static content and JS, with CloudFront CDN - Key to success is a highly reliable db with high throughput, all while conserving resources

# Deployment Diagram
Please reference the [Proposed Workflow Diagram](https://github.com/chadvt/SampleDellProject/blob/master/WareCostCo.jpg) for a visual reference.
![alt text](https://github.com/chadvt/SampleDellProject/blob/master/WareCostCo.jpg)
