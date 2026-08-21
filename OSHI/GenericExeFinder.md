# Generic approach to search an executable on local file system:
 - The current PATH environment variable
 - Common Windows executable locations
 - Any executable name supplied by the user

# Generic Executable Finder

```java
import java.io.File;

public class ExecutableFinder {
	public static String findExecutable(String executableName) {
		// Add .exe if missing
		if (!executableName.toLowerCase().endsWith(".exe")) {
			executableName += ".exe";
		}

		// Search PATH variable
		String path = System.getenv("PATH");
		if (path != null) {
			String[] directories = path.split(File.pathSeparator);
			for (String dir : directories) {
				File executable = new File(dir, executableName);
				if (executable.exists() && executable.isFile()) {
					return executable.getAbsolutePath();
				}
			}
		}
		return null;
	}

	public static void main(String[] args) {
		String executable = findExecutable("ping");
		if (executable != null) {
			System.out.println("Found: " + executable);
		} else {
			System.out.println("Executable not found.");
		}
	}
}
```

**Usage**

findExecutable("ping");
findExecutable("java");
findExecutable("git");
findExecutable("curl");
findExecutable("notepad");
findExecutable("cmd");

