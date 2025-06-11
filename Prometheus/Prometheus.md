
# 📈 Prometheus Monitoring Guide

## 🧭 Purpose of Prometheus

Whenever we deploy or release our application in production,it may go wrong because of sortage of memory,denial of services in network.So,It is way much better if we know those things before going to happen .For that we need to monitor .Here Prometheus is helping us to monitor our application.

---

## 🛠️ Understand How Prometheus Works

* free open source monitoring and alerting toolkit
* Server component which we run the prometheus instance on each environment.
* Its a cross-platform software and it runs same way on both linus and windows.

---

## 📊 Metrics

Metrics are the piece of numerical data which give the current state about performance and usage of the of system(eg CPU usage,Memory consumption)

---

## 🧠 Functions run inside the prometheus server:

![Alt text](Asserts/architecture.JPG?raw=true)

### Scheduler component 
   Fetches metrics from the system we want to monitor.Uses a pull model to get all the metrics from the sytem for every time period based on our configuration.For that our system which is the window need to expose all the metrics .It can be the memory usage,network speed,or even it can be a website expose metrics through http response and can be custom written metrics.

### Storage component
Stores all the metrics in a time series database.So that If we want to see some metric within certain period we can able to do that .Also we can aggregate over time, and we can see those trends and spikes(How much word requried to solve or work around software issue)

### Note:
Prometheus has its own query language, because time series data doesn't really fit well with standard SQL

### API component 
There is an http api running in the server which is used to run the queries and give the appropriate response.

### UI component 
The UI component is used to do some admin operation and can also run queries with the UI and see the current values of metrics and have them visualized in a graphical viw. But it will not give full-fledged operations to perform.

### Alerting component
Here we will register some alerting rules .When those rules are triggered ,it will send the alert to the alert manager which is a separate component of prometheus.The alert manager will send the message through email ,pages and evening by creating tickets.

---

## 💎 What makes Promethus so awasome ?

 When I need to monitor any system,I need to run exporter which expose all the metrics in a http endpoints.Those exporters provide metrics in a standard Prometheus formats.

### Exporter:
   For a demo,We use a tool called Node exporter which is a part of the Prometheus ecosystem which expose all the metrics in a http endpoints.
### Note :
   Metrics which begin with the word node is apply across the whole server which means it is applied for the entire machine or system,not for a single application.
   Eg: node_cpu_seconds_total{mode="idle"} 1.234567890
![Alt text](Asserts/exporterData.JPG?raw=true)
In the above image,the node_filefd is the total number of file descriptors allocated by all the processess running in the server.

### Client Library :
   A linux server is running a application.That application also need to expose some metrics.For that we need to have some code to perform.Most of the languages like C#,Java ,python have Prometheus client libraries which we can use to expose the metrics in a http endpoint.
  
![Alt text](Asserts/clientLibrary.JPG?raw=true)

### Note :  
  Also understand ,Node exporter is not a client library,It exposes the metrics of the entire server,not for a single application.But client library is used to expose the metrics of a single application.

## Level of metrics:

![Alt text](Asserts/levelsOfMetrics.JPG?raw=true)

* Runtime metrics
     These are the metrics which contains detail about the runtime of the application,like how many threads are running,how many garbage collections are happening,how much memory is being used by the application.
* Application metrics
        These are the metrics which are exposed by the client library and it is used to monitor the application itself.
* Hardware metrics
      These are the metrics which are exposed by the node exporter and it is used to monitor the hardware of the server like CPU usage,Memory consumption,Disk usage etc.

### Bundle Of Exporter And Client Libraries

![Alt text](Asserts/bundleOfExporterAndClientLibraries.JPG?raw=true)

 The combination of the lightweight server and the wide range of exporters and client libraries make the Prometheus so awesome.

 ## Most attractive things about Prometheus


 we have the prometheus server and our application in production.We are facing some performance issue.Now we are running the applicaton locally with some added metrics and another prometheus server locally.So that we can find and fix the issue locally.Now we make the application live also with the added metrics.After that when we try to monitor those metrices through our production prometheus ,we will get the same exact results.

### Benefits :
   We will be more confident about the results because we already tested locally
   We can easily find and fix the issue because we can test it in locally with custom metrics.
   We can easily add new metrics to the application and monitor it in production.

   ![Alt text](Asserts/attractiveThing.JPG?raw=true)
