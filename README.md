# touch-sensored-stepper-motor
from gpiozero import Button, OutputDevice as Stepper from signal import pause from time import sleep

STEPPER_PINS = [17, 27, 22, 23] TOUCH_PIN = 4 STEPS_PER_REV = 4096
STEP_DELAY = 0.005
QUARTER_TURN = int(STEPS_PER_REV / 4)

HALF_STEP_SEQ = [ [1, 0, 0, 0], [1, 1, 0, 0], [0, 1, 0, 0], [0, 1, 1, 0], [0, 0, 1, 0], [0, 0, 1, 1], [0, 0, 0, 1], [1, 0, 0, 1] ]

in1 = Stepper(STEPPER_PINS[0]) in2 = Stepper(STEPPER_PINS[1]) in3 = Stepper(STEPPER_PINS[2]) in4 = Stepper(STEPPER_PINS[3]) coil_pins = [in1, in2, in3, in4]

touch_sensor = Button(TOUCH_PIN) print(f"Stepper connected to GPIOs {STEPPER_PINS}. Touch sensor on GPIO {TOUCH_PIN}.")

def rotate_motor(steps, direction): print(f"Starting rotation: {steps} steps, {'CW' if direction == 1 else 'CCW'}") if direction == 1: step_sequence = HALF_STEP_SEQ else: step_sequence = HALF_STEP_SEQ[::-1] for i in range(steps): step = step_sequence[i % 8] # Cycle through the 8 half-steps for pin_index in range(4): if step[pin_index] == 1: coil_pins[pin_index].on() else: coil_pins[pin_index].off()

sleep(STEP_DELAY) for pin in coil_pins: pin.off() print("Rotation complete. Coils de-energized.")

def on_touch(): rotate_motor(QUARTER_TURN, 1)

def on_release(): rotate_motor(QUARTER_TURN, -1)

touch_sensor.when_pressed = on_touch touch_sensor.when_released = on_release

print("Ready. Press the touch sensor to move the stepper motor 90 degrees.")

try: pause()

except KeyboardInterrupt: print("\nProgram stopped by user.")

finally: for pin in coil_pins: pin.close() touch_sensor.close() print("GPIO resources cleaned up.")
