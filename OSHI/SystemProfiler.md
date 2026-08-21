## OSHI (Operating System and Hardware Information)

**OSHI (Operating System and Hardware Information)** is a lightweight Java library that can capture CPU, Memory, Disk, Network, Process, OS, and Hardware metrics without native code dependencies.


# Maven Dependency

```xml
<dependency>
    <groupId>com.github.oshi</groupId>
    <artifactId>oshi-core</artifactId>
    <version>6.6.5</version>
</dependency>
```

# 1. Basic System Information

```java
import oshi.SystemInfo;
import oshi.software.os.OperatingSystem;
import oshi.hardware.HardwareAbstractionLayer;

public class SystemInfoExample {
	public static void main(String[] args) {
		SystemInfo si = new SystemInfo();

		OperatingSystem os = si.getOperatingSystem();
		HardwareAbstractionLayer hal = si.getHardware();

		System.out.println("OS: " + os);
		System.out.println("Manufacturer: " + hal.getComputerSystem().getManufacturer());
		System.out.println("Model: " + hal.getComputerSystem().getModel());
		System.out.println("Serial Number: " + hal.getComputerSystem().getSerialNumber());
	}
}
```


# 2. Physical Memory Information

```java
import oshi.SystemInfo;
import oshi.hardware.GlobalMemory;

public class MemoryInfoExample {

	public static void main(String[] args) {
		SystemInfo si = new SystemInfo();
		GlobalMemory memory = si.getHardware().getMemory();

		long total = memory.getTotal();
		long available = memory.getAvailable();
		long used = total - available;

		System.out.println("Total Memory (GB): " + total / (1024.0 * 1024 * 1024));
		System.out.println("Used Memory (GB): " + used / (1024.0 * 1024 * 1024));
		System.out.println("Available Memory (GB): " + available / (1024.0 * 1024 * 1024));
		double usedPercent = (used * 100.0) / total;
		System.out.printf("Memory Usage %% : %.2f%n", usedPercent);
	}
}

```

# 3. CPU Utilization Percentage

OSHI calculates CPU usage based on CPU ticks.

```java
import oshi.SystemInfo;
import oshi.hardware.CentralProcessor;

public class CpuUsageExample {

	public static void main(String[] args) throws Exception {
		SystemInfo si = new SystemInfo();
		CentralProcessor cpu = si.getHardware().getProcessor();

		long[] prevTicks = cpu.getSystemCpuLoadTicks();
		Thread.sleep(1000);
		double cpuLoad = cpu.getSystemCpuLoadBetweenTicks(prevTicks) * 100;
		System.out.printf("CPU Usage: %.2f%%%n", cpuLoad);
	}
}
```

# 4. CPU Core Details

```java
import oshi.SystemInfo;
import oshi.hardware.CentralProcessor;

public class CpuDetails {

	public static void main(String[] args) {
		CentralProcessor cpu = new SystemInfo().getHardware().getProcessor();

		System.out.println("Processor : " + cpu.getProcessorIdentifier().getName());
		System.out.println("Physical Cores : " + cpu.getPhysicalProcessorCount());
		System.out.println("Logical Cores : " + cpu.getLogicalProcessorCount());
		System.out.println("CPU Frequency : " + cpu.getMaxFreq() / 1_000_000 + " MHz");
	}
}
```


# 5. Disk Usage Information

```java
import oshi.SystemInfo;
import oshi.software.os.FileSystem;
import oshi.software.os.OSFileStore;

public class DiskUsageExample {

	public static void main(String[] args) {

		FileSystem fs = new SystemInfo().getOperatingSystem().getFileSystem();

		for (OSFileStore store : fs.getFileStores()) {

			long total = store.getTotalSpace();
			long free = store.getUsableSpace();

			double usedPercent = ((total - free) * 100.0) / total;

			System.out.println("Drive : " + store.getMount());

			System.out.printf("Used %% : %.2f%n", usedPercent);
		}
	}
}
```

# 6. Network Information

```java
import oshi.SystemInfo;
import oshi.hardware.NetworkIF;

public class NetworkInfo {

	public static void main(String[] args) {

		SystemInfo si = new SystemInfo();

		for (NetworkIF net : si.getHardware().getNetworkIFs()) {

			net.updateAttributes();

			System.out.println("Interface : " + net.getDisplayName());

			System.out.println("Bytes Sent : " + net.getBytesSent());

			System.out.println("Bytes Received : " + net.getBytesRecv());
		}
	}
}
```


# 7. Running Processes

```java
import oshi.SystemInfo;
import oshi.software.os.OSProcess;
import oshi.software.os.OperatingSystem;

import java.util.List;

public class ProcessInfo {

	public static void main(String[] args) {

		OperatingSystem os = new SystemInfo().getOperatingSystem();

		List<OSProcess> processes = os.getProcesses();

		for (OSProcess p : processes.stream().limit(10).toList()) {

			System.out.println(p.getProcessID() + " | " + p.getName() + " | CPU=" + p.getProcessCpuLoadCumulative());
		}
	}
}
```


## Utility to capture Performance Metrics on local system

```java
import oshi.SystemInfo;
import oshi.hardware.*;
import oshi.software.os.*;

public class SystemMetricsUtil {

	public static void main(String[] args) throws Exception {

		SystemInfo si = new SystemInfo();

		GlobalMemory memory = si.getHardware().getMemory();

		long totalMemory = memory.getTotal();
		long availableMemory = memory.getAvailable();

		double memoryUsage = ((totalMemory - availableMemory) * 100.0) / totalMemory;

		CentralProcessor cpu = si.getHardware().getProcessor();

		long[] ticks = cpu.getSystemCpuLoadTicks();

		Thread.sleep(1000);

		double cpuUsage = cpu.getSystemCpuLoadBetweenTicks(ticks) * 100;

		System.out.println("==============================");
		System.out.printf("CPU Usage      : %.2f%%%n", cpuUsage);

		System.out.printf("Memory Usage   : %.2f%%%n", memoryUsage);

		System.out.println("Logical CPUs   : " + cpu.getLogicalProcessorCount());

		System.out.println("Physical CPUs  : " + cpu.getPhysicalProcessorCount());

		System.out.println("==============================");
	}
}
```
