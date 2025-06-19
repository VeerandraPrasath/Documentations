Grafana
It is a company name 
It is a open source visualization tool that allows you to create dashboards and graphs from various data sources, including Prometheus.

SRE Principals (Site-Reliability Engineering)
![Alt text](Asserts/SRE.JPG?raw=true)

SLI (Service Level Indicator)
A metric that measures the performance of a service, such as response time or error rate. It is used to assess the reliability of a service.

SLO (Service Level Objective)
A target range value for a specific SLI. It defines the acceptable level or threshold for the SLI. For example, an SLO might state that 99% of requests should be served within 200 milliseconds.

SLA (Service Level Agreement)
SLA validate the SLOs and SLIs.It is a agreement that defines the level of service expected from a service provider.If not met,it may raise penalties or compensation for the customer. It is a formal contract between the service provider and the customer.

Proactive Monitoring and Alerting
Proactive monitoring involves continuously observing and analyzing system performance to identify potential issues and raise alerts before they impact users. It helps in preventing outages and ensuring system reliability.

Dashboards
Dashboards are visual representations of data that provide insights into system performance and health. They can display various metrics, graphs, and charts to help users understand the state of their systems at a glance.

![Alt text](Asserts/monVSobs.JPG?raw=true)]

What to observe ?
   The grafana uses multiple data sources to visualize the data such as Prometheus for metrics, Loki for logs, and Tempo for traces. It allows users to create custom dashboards and panels to visualize data from these sources.

Pull data from other Additional Sources

![Alt text](Asserts/additionalSources://gra.JPG?raw=true)]


Install Grafana 

There are lot of ways to install Grafana, such as using Docker, Cloud, or directly installing it on a server.But now we are going to see using cloud

1.Go to ```http://grafana.com/products/cloud```
2.Sign up for a free account or log in if you already have one.
3.Create a new Grafana Cloud instance.You are now inside the Grafana Cloud instance.
4.Go to More apps->Demo Data Dashboards to see the pre-built dashboards and explore the features of Grafana.

![Alt text](Asserts/grafanaCloud.JPG?raw=true)
![Alt text](Asserts/demoDashboard.JPG?raw=true)]

Additional Configuration

![Alt text](Asserts/additionalConfig.JPG?raw=true)
1.Our instance is our organization,We can set organization name,default dashboard which will reflects everyone in the organization.
2.We can also create team inside the organization
3.Provide roles to user such as viewer,editor,admin

Data Sources

Grafana will not display anything without data.Those data are pulled from the data sources such as Prometheus, Loki, and Tempo.

Aggregate Multiple Data Source

![Alt text](Asserts/aggregateDataSources.JPG?raw=true)

Grafana pull data from different data sources and can able to visualize them in a single or multiple dashboard.
Also allows to view both infrastructure and business metrics together, providing a comprehensive view of system performance and business impact.For example for infrastructure metrics, we can use Prometheus for CPU usage,Memory consumption,Disk I/O,Network latency,Error rates, and for business metrics, we can use a database like MySQL or PostgreSQL for Number of orders placed (in a food delivery app),Revenue per day/hour.

What is Prometheus ?

![Alt text](Asserts/prometheus.JPG?raw=true)

Add Data Sources

1. Naviagate Connection and Search for Prometheus
2. Give your Prometheus URL (e.g., http://localhost:9090) and click Save & Test to verify the connection.You can also upload TLS certificate for secure connections if needed.
3. Once the connection is successful, you can start creating dashboards using Prometheus data.

Creare Dashboard

1.By our own
2.Importing them by json file or we can give the dashboard ID from Grafana.com















