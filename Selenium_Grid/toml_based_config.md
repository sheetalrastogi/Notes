## Selenium Grid 4 Fully Distributed Mode Using TOML Configuration Files
-------------------------------------------------------------------------------


In Selenium Grid 4, TOML (Tom's Obvious Minimal Language) files provide a clean, maintainable way to configure Grid components instead of supplying numerous command-line arguments. Grid 4 supports modular components such as Event Bus, Session Map, Distributor, Router, and Nodes, making TOML-based configuration ideal for enterprise deployments.1.md

Example Architecture

```text
+------------+
|  Router    |  Port 4444
+------------+
      |
      v
+------------+
| Distributor|  Port 5553
+------------+
      |
      v
+------------+
| SessionMap |  Port 5556
+------------+
      |
      v
+------------+
| Event Bus  |
+------------+

      |
      +------ Chrome Node
      |
      +------ Firefox Node
      |
      +------ Edge Node
```

## Directory Structure

```text
grid-config
│
├── event-bus.toml
├── sessions.toml
├── distributor.toml
├── router.toml
│
├── chrome-node.toml
├── firefox-node.toml
└── edge-node.toml

```
## 1. Event Bus Configuration
event-bus.toml

```text
[events]
publish = "tcp://0.0.0.0:4442"
subscribe = "tcp://0.0.0.0:4443"
bind = true
```

## Start Event Bus:
```java
java -jar selenium-server-4.x.x.jar event-bus \
  --config event-bus.toml
```

## 2. Session Map Configuration
sessions.toml

```text
[sessions]
host = "0.0.0.0"
port = 5556
```

## Start Session Map:

```java
java -jar selenium-server-4.x.x.jar sessions \
  --config sessions.toml
```

## 3. Distributor Configuration
distributor.toml

```text
[server]
port = 5553

[distributor]
sessionqueue = "http://localhost:5559"

[events]
publish = "tcp://localhost:4442"
subscribe = "tcp://localhost:4443"

[sessions]
server = "http://localhost:5556"
```

## Start Distributor:

```text
java -jar selenium-server-4.x.x.jar distributor \
  --config distributor.toml
```

## 4. Router Configuration
router.toml

```text
[server]
port = 4444

[router]
username = ""
password = ""

[sessions]
server = "http://localhost:5556"

[distributor]
server = "http://localhost:5553"
```

## Start Router:

```text
java -jar selenium-server-4.x.x.jar router \
  --config router.toml
```

## 5. Chrome Node Configuration
chrome-node.toml

```text
[server]
port = 5555

[node]
detect-drivers = true
max-sessions = 5

[events]
publish = "tcp://localhost:4442"
subscribe = "tcp://localhost:4443"

[[node.driver-configuration]]
display-name = "Chrome"
max-sessions = 5
stereotype = '{"browserName":"chrome"}'
```

## Start Chrome Node:

```text
java -jar selenium-server-4.x.x.jar node \
  --config chrome-node.toml
```

## 6. Firefox Node Configuration
firefox-node.toml

```text
[server]
port = 5557

[node]
detect-drivers = true
max-sessions = 5

[events]
publish = "tcp://localhost:4442"
subscribe = "tcp://localhost:4443"

[[node.driver-configuration]]
display-name = "Firefox"
max-sessions = 5
stereotype = '{"browserName":"firefox"}'
```

## Start Firefox Node:

```text
java -jar selenium-server-4.x.x.jar node \
  --config firefox-node.toml
```

## 7. Edge Node Configuration
edge-node.toml

```text
[server]
port = 5558

[node]
detect-drivers = true
max-sessions = 5

[events]
publish = "tcp://localhost:4442"
subscribe = "tcp://localhost:4443"

[[node.driver-configuration]]
display-name = "Edge"
max-sessions = 5
stereotype = '{"browserName":"MicrosoftEdge"}'
```

## Start Edge Node:
```text
java -jar selenium-server-4.x.x.jar node \
  --config edge-node.toml
```
## Dedicated Chrome Node with Headless Browser

```text
[node]
detect-drivers = true
max-sessions = 10

[[node.driver-configuration]]
display-name = "Chrome Headless"
max-sessions = 10

stereotype = '''
{
  "browserName": "chrome",
  "goog:chromeOptions": {
      "args": [
          "--headless=new",
          "--disable-gpu",
          "--no-sandbox"
      ]
  }
}
```

## High-Capacity Node

For CI/CD execution:

```text
[node]
max-sessions = 20
override-max-sessions = true
```

## Start Components in Order
---------------------------------

# Event Bus
java -jar selenium-server.jar event-bus --config event-bus.toml

# Session Map
java -jar selenium-server.jar sessions --config sessions.toml

# Distributor
java -jar selenium-server.jar distributor --config distributor.toml

# Router
java -jar selenium-server.jar router --config router.toml

# Chrome Node
java -jar selenium-server.jar node --config chrome-node.toml

# Firefox Node
java -jar selenium-server.jar node --config firefox-node.toml

# Edge Node
java -jar selenium-server.jar node --config edge-node.toml

## Verify Grid

Open:
```text
http://localhost:4444/ui
```

## Check Grid status:
```text
curl http://localhost:4444/status
```

Expected:
```java
{
  "value": {
    "ready": true,
    "message": "Selenium Grid ready."
  }
}

```

Java Test Example

```java

WebDriver driver = new RemoteWebDriver(
        new URL("http://localhost:4444"),
        new ChromeOptions());

driver.get("https://www.google.com");

```

## Enterprise Best Practices

- Maintain separate TOML files for each Grid component.
- Run Event Bus, Router, Distributor, and Session Map on dedicated infrastructure.
- Use Docker/Kubernetes for node scaling.
- Configure max-sessions based on CPU and memory.
- Create dedicated TOML configurations for Chrome, Firefox, Edge, Mobile Emulation, and Headless nodes.
- Store all TOML files in source control for environment consistency.

This TOML-based approach makes Selenium Grid 4 Fully Distributed deployments much easier to manage, scale, and maintain compared to long command-line configurations.
