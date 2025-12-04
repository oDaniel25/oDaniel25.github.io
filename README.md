# Blood Glucose Monitor
**Company Synopsis**

Blood glucose monitors are one of the main devices that allow diabetics to live normal lives 
with their condition. Although they are extremely beneficial, the leading brands, like Dexcom, only
give users updates to their blood sugar levels once every 5 minutes. Although this is reliable in most 
situations, there are often moments where being able to check glucose levels immediatly (without finger pokes)
is necessary. My company is designing monitors that enable this functionality on top of the regular
updates every few minutes. It also comes with a web page that family and doctors can subscribe to 
to recieve updates and potential urgent messages. Real time is important for this project since 
a user would not want any delays when calling for help with the urgent button or checking their blood sugar.

**Scheduler Fit**

One of the most crucial timing relationships is that between the urgent LED and button. The LED would
have to change its behavior to the emergency mode in less than a millisecond of the button press. 
This is upheld in the simulation as it only took 890us for the LED to change. The other hard deadline
was not measured as there is no signal for a print from the manual read button.


**Race-Proofing**

Normally, a trace program would occur with the automatic and manual reads. This is because there is
a possibility that both would try to read the analog value at the same time. I avoided this by implementing 
a pair of binary semaphores so that only one task can read at a time.
```
    if (xSemaphoreTake(autoSem, 0)){
        xSemaphoreTake(serialMutex, portMAX_DELAY);
        sensorVal = analogRead(SENSOR_PIN)/12;

        if(sensorVal>=SENSOR_THRESHOLD)Serial.print("High Blood Sugar! --->");
        else if(sensorVal<=SENSOR_MIN) Serial.print("Low Blood Sugar! --->");    
        Serial.print(sensorVal);
        Serial.println("mg/dL");
        xSemaphoreGive(serialMutex);

    }
    if (xSemaphoreTake(manSem, 0)){
        xSemaphoreTake(serialMutex, portMAX_DELAY);
        sensorVal = analogRead(SENSOR_PIN)/12;
        Serial.print("Manual Reading-->");

        if(sensorVal>=SENSOR_THRESHOLD)Serial.print("High Blood Sugar! --->");
        else if(sensorVal<=SENSOR_MIN) Serial.print("Low Blood Sugar! --->");    
        Serial.print(sensorVal);
        Serial.println("mg/dL");
        xSemaphoreGive(serialMutex);

    }

```

**Worst-Case Spike**

The highest load I could put on the system was when I spammed the urgent button. This didn't
result in much since I made the delay for each button press 150ms. This delay prevented any negative affects
from repeated presses.


**Design Trade-Off**

As stated before, I added a sizeable delay to the button press ISR. This prevents any extremely 
quick spamming, but this is fine as a user isn't expected to be needing repeated presses in a row.
This also makes meeting timing constraints much easier as it prevents button presses from delaying the
rest of the system. This is extremely important as it is on an ISR and it would be an issue if it 
held everything up.
