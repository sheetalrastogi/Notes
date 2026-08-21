## Checks Common Windows Locations (C:\windows32, c:\system32 etc)


```java
import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class ExecutableFinder {

	public static String findExecutable(String executableName) {

		if (!executableName.toLowerCase().endsWith(".exe")) {
			executableName += ".exe";
		}

		List<String> searchLocations = new ArrayList<>();

		// PATH locations
		String path = System.getenv("PATH");
		if (path != null) {
			String[] dirs = path.split(File.pathSeparator);
			for (String dir : dirs) {
				searchLocations.add(dir);
			}
		}

		// Common Windows locations
		searchLocations.add(System.getenv("SystemRoot") + "\\System32");
		searchLocations.add(System.getenv("SystemRoot"));
		searchLocations.add("C:\\Program Files");
		searchLocations.add("C:\\Program Files (x86)");

		for (String dir : searchLocations) {
			File executable = new File(dir, executableName);
			if (executable.exists() && executable.isFile()) {
				return executable.getAbsolutePath();
			}
		}

		return null;
	}

	public static void main(String[] args) {

		String exe = findExecutable("ping");

		System.out.println(exe != null ? "Found: " + exe : "Executable not found");
	}
}
```


# Recursive Search Anywhere on Local Drives
If you want to locate an executable even when it is not in PATH, search all drives:

```java
import java.io.File;

public class DeepExecutableSearch {

	public static String findExecutable(String executableName) {
		if (!executableName.endsWith(".exe")) {
			executableName += ".exe";
		}

		for (File root : File.listRoots()) {
			String result = search(root, executableName);
			if (result != null) {
				return result;
			}
		}

		return null;
	}

	private static String search(File dir, String executableName) {

		try {
			File[] files = dir.listFiles();
			if (files == null) {
				return null;
			}
			for (File file : files) {
				if (file.isFile() && file.getName().equalsIgnoreCase(executableName)) {
					return file.getAbsolutePath();
				}
				if (file.isDirectory()) {
					String result = search(file, executableName);
					if (result != null) {
						return result;
					}
				}
			}

		} catch (Exception ignored) {
		}

		return null;
	}

	public static void main(String[] args) {

		String exe = findExecutable("ping");

		System.out.println(exe != null ? "Found: " + exe : "Executable not found");
	}
}
```
