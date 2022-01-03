# QUELL-DESIGN
Design: The control units need to be placed into a deep sleep state under various conditions: 
* When not in use for an extended period of time 
* When disconnected from the network 
* When the connection is unstable. Assuming the connection is UDP based how could we assess the stability? 
* Immediately when placed on the charging dock (2 pins for Vchg & GND)  

They also need to be woken up under certain conditions: 
* When removed from the charging dock 
* After a short press of one of the buttons  


Your task is to design how each of these features may be achieved. We’d be looking for an understanding of how they can be accomplished on the ESP32 platform, preferably with code stubs to support your answer.

--------------------------------------------------------------------------------------------------------

Deep Sleep:
The system needs to be put in deep sleep calling the function esp_deep_sleep_start().
* When not in use for an extended period of time:
  - Could be sensed using IMU.
  - When no button/external interrupts are triggered.
* When disconnected from the network : 
  - ESP SDK has a bucnh of callback functions that can be used to detect states of network in station or AP mode. Deep sleep should not be trigger if the connection is reestabilished before a certain amount of time.
* When the connection is unstable. Assuming the connection is UDP based how could we assess the stability? 
  - This question depends a lot what is the cause of the instability:
  1. Bottle neck in communication for low signal: a priority choice among new packets and queued ones should be done. Does the history need to be kept? Does the data can be grouped to try to minimize the packetization time? Do the send and reception buffers/queue can be increasead?
  2. If it is set as TCP, could it be reinitalized as UDP to avoid communication assertion and delivery? For UDP, extra care need to be implemented (once it has no guarantee of delivery), with checksum and/or crc, packet number, and possibly a small protocol request for lost packets. MQTT has some QoS options, but it is a protocol that requires some RAM.
  3. Code must nove have any blocking routine. If it is waiting for some response, should not get locked, but keep processing if something else ended up coming.
  4. Why to go into deep sleep if the connection is unstable? ESP32 has Modem Sleep only feature as well, which tries to keep the connection alive.
* Immediately when placed on the charging dock (2 pins for Vchg & GND): 
  - ESP32 has two external wakeup pins that can be configured in the bus matrix. The same pin can be used to detect the Vchg as a digital input and detect charging logic state and enter deep sleep.

Wake up:
* When removed from the charging dock:
  - External wakeup (ext0 & ext1) using esp_sleep_enable_ext0_wakeup(GPIO_PIN,LOGIC_LEVEL) API function.
* After a short press of one of the buttons:
  - Same thing as the charging dock, two GPIO could be used, but It would be better to save pins using logic gates, for example.
