[Home](index.html) | [About Me](about.html) | [Resume](resume.html) | [Career Goals](goals.html) | <span style="color: #888; font-weight: bold;">Technical Projects</span>

---

# Technical Projects

## RoboNav - Electrical Subteam
*August 2025 - Present*
As part of a student organization focused on designing autonomous navigation systems for ground and aerial robots, I engineer end-to-end electromechanical subsystems. This includes initial component selection, circuit design, and the seamless integration of hardware and mechanical assemblies. I also developed embedded firmware to program microcontroller logic, process sensor data, and implement precise control systems for various motors, including DC, servo, and stepper.
<details>
  <summary style="cursor: pointer; color: #2B7CBA; font-weight: bold; margin-bottom: 10px;">See the rover!</summary>
  <img src="rover.jpg" alt="Autonomous Navigation Rover" width="600" style="border-radius: 8px; margin-top: 10px; max-width: 100%; border: 1px solid #ddd; box-shadow: 2px 2px 10px rgba(0,0,0,0.1);">
</details>

## Lighter Than Air Autonomy - Computer Vision Subteam
*August 2025 - Present*
I contribute to the research and development of lighter-than-air aerial vehicles (blimps), focusing on navigation and indoor flight autonomy. I implemented computer vision pipelines in Python using the OpenCV library to process, detect, and track visual features for autonomous vehicles. By analyzing camera data, I helped improve localization, obstacle awareness, and system robustness, collaborating with autonomy controls teams to integrate vision outputs into real-time decision-making.
<details>
  <summary style="cursor: pointer; color: #2B7CBA; font-weight: bold; margin-bottom: 10px;">See the github repo!</summary>
  <a href="https://github.com/BUZZ-Blimps" target="_blank" rel="noopener noreferrer">
    <img src="https://images.unsplash.com/photo-1618401471353-b98afee0b2eb?ixlib=rb-4.0.3&auto=format&fit=crop&w=800&q=80" alt="BUZZ-Blimps GitHub Repository" width="600" style="border-radius: 8px; margin-top: 10px; max-width: 100%; border: 1px solid #ddd; box-shadow: 2px 2px 10px rgba(0,0,0,0.1); transition: transform 0.2s;">
  </a>
</details>

## MyWatkinsville
*May 2025*
I developed MyWatkinsville, a geolocation-driven mobile app designed to enhance cultural engagement in Watkinsville, GA. Built using Jetpack Compose, JavaScript, and Kotlin, the app integrates the Google Maps API for core location services and utilizes a Firebase backend to manage and sync real-time event and business data. The application achieved over 500 downloads, was officially adopted by the local tourism department, and earned a second-place finish in the Congressional App Challenge.
<details>
  <summary style="cursor: pointer; color: #2B7CBA; font-weight: bold; margin-bottom: 10px;">Watch my submission video!</summary>
  <iframe width="560" height="315" src="https://www.youtube.com/embed/F3sG871hs7o" title="MyWatkinsville Submission Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="border-radius: 8px; margin-top: 10px; max-width: 100%;"></iframe>
</details>

## EV Delivery Fleet Feasibility Study
*May 2025*
I conducted a comprehensive feasibility study assessing the economic and logistical viability of transitioning commercial delivery fleets to electric vehicles (EVs). Leveraging PyTorch and NumPy, I processed, cleaned, and analyzed large datasets to perform numerical simulations modeling vehicle range, operational costs, and charging requirements. I also created compelling data visualizations using MATLab and Seaborn to illustrate model outputs and key financial metrics within the final feasibility report.
<details>
  <summary style="cursor: pointer; color: #2B7CBA; font-weight: bold; margin-bottom: 10px;">See the data!</summary>
  <img src="EVGraph.png" width="600" style="border-radius: 8px; margin-top: 10px; max-width: 100%; border: 1px solid #ddd; box-shadow: 2px 2px 10px rgba(0,0,0,0.1);">
</details>

## ECE 1100 Individual Discovery Project
*Status: In Progress*

## Custom Hardware Synthesizer
*Status: Completed*

---

### The Problem

Standard computer soundcards handle audio generation invisibly, abstracting away the physical creation of electrical signals. The engineering challenge of this project was to bypass those standard internal systems and build a custom, end-to-end hardware synthesizer from scratch. This required successfully bridging high-level desktop software with low-level embedded hardware. The goal was to create a reliable pipeline where a user interface could dictate precise musical parameters to a microcontroller, which would then mathematically generate and physically construct a raw analog audio wave. Furthermore, because digital microcontrollers output a strictly positive Direct Current (DC) voltage, a major secondary challenge was conditioning that raw electrical signal. Standard consumer audio equipment strictly requires an Alternating Current (AC) signal that swings perfectly above and below 0V, meaning the raw output would cause severe distortion and potential physical damage to the speaker cones if not correctly filtered.

---

### The Process

To solve this, the synthesizer architecture was divided into a master software controller and a dedicated embedded hardware audio engine.

---

### The Software Controller (Java)

The development began by building a Java-based desktop application to serve as the user interface. Using the jSerialComm library managed via Maven in IntelliJ, the Java program identifies the specific USB COM port connected to the Arduino (e.g., COM5). It configures the connection to establish a standard 8N1 serial data link at a baud rate of 115200 to match the hardware. This application opens a communication stream and sets up a standard input listener. When a user types a desired pitch delay number into the console, the Java program packages that integer into a string, adds a newline character (\n) for termination, and instantly flushes the data down the USB cable to the hardware.

<details>
<summary><b>View Java Code</b></summary>

```java
import com.fazecast.jSerialComm.SerialPort;
import java.io.PrintWriter;
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        
        System.setProperty("java.io.tmpdir", System.getProperty("user.dir") + "\\jserial_temp");

        System.out.println("--- Starting Synthesizer Controller ---");

        SerialPort[] ports = SerialPort.getCommPorts();
        if (ports.length == 0) {
            System.err.println("Error: No serial ports found. Is the Arduino plugged in?");
            return;
        }

        SerialPort arduinoPort = ports[0]; 
        
        arduinoPort.setBaudRate(115200);
        arduinoPort.setNumDataBits(8);
        arduinoPort.setNumStopBits(1);
        arduinoPort.setParity(SerialPort.NO_PARITY);

        if (!arduinoPort.openPort()) {
            System.err.println("Error: Failed to open port " + arduinoPort.getSystemPortName());
            return;
        }

        System.out.println("Successfully connected to " + arduinoPort.getSystemPortName());

        try {
            Thread.sleep(1500);

            PrintWriter serialWriter = new PrintWriter(arduinoPort.getOutputStream(), true); 

            Scanner inputScanner = new Scanner(System.in);
            while (true) {
                String userInput = inputScanner.nextLine();
                
                if (userInput.equals("0")) {
                    break;
                }

                try {
                    serialWriter.println(userInput);
                } catch (Exception e) {
                    System.err.println("Error sending data.");
                }
            }
            inputScanner.close();

        } catch (InterruptedException e) {
            System.err.println("Thread interrupted.");
        } finally {
            if (arduinoPort.isOpen()) {
                arduinoPort.closePort();
                System.out.println("Synthesizer disconnected.");
            }
        }
    }
}
```
</details>
