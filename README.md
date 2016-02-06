# ev3dev-lang-java

*EV3Dev-lang-Java* is a Java library designed to interact with hardware managed by [EV3Dev](http://www.ev3dev.org/) using the [LeJOS](http://www.lejos.org/) way.

![ScreenShot](https://raw.githubusercontent.com/jabrena/ev3dev-lang-java/master/docs/uml/graph.png)

Execute the following statements to generate the UML diagram in local.

``` bash
git clone https://github.com/jabrena/ev3dev-lang-java.git
cd library
ant -buildfile tools.xml uml
```

# Roadmap

* Support for PiStorm & BrickPi+
* Support for Java 8
* Support for more Gyro Sensors
* Support for RPLidar
* Create repository for navigation
* Create repository for behaviours

# Features

* Regulated Motor Support
* Unregulated Motor Support
* Sensor Support (Few sensors)
* Sounds
* [LeJOS Sensor filter](http://sourceforge.net/p/lejos/wiki/Sensor%20Framework/) Support
* [OpenCV](http://opencv.org/) Computer Vision Support
* [eSpeak](http://espeak.sourceforge.net/) TTS (Text to speech) Support
* Java profiling tools Support ([VisualVM](https://visualvm.java.net/) & [JConsole](http://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html))
* EV3Dev constants autogenerated by [EV3Dev Autogen](https://github.com/ev3dev/ev3dev-lang/tree/develop/autogen)

# Getting Started.

Check if your EV3Brick with EV3Dev need some upgrade:

``` bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
sudo echo LC_ALL="en_GB.utf8" > /etc/environment 
sudo reboot
```

Current development has been tested with this version:

``` bash
root@ev3dev:/home# uname -a                            
Linux ev3dev 3.16.7-ckt21-9-ev3dev-ev3 #1 PREEMPT Tue Dec 15 15:16:17 CST 2015 armv5tejl GNU/Linux
```

If you have your OS for EV3 updated, continue with your Java project.

Create a Java Maven project:

``` bash
mvn archetype:generate 
-DgroupId=ev3dev.examples.demo 
-DartifactId=Test 
-DarchetypeArtifactId=maven-archetype-quickstart 
-DinteractiveMode=false
```

Update the file pom.xml to add the following repository:

``` xml
	<repository>
	    <id>jitpack.io</id>
	    <url>https://jitpack.io</url>
	</repository>
```

and add the dependency to offer Java support for EV3Dev:

``` xml
	<dependency>
	    <groupId>com.github.jabrena</groupId>
	    <artifactId>ev3dev-lang-java</artifactId>
	    <version>v0.2.0</version>
	</dependency>
```

Create a Java Class in your Maven project:


``` java

package ev3dev.examples.demo;

import ev3dev.hardware.Battery;
import ev3dev.hardware.port.MotorPort;
import ev3dev.hardware.port.SensorPort;
import ev3dev.hardware.motor.EV3LargeRegulatedMotor;
import ev3dev.hardware.sensor.ev3.EV3IRSensor;
import lejos.robotics.SampleProvider;
import lejos.utility.Delay;

//java -cp ev3-lang-java-0.2-SNAPSHOT.jar ev3dev.examples.misc.BumperCar
public class BumperCar {
    
    //Robot Definition
    private final static EV3LargeRegulatedMotor mA = new EV3LargeRegulatedMotor(MotorPort.A);
    private final static EV3LargeRegulatedMotor mB = new EV3LargeRegulatedMotor(MotorPort.B);
    private final static EV3IRSensor ir1 = new EV3IRSensor(SensorPort.S1);

    //Configuration
    private final static int motorSpeed = 500;
    
    public static void main(String[] args) {
        
        final SampleProvider sp = ir1.getDistanceMode();
        int distance = 255;

        final int distance_threshold = 35;
        
        //Robot control loop
        final int iteration_threshold = 100;
        for(int i = 0; i <= iteration_threshold; i++) {
            forward();

            float [] sample = new float[sp.sampleSize()];
            sp.fetchSample(sample, 0);
            distance = (int)sample[0];
            if(distance <= distance_threshold){
                backwardWithTurn();
            }

            System.out.println("Iteration: " + i);
            System.out.println("Battery: " + Battery.getInstance().getVoltage());
            System.out.println("Distance: " + distance);
            System.out.println();
        }

        mA.stop();
        mB.stop();
        System.exit(0);
    }
    
    private static void forward(){
        mA.setSpeed(motorSpeed);
        mB.setSpeed(motorSpeed);
        mA.forward();
        mB.forward();
    }
    
    private static void backwardWithTurn(){
        mA.backward();
        mB.backward();
        Delay.msDelay(1000);
        mA.stop();
        mB.stop();
        mA.backward();
        mB.forward();
        Delay.msDelay(1000);
        mA.stop();
        mB.stop();
    }
}


```

To run the example, package your project with Maven to generate a .jar file:

``` bash
mvn package
```

upload your .jar and the library (ev3-lang-java) on your brick. In the path where you have uploaded the jar, execute the following example to run the example:

``` bash
ssh root@192.168.2.3
r00tme
```


``` bash
cd /home
java -cp Test-1.0-SNAPSHOT.jar:ev3-lang-java-0.2-SNAPSHOT.jar ev3dev.examples.demo.BumperCar

```

This example is included in the [test examples](https://github.com/jabrena/ev3dev-lang-java/tree/master/library/src/test/java/ev3dev/examples).

## Docs:

Generate Javadocs in local in a easy way:

``` bash
ant -buildfile tools.xml javadocs
```

## References:

* Maven in 5 Minutes: https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html
* LeJOS: http://www.lejos.org/
* LeJOS Git: http://sourceforge.net/p/lejos/ev3/code/ci/master/tree/ 
* EV3Dev: http://www.ev3dev.org/
* EV3Dev // Getting Started: http://www.ev3dev.org/docs/getting-started/
* EV3Dev autogen: https://github.com/ev3dev/ev3dev-lang/tree/develop/autogen
* Liquid Templates: https://github.com/Shopify/liquid/wiki
