# Session: Stream data and IoT
Chairs: Andrea Mannocci, Niaz Chowdhuri

## 1st part
Chair: Niaz Chowdhuri   
Slides are available here: https://bit.ly/2MkZ5I0
### Key points
1. Introduction to the Internet of Things (IoT)
1. Looking at IoT Use Cases
1. Hands-on experience with an IoT device

### Task: Exploring WirelessTag used by a CityLabs SME
#### Requirements for this task
- WirelessTag Dashboard and/or App

#### What to do
1. Launch the dashboard using the credentials provided
1. Getting familiarised with the functionalities of the WirelessTag
1. Introduction to the IFTTT and how it can be used with the WirelessTag and other IoT devices.



## 2nd part
Chair: Andrea Mannocci   
Slides are available here: https://drive.google.com/open?id=1oSj0FM02GdWm3SC-XSXv-Cx7A06buMqZgWkCxlP9yik

### Key points
1. Get sense of out data
1. Lightweight prototyping of dashboards
1. Get started with Elasticsearch and Kibana

### Task 1: Twitter Streaming API
#### Requirements for this task
- Twitter API keys
- Python 3
- Libraries: Twithon, Elasticsearch-py, Elasticsearch-dsl
- Elasticsearch and Kibana running on your machine

*Note:* We have a pre-canned Virtual Machine, dedicated to our workshop.

#### What to do
1. Create index pattern
1. Launch the provided Python script and collect some tweets
1. Head to Kibana and have fun


### Task 2: Dissecting bank accounts
#### Requirements for this task
- Dataset: https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip

*Note:* the dataset has been loaded already on the virtual machine.

#### What to do
1. Load the dataset on Elasticsearch    
`curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json`
1. Slice and dice the dataset
