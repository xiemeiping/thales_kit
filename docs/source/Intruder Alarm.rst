Intruder Alarm
==========================================

In the previous chapters, we used simple electronic components (such as LED, button). This time we will use the sensor module - PIR.

Passive infrared sensor (PIR sensor) is a common sensor that can measure infrared (IR) light emitted by objects in its field of view.
Simply put, it will receive infrared radiation emitted from the body, thereby detecting the movement of people and other animals.
More specifically, it tells the main control board that someone has entered your room.

:ref:`PIR Motion Sensor`

Now, let's use PIR and active buzzer to build an Intruder Alarm.

Wiring
-------------------------------------------

Two types of buzzers are included in the kit. We need to use active buzzer. Turn them around, the sealed back (not the exposed PCB) is the one we want.

.. image:: img/buzzer.png

The buzzer needs to use a transistor when working, here we use S8050.

.. image:: img/wiring_intruder_alarm.png

1. Connect 3V3 and GND of Pico to the power bus of the breadboard.
#. Connect the positive pin of the buzzer to the positive power bus.
#. Connect the cathode pin of the buzzer to the **collector** lead of the transistor.
#. Connect the base lead of the transistor to the GP15 pin through a 1kΩ resistor.
#. Connect the **emitter** lead of the transistor to the negative power bus.
#. Connect the OUT of PIR to the GP14 pin, VCC to the positive power bus, and GND to the negative power bus.

.. note::
    The color ring of the 1kΩ resistor is brown, black, black, brown and brown.

Code
--------------------------------------------

When the program is executed, if someone walks into the PIR detection range, the buzzer will be'BEEP BEEP' for 5 seconds!

.. code-block:: python

    import machine
    import utime

    pir_sensor = machine.Pin(14, machine.Pin.IN)
    buzzer = machine.Pin(15, machine.Pin.OUT)    

    def motion_detected(pin):
        for i in range(50):
            buzzer.toggle()
            utime.sleep_ms(100)

    pir_sensor.irq(trigger=machine.Pin.IRQ_RISING, handler=motion_detected)
    print("Intruder Alarm Start!")




What more?
-------------------------------------

PIR is a very sensitive sensor. In order to adapt it to the environment of use, it needs to be adjusted. Let the side with the 2 potentiometers facing you, turn both potentiometers counterclockwise to the end and insert the jumper cap on the pin with L and the middle pin.

Copy the following code into Thonny and run it, let us analyze its adjustment method along with the experimental results.

.. code-block:: python

    import machine
    import utime

    pir_sensor = machine.Pin(14, machine.Pin.IN)

    global timer_delay
    timer_delay = utime.ticks_ms()
    print("start")

    def pir_in_high_level(pin):
        global timer_delay    
        pir_sensor.irq(trigger=machine.Pin.IRQ_FALLING, handler=pir_in_low_level)    
        intervals = utime.ticks_diff(utime.ticks_ms(), timer_delay)
        timer_delay = utime.ticks_ms()
        print("the dormancy duration is " + str(intervals) + "ms")

    def pir_in_low_level(pin):
        global timer_delay    
        pir_sensor.irq(trigger=machine.Pin.IRQ_RISING, handler=pir_in_high_level) 
        intervals2 = utime.ticks_diff(utime.ticks_ms(), timer_delay)
        timer_delay = utime.ticks_ms()        
        print("the duration of work is " + str(intervals2) + "ms")

    pir_sensor.irq(trigger=machine.Pin.IRQ_RISING, handler=pir_in_high_level) 

.. image:: img/pir_back.png

1. Trigger Mode

    Let's take a look at the pins with jumper cap at the corner.
    It allows PIR to enter Repeatable trigger mode or Non-repeatable trigger mode

    At present, our jumper cap connects the middle Pin and L Pin, which makes the PIR in non-repeatable trigger mode.
    In this mode, when the PIR detects the movement of the organism, it will send a high-level signal for about 2.8 seconds to the main control board.
    We can see in the printed data that the duration of work will always be around 2800ms.

    Next, we modify the position of the lower jumper cap and connect it to the middle Pin and H Pin to make the PIR in repeatable trigger mode.
    In this mode, when the PIR detects the movement of the organism (note that it is movement, not static in front of the sensor), as long as the organism keeps moving within the detection range, the PIR will continue to send a high-level signal to the main control board.
    We can see in the printed data that the duration of work is an uncertain value.

#. Delay Adjustment

    The potentiometer on the left is used to adjust the interval between two jobs.
    
    At present, we screw it counterclockwise to the end, which makes the PIR need to enter a sleep time of about 5 seconds after finishing sending the high level work. During this time, the PIR will no longer detect the infrared radiation in the target area.
    We can see in the printed data that the dormancy duration is always no less than 5000ms.

    If we turn the potentiometer clockwise, the sleep time will also increase. When it is turned clockwise to the end, the sleep time will be as high as 300s.

#. Distance Adjustment

    The centered potentiometer is used to adjust the sensing distance range of the PIR.

    Turn the knob of the distance adjustment potentiometer **clockwise** to increase the sensing distance range, and the maximum sensing distance range is about 0-7 meters.
    If it rotates **counterclockwise**, the sensing distance range is reduced, and the minimum sensing distance range is about 0-3 meters.