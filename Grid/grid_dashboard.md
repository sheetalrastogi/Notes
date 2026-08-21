Copilot said:

For an enterprise Selenium Grid monitoring solution, you can:

1. Query Selenium Grid /status API to discover all registered nodes.
2. Query Selenium GraphQL API for queue/session metrics.
3. Query a lightweight agent running on each node (Spring Boot/Javalin REST endpoint) to collect:
  - CPU %
  - Memory %
  - Disk %
  - Network Usage
  - Container Memory %
4. Build a consolidated dashboard object.


# Architecture

```text
                  Selenium Hub
                        |
         ------------------------------
         |            |               |
         |            |               |
      Node-1       Node-2         Node-3
         |            |               |
         |            |               |
   Metrics API   Metrics API    Metrics API
         |            |               |
         --------------------------------
                        |
               Grid Monitoring Utility

```

# Maven Dependency

```xml
<dependencies>

    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.5.1</version>
    </dependency>

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.18.0</version>
    </dependency>

    <dependency>
        <groupId>com.github.oshi</groupId>
        <artifactId>oshi-core</artifactId>
        <version>6.6.5</version>
    </dependency>

</dependencies>
```

# POJO for Node Metrics

```java
public class GridNodeMetrics {

    private String nodeId;
    private String nodeUri;

    private double cpuPercent;
    private double memoryPercent;
    private double diskPercent;
    private double networkUsageMB;

    private double containerMemoryPercent;

    private int runningSessions;
    private int availableSlots;
    private double availabilityPercent;

    // getters setters
}
```

# Step 1: Discover Selenium Grid Nodes

```java
import io.restassured.RestAssured;
import io.restassured.path.json.JsonPath;

import java.util.ArrayList;
import java.util.List;

public class GridNodeDiscovery {

	private static final String GRID_STATUS = "http://localhost:4444/status";

	public static List<String> getRegisteredNodes() {
		List<String> nodes = new ArrayList<>();

		JsonPath json = RestAssured.get(GRID_STATUS).jsonPath();
		List<String> uris = json.getList("value.nodes.uri");
		nodes.addAll(uris);
		return nodes;
	}
}
```

# Step 2: Capture Grid Metrics

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;

public class GridMetrics {

	public static int getSessionQueueSize() {

		String query = "{\"query\":\"{ sessionsInfo { sessionQueueSize }}\"}";

		Response response = RestAssured.given().contentType("application/json").body(query)
				.post("http://localhost:4444/graphql");

		return response.jsonPath().getInt("data.sessionsInfo.sessionQueueSize");
	}
}
```


# Grid Utilization %

```java
	public static double getGridUtilization() {

		JsonPath status = RestAssured.get("http://localhost:4444/status").jsonPath();

		List<Object> nodes = status.getList("value.nodes");

		int totalSlots = 0;
		int usedSlots = 0;

		for (int i = 0; i < nodes.size(); i++) {

			List<Object> slots = status.getList("value.nodes[" + i + "].slots");

			totalSlots += slots.size();

			for (int j = 0; j < slots.size(); j++) {

				Object session = status.get("value.nodes[" + i + "].slots[" + j + "].session");

				if (session != null) {
					usedSlots++;
				}
			}
		}

		return totalSlots == 0 ? 0 : ((double) usedSlots / totalSlots) * 100;
	}
```

# Step 3: Node Metrics Agent

Deploy on each Grid Node. to capture CPU%, Memory % etc.

**CPU %**

```java
	public static double getCpuPercent() {
		SystemInfo si = new SystemInfo();
		CentralProcessor processor = si.getHardware().getProcessor();

		long[] prev = processor.getSystemCpuLoadTicks();
		try {
			Thread.sleep(1000);
		} catch (Exception e) {
		}
		return processor.getSystemCpuLoadBetweenTicks(prev) * 100;
	}
```

**Memory %**

```java
	public static double getMemoryPercent() {

		GlobalMemory memory = new SystemInfo().getHardware().getMemory();

		long used = memory.getTotal() - memory.getAvailable();

		return used * 100.0 / memory.getTotal();
	}
```


**Disk %**

```java
	public static double getDiskPercent() {
		File root = new File("/");
		long total = root.getTotalSpace();
		long free = root.getUsableSpace();
		long used = total - free;
		return used * 100.0 / total;
	}

```

**Network Usage**

```java
	public static double getNetworkUsageMB() {
		SystemInfo si = new SystemInfo();
		List<NetworkIF> interfaces = si.getHardware().getNetworkIFs();
		long bytes = 0;

		for (NetworkIF net : interfaces) {
			net.updateAttributes();
			bytes += net.getBytesRecv();
			bytes += net.getBytesSent();
		}
		return bytes / (1024.0 * 1024);
	}
```

**Container Memory %**

```java
	public static double getContainerMemoryPercent() throws Exception {

		Process process = Runtime.getRuntime()
				.exec("docker stats --no-stream --format {{.MemPerc}} selenium-node-chrome");

		BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));

		String value = br.readLine();

		return Double.parseDouble(value.replace("%", ""));
	}

```

**Node Availability %**

```java
	public static double getAvailability(int onlineNodes, int registeredNodes) {

		return registeredNodes == 0 ? 0 : onlineNodes * 100.0 / registeredNodes;
	}
```


**Running Sessions & Available Slots**

```java
	public static void collectNodeSlots() {
		JsonPath status = RestAssured.get("http://localhost:4444/status").jsonPath();
		List<Object> nodes = status.getList("value.nodes");

		for (int i = 0; i < nodes.size(); i++) {
			List<Object> slots = status.getList("value.nodes[" + i + "].slots");
			int running = 0;
			for (int j = 0; j < slots.size(); j++) {
  				Object session = status.get("value.nodes[" + i + "].slots[" + j + "].session");
				if (session != null) {
					running++;
				}
			}

			int available = slots.size() - running;
			System.out.println("Running Sessions : " + running);
			System.out.println("Available Slots : " + available);
		}
	}

```

**Grid Up Time**

```java
import java.lang.management.ManagementFactory;
	public static String getGridUptime() {

		long uptime = ManagementFactory.getRuntimeMXBean().getUptime();

		long seconds = uptime / 1000;

		long hours = seconds / 3600;

		long minutes = (seconds % 3600) / 60;

		return hours + "h " + minutes + "m";
	}
```


# Example Dashboard Output

```text
=========================================================
             SELENIUM GRID HEALTH DASHBOARD
=========================================================

Grid Up Time            : 3d 05h 22m
Grid Utilization        : 67.5 %
Session Queue Size      : 4

Registered Nodes        : 4
Online Nodes            : 4
Node Availability       : 100 %

---------------------------------------------------------

Node : ChromeNode01

CPU %                   : 41.2
Memory %                : 58.4
Disk %                  : 62.7
Network Usage (MB)      : 1054
Container Memory %      : 39.6

Running Sessions        : 4
Available Slots         : 1

---------------------------------------------------------

Node : ChromeNode02

CPU %                   : 53.8
Memory %                : 67.0
Disk %                  : 61.1
Network Usage (MB)      : 975
Container Memory %      : 43.2

Running Sessions        : 3
Available Slots         : 2

=========================================================
GRID HEALTH : HEALTHY
=========================================================
```



