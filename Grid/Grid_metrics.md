# Selenium Grid capacity:
-------------------------

In Selenium Grid 4, you can monitor grid capacity and identify **active nodes**, **total slots**, **used slots**, and **available capacity** by calling the Grid Status API as follow:

```text
http://<hub-host>:4444/status
```

In order to monitor it via Selenium Java code:

# 1. Maven Dependencies

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.5.0</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.19.0</version>
</dependency>
```


# 2. POJO to capture Grid statistics

```java
public class GridCapacity {

    private int registeredNodes;
    private int totalSlots;
    private int usedSlots;
    private int availableSlots;
    private double utilization;

    private int chromeSlots;
    private int firefoxSlots;
    private int edgeSlots;

    // getters & setters
}
```

# 3. Utility Class

```java
	public static GridCapacity getGridCapacity(String gridUrl) {

		GridCapacity capacity = new GridCapacity();

		try {

			Response response = RestAssured.get(gridUrl + "/status");

			ObjectMapper mapper = new ObjectMapper();
			JsonNode root = mapper.readTree(response.asString());
			JsonNode nodes = root.path("value").path("nodes");

			int registeredNodes = nodes.size();
			int totalSlots = 0;
			int usedSlots = 0;

			Map<String, Integer> browserSlots = new HashMap<>();
			for (JsonNode node : nodes) {
				JsonNode slots = node.path("slots");
				totalSlots += slots.size();

				for (JsonNode slot : slots) {
					String browserName = slot.path("stereotype").path("browserName").asText("Unknown");
					browserSlots.put(browserName, browserSlots.getOrDefault(browserName, 0) + 1);

					if (!slot.path("session").isMissingNode() && !slot.path("session").isNull()) {
						usedSlots++;
					}
				}
			}

			int availableSlots = totalSlots - usedSlots;
			double utilization = totalSlots == 0 ? 0 : ((double) usedSlots / totalSlots) * 100;

			capacity.setRegisteredNodes(registeredNodes);
			capacity.setTotalSlots(totalSlots);
			capacity.setUsedSlots(usedSlots);
			capacity.setAvailableSlots(availableSlots);
			capacity.setUtilization(utilization);

			capacity.setChromeSlots(browserSlots.getOrDefault("chrome", 0));
			capacity.setFirefoxSlots(browserSlots.getOrDefault("firefox", 0));
			capacity.setEdgeSlots(browserSlots.getOrDefault("MicrosoftEdge", 0) + browserSlots.getOrDefault("edge", 0));

		} catch (Exception e) {
			throw new RuntimeException(e);
		}

		return capacity;
	}

```

# Usage:

```java
@BeforeSuite
public void gridHealthCheck() {

    GridCapacity capacity = GridMonitor.getGridCapacity();

    ExtentReportManager.logInfo("Grid Utilization : " + capacity.getUtilization() + "%");

    Assert.assertTrue(capacity.getAvailableSlots() > 0, "No Grid Capacity Available");

	// other checks:
	ExtentReportManager.logInfo("Registered Nodes : " + grid.getRegisteredNodes());
	ExtentReportManager.logInfo("Total Slots : " + grid.getTotalSlots());
	ExtentReportManager.logInfo("Used Slots : " + grid.getUsedSlots());
	ExtentReportManager.logInfo("Available Slots : " + grid.getAvailableSlots());
	ExtentReportManager.logInfo("Utilization : " + grid.getUtilization() + "%");
	}

}
```

## # Additional Selenium Grid Metrics Worth Tracking

- Grid Up Time
- Registered Nodes
- Node Health
- Total Slots
- Used Slots
- Available Slots
- Browser-wise Capacity
- Session Queue Size
- Pending Requests
- Average Session Duration
- Failed Session Count
- Grid Utilization %
- Node CPU %
- Node Memory %
