## vehicle tracking simulation with Appium
--------------------------------------------

Vehicle identifier
Route points
Travel speed
Timestamp updates
Continuous movement along the route

```java

package assurant;

Example: Simulate Vehicle Moving Along a Route
import org.openqa.selenium.html5.Location;

public class VehicleTracker {

    public static void simulateVehicleRoute(AndroidDriver driver)
            throws InterruptedException {

        // Delhi route (sample vehicle journey)
        double[][] route = {
                {28.6139, 77.2090}, // Start
                {28.6145, 77.2100},
                {28.6150, 77.2110},
                {28.6160, 77.2125},
                {28.6175, 77.2140},
                {28.6190, 77.2160}  // Destination
        };

        String vehicleId = "VEH-1001";

        for (int i = 0; i < route.length; i++) {

            double latitude = route[i][0];
            double longitude = route[i][1];

            System.out.printf(
                    "[%s] Location Update %d -> Lat: %.6f, Long: %.6f%n",
                    vehicleId,
                    i + 1,
                    latitude,
                    longitude);

            driver.setLocation(
                    new Location(
                            latitude,
                            longitude,
                            10));

            // Simulate vehicle movement every 10 seconds
            Thread.sleep(10000);
        }

        System.out.println(
                vehicleId + " reached destination.");
    }
}

```

## Simulate a vehicle movement between cities

```java
package assurant;

public class abc {

	public static void main(String[] args) {
		Simulate a Vehicle Trip Between Two Cities
		double[][] route = {

		        // Noida
		        {28.5355, 77.3910},

		        // Greater Noida
		        {28.4744, 77.5040},

		        // Jewar
		        {28.1220, 77.5570},

		        // Agra
		        {27.1767, 78.0081}
		};

		for (double[] point : route) {

		    driver.setLocation(
		            new Location(
		                    point[0],
		                    point[1],
		                    10));

		    System.out.println(
		            "Vehicle moved to : "
		                    + point[0] + ", "
		                    + point[1]);

		    Thread.sleep(15000);
		}


	}

}

```
