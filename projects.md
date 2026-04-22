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

## ECE 1100 Individual Discovery Project: Custom Hardware Synthesizer
*Status: Completed*

### The Problem
Standard computer soundcards handle audio generation invisibly, abstracting away the physical creation of electrical signals. The engineering challenge of this project was to bypass those standard internal systems and build a custom, end-to-end hardware synthesizer from scratch. This required successfully bridging high-level desktop software with low-level embedded hardware. The goal was to create a reliable pipeline where a user interface could dictate precise musical parameters to a microcontroller, which would then mathematically generate and physically construct a raw analog audio wave. Furthermore, because digital microcontrollers output a strictly positive Direct Current (DC) voltage, a major secondary challenge was conditioning that raw electrical signal. Standard consumer audio equipment strictly requires an Alternating Current (AC) signal that swings perfectly above and below 0V, meaning the raw output would cause severe distortion and potential physical damage to the speaker cones if not correctly filtered.

### The Process
To solve this, the synthesizer architecture was divided into a master software controller and a dedicated embedded hardware audio engine.

**The Software Controller (Java)**
The development began by building a Java-based desktop application to serve as the user interface. Using the jSerialComm library managed via Maven in IntelliJ, the Java program identifies the specific USB COM port connected to the Arduino (e.g., COM5). It configures the connection to establish a standard 8N1 serial data link at a baud rate of 115200 to match the hardware. This application opens a communication stream and sets up a standard input listener. When a user types a desired pitch delay number into the console, the Java program packages that integer into a string, adds a newline character (`\n`) for termination, and instantly flushes the data down the USB cable to the hardware.

```java
import com.fazecast.jSerialComm.SerialPort;
import java.io.PrintWriter;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        
        // 1. Force jSerialComm to extract its native DLL locally to bypass potential Windows 'Access is Denied' errors in the Temp folder
        System.setProperty("java.io.tmpdir", System.getProperty("user.dir") + "\\jserial_temp");

        System.out.println("--- Starting Synthesizer Controller ---");

        // 2. Identify available ports and select the first one (assuming it's COM5 for the Arduino Nano)
        SerialPort[] ports = SerialPort.getCommPorts();
        if (ports.length == 0) {
            System.err.println("Error: No serial ports found. Is the Arduino plugged in?");
            return;
        }

        SerialPort arduinoPort = ports[0]; 
        
        // 3. Configure the standard connection parameters to match the Arduino Nano's expected setup
        // Baud rate of 115200, 8 data bits, 1 stop bit, no parity
        arduinoPort.setBaudRate(115200);
        arduinoPort.setNumDataBits(8);
        arduinoPort.setNumStopBits(1);
        arduinoPort.setParity(SerialPort.NO_PARITY);

        // 4. Attempt to open the communication port
        if (!arduinoPort.openPort()) {
            System.err.println("Error: Failed to open port " + arduinoPort.getSystemPortName());
            return;
        }

        System.out.println("Successfully connected to " + arduinoPort.getSystemPortName());

        try {
            // 5. CRITICAL WAIT FOR AUTO-RESET
            // Opening the serial connection forces the Arduino hardware to reboot. 
            // We must pause before sending data, otherwise the Arduino will miss the first command while booting up.
            Thread.sleep(1500);

            // 6. Set up the data stream using a PrintWriter for easy, auto-flushed text transmission
            PrintWriter serialWriter = new PrintWriter(arduinoPort.getOutputStream(), true); 

            System.out.println("\n--- Synthesizer is Ready! ---");
            System.out.println("Type a delay number to change the pitch (Example: 50 = high, 250 = low).");
            System.out.println("Type '0' to quit.\n");

            // 7. Input loop: Read user commands from the console and send them to the hardware
            Scanner inputScanner = new Scanner(System.in);
            while (true) {
                System.out.print("Enter pitch delay: ");
                String userInput = inputScanner.nextLine();
                
                if (userInput.equals("0")) {
                    break;
                }

                try {
                    // Send the number. Important: the C++ code relies on the implicit newline char added by println()
                    serialWriter.println(userInput);
                    System.out.println("Sent command: " + userInput);
                    
                } catch (Exception e) {
                    System.err.println("Error sending data.");
                }
            }
            inputScanner.close();

        } catch (InterruptedException e) {
            System.err.println("Thread interrupted.");
        } finally {
            // 8. Clean up: Always close the port when the program exits to free the hardware resource
            if (arduinoPort.isOpen()) {
                arduinoPort.closePort();
                System.out.println("Synthesizer disconnected.");
            }
        }
    }
}
```

