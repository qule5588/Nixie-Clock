# Nixie-Clock brown(1), blue(3), red(8), orange(9), purple(11)
Code for nixie tube clock in raspberry pi

import RPi.GPIO as GPIO
import time
import datetime

PIN_DATA = 23
PIN_LATCH = 24
PIN_CLK = 25

class Nixie:
    def __init__(self, pin_data, pin_latch, pin_clk, digits):
        self.pin_data = pin_data
        self.pin_latch = pin_latch
        self.pin_clk = pin_clk
        self.digits = digits

        GPIO.setmode(GPIO.BCM)

        # Setup the GPIO pins as outputs
        GPIO.setup(self.pin_data, GPIO.OUT)
        GPIO.setup(self.pin_latch, GPIO.OUT)
        GPIO.setup(self.pin_clk, GPIO.OUT)

        # Set the initial state of our GPIO pins to 0
        GPIO.output(self.pin_data, False)
        GPIO.output(self.pin_latch, False)
        GPIO.output(self.pin_clk, False)

    def delay(self):
        # We'll use a 10ms delay for our clock
        time.sleep(1.000)

    def transfer_latch(self):
        # Trigger the latch pin from 0->1. This causes the value that we've
        # been shifting into the register to be copied to the output.
        GPIO.output(self.pin_latch, True)
        self.delay()
        GPIO.output(self.pin_latch, False)
        self.delay()

    def tick_clock(self):
        # Tick the clock pin. This will cause the register to shift its
        # internal value left one position and the copy the state of the DATA
        # pin into the lowest bit.
        GPIO.output(self.pin_clk, True)
        self.delay()
        GPIO.output(self.pin_clk, False)
        self.delay()

    def shift_bit(self, value):
        # Shift one bit into the register.
        GPIO.output(self.pin_data, value)
        self.tick_clock()

    def shift_digit(self, value):
        # Shift a 4-bit BCD-encoded value into the register, MSB-first.
        self.shift_bit(value&0x08)
        value = value << 1
        self.shift_bit(value&0x08)
        value = value << 1
        self.shift_bit(value&0x08)
        value = value << 1
        self.shift_bit(value&0x08)

    def set_value(self, value):
        # Shift a decimal value into the register

        str = "%0*d" % (self.digits, value)

        for digit in str:
            self.shift_digit(int(digit))
            value = value * 10

        self.transfer_latch()

def main():
    try:
        nixie = Nixie(PIN_DATA, PIN_LATCH, PIN_CLK, 1)

        # Uncomment for a simple test pattern
        #nixie.set_value(4)

        # Repeatedly get the current time of day and display it on the tubes.
        # (the time retrieved will be in UTC; you'll want to adjust for your
        # time zone)
        while True:
            dt = datetime.datetime.now() #replace with temperature
            nixie.set_value(dt.hour*100 + dt.minute)

    finally:
        # Cleanup GPIO on exit. Otherwise, you'll get a warning next time toy
        # configure the pins.
        GPIO.cleanup()

if __name__ == "__main__":
    main()
