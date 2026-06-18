# EventBoard-RTC-Driven-Message-Display-System
The EventBoard - RTC-Driven Message Display System is an LPC2148-based system that uses an on-chip RTC to trigger scheduled messages and scroll on a 16x2 LCD. It includes an Admin Mode by using External interrupt for time and message management via a 4x4 keypad and switch. The system uses an LM35 sensor for temperature monitoring.  

## Features  
1.Real-Time Message Scheduling.  
2.Interactive Admin Mode.  
3.Sensor Integration.  
4.Input Handling.    
## Required Components  
1.LPC2148 Board  
2.LCD(16x2)  
3.Keypad Matrix  
4.LM35  
5.On chip ADC  
6.On chip RTC  
7.Switch  
## Tools  
Compiler: Keil uVision4  
Flashing Tools: Flash Magic  

## System Architecture and Component roles    
1.The Central Controller: LPC2148  
The LPC2148 manages the entire execution flow. It handles:   
Peripheral Management: Managing the GPIO peripherals required to interface like the LCD, RTC, ADC, Keypad and LM35 sensor.    
Logic Processing: Running the main control loop for device state management, including message scheduling and environment monitoring.    
2.Data Management:  
Keypad Matrix: Acts as the primary input interface. It provides a tactile interface for the user to navigate the system, select active messages, and edit the   current time in Admin Mode.  
3.User Interface:  
LCD Module: Acts as the Visual Dashboard, providing immediate feedback to the user. It displays scheduled messages using a scrolling mechanism, the current   time,date,day and room temperature.  
4.Environmental Monitoring:  
LM35 Sensor: Communicates with the controller via the on-chip ADC. This allows the system to measure and display ambient room temperature when no message is   scheduled.  
## Set-Up Instructions  
1.Before using the Peripherals you must initialize the peripherals by calling InitLCD(),InitKPM(),RTC_Init(),Init_ADC().  
2.Notes that you must include all the required headers like "LPC21xx.h","string.h" and user defined  hearders like                                  -----------"lcd_defines.h","lcd.h","delay.h","types.h","defines.h","KPM.h","KPM_defines.h","ADC.h","minimain_rtc.h".  
>>"lcd_defines" contains the command values and the pin connections of LCD.  
>>"lcd.h" contains the function declarations of LCD.  
>>"delay.h" contains the function declarations of delay functions according to the time of delay required.    
>>"types.h" contains the type casted details of the existing data types.  
>>"defines.h" contains the macro expansions of Bit/Byte manipulation.  
>>"KPM.h" contains the function declarations of the Keypad related operations.  
>>"KPM_defines.h" contains the Pin connections of the Keypad.  
>>"ADC.h" contains the function declarations of the ADC related functions.  
>>"minimain_rtc.h" contains the function declaations of the RTC related functions.  
## Code Flow Execution  
Main block:  
FUNCTION main():  
INITIALIZE peripherals (LCD, RTC, ADC, KPM)  
ENABLE EINT0 interrupt  
SET RTC to predefined time and date  

LOOP FOREVER:  
    SET flag = 0    
    FOR i FROM (totalmsgs - 1) DOWN TO 0:    
        IF (RTC.hour == msglist[i].hour) AND     
           (RTC.min >= msglist[i].min) AND  
           (RTC.min <= msglist[i].min + 14):    
              
            IF msglist[i].enabled == 1:  
                COPY msglist[i].text TO buffer     
            ELSE:  
                COPY "                   " TO buffer  
              
            SET flag = 1  
            BREAK loop  
      
    IF flag == 1:  
        CALL scrolllcd(buffer, i)    
    ELSE:  
        GET current RTC time, date, day    
        READ ADC value  
        DISPLAY RTC info, Date, Day, and ADC on LCD    
ISR BLock:  
FUNCTION eint0_isr():  
DISPLAY menu: "1.RTC, 2.msg, 3.ext"  
READ option from Keypad (op)  

SWITCH op:  
    CASE '1':     
        INPUT new Time (H, M, S) -> SET RTC Time    
        INPUT new Date (D, M, Y) -> SET RTC Date  
        INPUT Day -> SET RTC Day  
          
    CASE '2':  
        INPUT message Index    
        INPUT enable status (1 or 0)    
        UPDATE msglist[index].enabled   
          
    CASE '3':  
        EXIT  
          
CLEAR interrupt flag (EXTINT)    
RESET vector address (VICVectAddr)      
## Pin Connections  
1.Connect P0.1 to the switch for an External interrupt, because P0.1 supports EINT0 for the 4th functionality.  
2.Connect P0.8 to P0.15 to the LCD pins.  
3.Connect the P0.17 to the RS(Register Select) pin of the LCD.  
4.Connect the P0.18 to the EN(Enable) pin of the LCD.  
5.Connect the P0.28 to the Output pin of the LM35 sensor, because P0.28 supports the Channel-1, we are connecting to the Channel-1 because in LPC2148 there is no   Channel-0.  
6.Connect the pins P1.16 to P1.23 to the Keypad Matrix.  
7.Connect the VCC and GND pins of the ADC to the 3.3V and the GND respectively.  
## How to Use  
1.System Power-Up:
Upon connecting the power supply after Loading the code into the hardware, the system will initialize all peripherals.
2.Operation:  
Once the system is powered on, it will automatically cycle through the following modes:  
>>Normal Mode: Displays the current time, date, day, and room temperature (ADC reading).  
>>Event Mode: When the RTC matches a scheduled message time, the system automatically switches to Scrolling Mode, displaying the event text on the first line and a   15-minute countdown timer on the second line.  
>>Interrupt Mode:When the Switch is pressed which is connected to the EINT0 (P0.1) Button at any time the interrupt is triggered.  
-> Option 1 (RTC): Use the keypad to update the current time, date, and day manually.  
-> Option 2 (Msg): Enter the message index to toggle it Enabled (1) or Disabled (0).  
-> Option 3 (Exit): Return to the main display.  
3.Implementation Details:  
>>Scrolling Logic: The code handles both short strings (<= 24 chars) and long strings (> 24 chars) using a buffer window method to ensure smooth display on the 16x2   LCD.  
>>Interrupt-Driven: The configuration menu is handled via EINT0_ISR, ensuring that system settings can be modified without resetting the controller.  
## Future Improvements  
1.EEPROM Persistence: Currently, all settings (time/messages) are lost upon power reset. Implementing I2C EEPROM storage will allow the system to save user-defined   schedules and RTC configurations permanently.  
2.Secured Admin Mode: Integrating a 4-digit PIN authentication adds a professional security layer that prevents unauthorized access to system settings. By masking     input with asterisks, the system ensures sensitive configuration data remains protected from casual observers. This feature transforms the device from as   impledisplay tool into a secure administrative platform suitable for academic or office environments.  
3.Sensor Integration: Expand the ADC module to include more environmental sensors (e.g., humidity or light sensors) and display them in a user-selectable toggle   mode on the second line.  
## Block Diagram  
<img width="1327" height="796" alt="Screenshot (2)" src="https://github.com/user-attachments/assets/b15048f5-26e2-429d-bbf8-4074fe2f82a9" />

