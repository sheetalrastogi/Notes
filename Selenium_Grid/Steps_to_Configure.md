## Selenium 4 Grid - **Standalone mode**
--------------------------------------------

# Prerequisites
- Java 11 or later
- Selenium Server JAR
- Chrome/Firefox/Edge installed on node machines

# Download Selenium Server:

```text

	https://github.com/SeleniumHQ/selenium/releases
	
```

**Step 1**: Start Grid:

```java

	java -jar selenium-server.jar standalone
	
```
**Step 2**: Check grid is up & running:

```java

	http://localhost:4444

```

**Step 3**: Connect Selenium tests using following code:

```java

	WebDriver driver = new RemoteWebDriver(new URL("http://localhost:4444"), new ChromeOptions());

```

## Selenium 4 Grid - **Hub-Node mode**
--------------------------------------------

**Step 1**: Start the Hub:

On the Hub machine:
```java

	java -jar selenium-server-4.x.x.jar hub

```

**Step 2**: Check Hub is up & running:

```java

	http://localhost:4444

```

**Step 2**: Start a Chrome Node

On the node machine:

```java

	java -jar selenium-server-4.x.x.jar node --hub http://<hub-host-ipAddress>:4444
	
```

Similarly register nodes for Edge & Firefox drivers (seperate machine):

```java

	java -jar selenium-server-4.x.x.jar node --hub http://192.168.1.100:4444

```

**Step 3**: Verify Nodes

Open:

	http://localhost:4444/ui


You should see:

Available Nodes, along with session capacity information

Chrome
Firefox
Edge


Alternatively **Node Configuration Using TOML**
---------------------------------------------------

# 1. Create **node.toml** with following contents:

```text
[node]
max-sessions = 5

[[node.driver-configuration]]
display-name = "Chrome"
stereotype = "{\"browserName\": \"chrome\"}"
max-sessions = 5

```

# 2. Start node with toml file:

java -jar selenium-server-4.x.x.jar node --config node.toml



## Selenium 4 Grid - **Fully Distributed mode**
--------------------------------------------------

Fully Distributed Mode is the most scalable Selenium Grid 4 deployment where each service runs as an independent process that communicates through an Event Bus. Ideal for large-scale enterprise, cloud, Docker, and Kubernetes deployment.

Architecture:

```text

                    WebDriver Tests
                           |
                           v
                     +-----------+
                     |  Router   |
                     +-----------+
                           |
           +---------------+---------------+
           |                               |
           v                               v

     +-----------+                 +-----------+
     | Session   |                 |Distributor|
     |   Map     |                 +-----------+
     +-----------+                       |
                                         |
                    +--------------------+------------------+
                    |                                       |
                    v                                       v

              +-----------+                         +-----------+
              | Chrome    |                         | Firefox   |
              | Node      |                         | Node      |
              +-----------+                         +-----------+

                    ^
                    |
             +-------------+
             |  Event Bus  |
             +-------------+

```

# Step 1: Download Selenium Server

# Step 2: Start Event Bus with default ports (Publish Port : 4442, Subscribe Port : 4443)

```text

	java -jar selenium-server-4.x.x.jar event-bus

```

# Step 3: Start Session Map at default port 5556

```text

	java -jar selenium-server-4.x.x.jar sessions

```

# Step 4: Start Distributor at default port 5553

```text

java -jar selenium-server-4.x.x.jar distributor \
--sessions http://localhost:5556 \
--publish-events tcp://localhost:4442 \
--subscribe-events tcp://localhost:4443

```

# Step 5: Start Router at default port 4444 (Router is the public entry point for all Grid requests)

```text

java -jar selenium-server-4.x.x.jar router \
--sessions http://localhost:5556 \
--distributor http://localhost:5553

```

# Verify Test grid is accessible:

``` text
	http://localhost:4444/ui
```

# Step 6: Start Chrome Node 

```text

java -jar selenium-server-4.x.x.jar node \
--publish-events tcp://localhost:4442 \
--subscribe-events tcp://localhost:4443

```

# Step 7: Start Firefox Node

```text

java -jar selenium-server-4.x.x.jar node \
--publish-events tcp://localhost:4442 \
--subscribe-events tcp://localhost:4443

```

# Step 8: Start Edge Node

```text

java -jar selenium-server-4.x.x.jar node \
--publish-events tcp://localhost:4442 \
--subscribe-events tcp://localhost:4443

```


# Verify Grid is registered with all above nodes

http://localhost:4444/ui

Output, you should see:

```text
Distributor
Session Map
Router
Event Bus

Chrome Node
Firefox Node
Edge Node
```


# Test Grid status:
```text
curl http://localhost:4444/status
```

Example output:

```java
{
  "value": {
    "ready": true,
    "message": "Selenium Grid ready."
  }
}
```


## Summary:
-----------------

```text

# Event Bus
java -jar selenium-server.jar event-bus

# Session Map
java -jar selenium-server.jar sessions

# Distributor
java -jar selenium-server.jar distributor \
--sessions http://localhost:5556 \
--publish-events tcp://localhost:4442 \
--subscribe-events tcp://localhost:4443

# Router
java -jar selenium-server.jar router \
--sessions http://localhost:5556 \
--distributor http://localhost:5553

# Nodes
java -jar selenium-server.jar node \
--publish-events tcp://localhost:4442 \
--subscribe-events tcp://localhost:4443


```








