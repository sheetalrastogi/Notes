```java
			// Execute OS command
			Process process = new ProcessBuilder("ipconfig").start();
			BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
			String line;
			while ((line = reader.readLine()) != null) {
				System.out.println(line);
			}
			int exitCode = process.waitFor();
			System.out.println("Command Exit Code : " + exitCode);
```