The Hardware Engine (C++ / Arduino)
The receiving end of this pipeline is the Arduino Nano, which runs a non-blocking embedded C++ loop.

The software uses the concept of a pre-calculated "wavetable"—an array containing 64 integers representing a perfectly smooth sine wave stored directly in the microcontroller's limited SRAM. This allows the audio engine to run at blistering speeds without wasting precious clock cycles calculating complex trigonometry on the fly.
```C++

#include <Wire.h>
#define DAC_ADDR 0x60 // Standard I2C address for the MCP4725 DAC chip breakout board

const int TABLE_SIZE = 64; 
int waveTable[TABLE_SIZE];
int tableIndex = 0;

// This variable dictates the speed of wavetable playback (lower = higher pitch)
// This is the variable updated in real-time by the Java controller over USB.
int sampleDelay = 100; 
unsigned long lastSampleTime = 0;

void setup() {
  Serial.begin(115200); // Initialize high-speed USB serial connection to accept Java GUI commands
  
  // Initialize the I2C communication protocol at its maximum speed (400kHz) for stutter-free audio output
  Wire.begin();
  Wire.setClock(400000); 

  // Pre-calculate a smooth sine wave and store the values in memory (a wavetable).
  // The amplitude is intentionally scaled down to 1000 (of the 4095 maximum) to keep
  // the resulting voltage at a safe line-level for consumer audio receivers.
  for (int i = 0; i < TABLE_SIZE; i++) {
    waveTable[i] = (int)(1000.0 * sin(2.0 * PI * i / TABLE_SIZE) + 1000.0);
  }
}

void loop() {
  // 1. LISTEN FOR COMMANDS
  // Check the USB input buffer for pitch change commands sent from the Java application
  if (Serial.available() > 0) {
    // Read the incoming string until the newline character is found and parse the integer
    int newDelay = Serial.parseInt();
    if (newDelay > 0) {
      sampleDelay = newDelay; // Instantly update the global playback speed variable
    }
  }

  // 2. EMBEDDED AUDIO LOOP
  // Non-blocking loop: compare the current time against the last update time and the sampleDelay
  if (micros() - lastSampleTime >= sampleDelay) {
    lastSampleTime = micros();
    
    // Rapidly transmit the 12-bit audio sample to the external DAC chip using the I2C bus
    Wire.beginTransmission(DAC_ADDR);
    // Send the 12-bit value in two distinct write operations
    Wire.write((waveTable[tableIndex] >> 8) & 0x0F); // Send the upper 4 bits
    Wire.write(waveTable[tableIndex] & 0xFF);        // Send the lower 8 bits
    Wire.endTransmission();

    // Increment the pointer to the next point in the wavetable, wrapping around at the end
    tableIndex++;
    if (tableIndex >= TABLE_SIZE) {
      tableIndex = 0;
    }
  }
}
```
The C++ loop dictates the exact speed at which it reads through this wavetable array based on the sampleDelay provided by the Java command. It then streams those digital numbers over the I2C protocol at 400kHz to an external MCP4725 Digital-to-Analog Converter (DAC) chip mounted on the breadboard. The DAC serves as the crucial physical interface, rapidly translating those 12-bit binary values back into physical electrical voltages, essentially drawing the continuous analog soundwave in real-time.
The Outcome

While the DAC successfully generates the physical soundwave, the raw output fluctuates strictly between 0V and 5V. Because consumer audio equipment requires an Alternating Current (AC) signal centered at 0V, this raw output possesses a permanent DC offset (a constant positive voltage baseline). If plugged directly into an amplifier, this DC voltage would force the speaker cones to strain unhealthily and cause severe, unmusical buzzing.

To resolve this critical final hardware bottleneck, the analog signal path incorporates an AC coupling stage. By integrating an electrolytic capacitor inline between the DAC chip's output and the 3.5mm stereo jack, the capacitor acts as a physical wall, completely blocking the harmful Direct Current component while allowing the fluctuating AC audio frequencies to pass through unharmed. This conditioned waveform is then bridged to both left and right channels on the output jack. The end result is a highly responsive, end-to-end hardware synthesis system that successfully translates software instructions into pure, clean analog audio.

<details>
<summary style="cursor: pointer; color: #2B7CBA; font-weight: bold; margin-bottom: 10px;">Watch the synthesizer demo!</summary>
<iframe width="560" height="315" src="https://www.youtube.com/embed/YOUR_VIDEO_ID" title="Hardware Synthesizer Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="border-radius: 8px; margin-top: 10px; max-width: 100%;"></iframe>
</details>
