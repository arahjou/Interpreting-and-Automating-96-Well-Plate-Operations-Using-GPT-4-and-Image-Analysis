# Interpreting-and-Automating-96-Well-Plate-Operations-Using-GPT-4-and-Image-Analysis

In my recent interaction with a multimodal language model, I experienced firsthand the advanced capabilities of modern AI. I presented the model with an image of a 96-well plate and provided contextual information about it. Impressively, not only did the model generate a Python script to automate the liquid dispensing process based on the details I provided, but it also inferred which wells were filled by analyzing color differences in the image.

Me:
 

The image displays a 96 wells plate with the center of the A1 well and 1 cm above being the zero coordinate. A needle's bottom, at this coordinate, is linked to a syringe holding a liquid. This syringe, controlled by an electric engine, dispenses 1 ul of liquid every second. Additionally, two engines enable the syringe's movement along the x and y axes, moving at 5 mm/s. To deposit liquid, the needle's bottom aligns with a well's center. Each well's center is spaced 0.5 cm apart. I'll now provide a python script that prompts for well numbers and liquid volume, moving the engine accordingly. The engines are managed by a raspberry pi, with connections specified for each axis and liquid control.

GPT-4:

```python
import time
import RPi.GPIO as GPIO

# Setup
GPIO.setmode(GPIO.BCM)
GPIO.setup(1, GPIO.OUT)  # Liquid control
GPIO.setup(2, GPIO.OUT)  # X-axis control
GPIO.setup(3, GPIO.OUT)  # Y-axis control

def move_to_well(well):
    x_distance = (ord(well[0]) - ord('A')) * 0.5
    y_distance = (int(well[1:]) - 1) * 0.5
    move_x_axis(x_distance)
    move_y_axis(y_distance)

def move_x_axis(distance):
    # Assuming positive direction for forward and negative for backward
    direction = 1 if distance > 0 else -1
    GPIO.output(2, direction)
    time.sleep(abs(distance) / 5)  # Moving 5mm/sec
    GPIO.output(2, 0)

def move_y_axis(distance):
    direction = 1 if distance > 0 else -1
    GPIO.output(3, direction)
    time.sleep(abs(distance) / 5)  # Moving 5mm/sec
    GPIO.output(3, 0)

def add_liquid(volume):
    GPIO.output(1, 1)  # Start the liquid engine
    time.sleep(volume / 1)  # 1ul/sec
    GPIO.output(1, 0)  # Stop the liquid engine

def return_to_origin(current_well):
    x_distance = ord('A') - ord(current_well[0])
    y_distance = 1 - int(current_well[1:])
    move_x_axis(x_distance * 0.5)
    move_y_axis(y_distance * 0.5)

def main():
    wells = input("Enter well numbers separated by commas (e.g. A1, B2): ").split(',')
    volumes = list(map(float, input("Enter corresponding volumes separated by commas (e.g. 10, 20): ").split(',')))

    for well, volume in zip(wells, volumes):
        well = well.strip().upper()
        move_to_well(well)
        add_liquid(volume)
        return_to_origin(well)

    GPIO.cleanup()

if __name__ == "__main__":
    main()
```

With this updated code, the user can enter multiple well names and volumes at once. The program will then sequentially move to each well, dispense the specified volume, and return to the origin after each process.

Me:

 

Given that the liquid is pink, can we identify a well that has been successfully filled?

GPT-4:

Based on the image provided, the wells with a successful liquid filling process (indicated by the pink color) are:

- B2
- D5
- G8

These wells are the ones that are filled with the pink liquid.

Me:

Can we determine if they have equal volumes of liquid, considering that varying liquid levels alter the color intensity?

GPT-4:

The image does not provide quantitative information about the volume of the liquid in each well. However, if different volumes of liquid lead to different color intensities, and all the pink-colored wells have the same intensity, then we can infer that they have equal volumes of liquid. 

From the image, the pink color in wells B2, D5, and G8 appears to be of similar intensity, which suggests that they likely have equal volumes of liquid. However, to be certain, you would need to use a more quantitative method or tool to measure the liquid volume in each well.
![image](https://github.com/arahjou/Interpreting-and-Automating-96-Well-Plate-Operations-Using-GPT-4-and-Image-Analysis/assets/88965480/f1eaacd9-4662-4bc2-8522-da00a9a2f537)
