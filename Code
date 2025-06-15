#include <lpc214x.h>
#include <stdio.h>

#define RS  (1 << 2)    // RS connected to P0.2
#define EN  (1 << 3)    // EN connected to P0.3
#define D4  (1 << 4)    // D4 connected to P0.4
#define D5  (1 << 5)    // D5 connected to P0.5
#define D6  (1 << 6)    // D6 connected to P0.6
#define D7  (1 << 7)    // D7 connected to P0.7

#define IR_PIN1 (1 << 9)   // IR Sensor 1 connected to P0.9 (Increment)
#define IR_PIN2 (1 << 10)  // IR Sensor 2 connected to P0.10 (Decrement)

void delay(unsigned int count) {
    unsigned int i;
    for (i = 0; i < count; i++);
}

void delay_ms(unsigned int ms) {
    // Assuming 12 MHz clock with default PLL settings for 60 MHz system clock
    unsigned int delay_count = 60000 * ms;  // Roughly 1ms delay loop
    delay(delay_count);
}

void lcd_send_nibble(unsigned char nibble) {
    IO0PIN &= ~(D4 | D5 | D6 | D7);  // Clear data pins
    if (nibble & 0x01) IO0SET |= D4;  // Set data pins
    if (nibble & 0x02) IO0SET |= D5;
    if (nibble & 0x04) IO0SET |= D6;
    if (nibble & 0x08) IO0SET |= D7;

    IO0SET = EN;  // Enable high
    delay(10000);   // Adjusted delay for LCD command execution
    IO0CLR = EN;  // Enable low
    delay(10000);   // Ensure adequate time before the next command
}

void lcd_cmd(unsigned char cmd) {
    IO0CLR = RS;  // Command mode
    lcd_send_nibble(cmd >> 4);  // Send high nibble
    lcd_send_nibble(cmd & 0x0F);  // Send low nibble
    delay(10000);   // Ensure adequate delay for command processing
}

void lcd_data(unsigned char data) {
    IO0SET = RS;  // Data mode
    lcd_send_nibble(data >> 4);  // Send high nibble
    lcd_send_nibble(data & 0x0F);  // Send low nibble
    delay(10000);   // Ensure adequate delay for data processing
}

void lcd_init() {
    IO0DIR |= (RS | EN | D4 | D5 | D6 | D7);  // Set pins as output
    delay(20000);  // Initial delay for LCD to power on
    
    lcd_cmd(0x02);  // Initialize in 4-bit mode
    delay(20000);
    lcd_cmd(0x28);  // 4-bit mode, 2 lines, 5x7 font
    delay(20000);
    lcd_cmd(0x0C);  // Display on, cursor off
    delay(20000);
    lcd_cmd(0x06);  // Increment cursor
    delay(20000);
    lcd_cmd(0x01);  // Clear display initially
    delay(20000);
}

void lcd_string(const char *str) {
    while (*str) {
        lcd_data(*str++);
    }
}

unsigned int read_ir_sensor1() {
    return !(IO0PIN & IR_PIN1);  // Return 1 if IR sensor 1 is active (object detected)
}

unsigned int read_ir_sensor2() {
    return !(IO0PIN & IR_PIN2);  // Return 1 if IR sensor 2 is active (object detected)
}

void display_count(int count) {
    char buffer[16]; // Buffer to hold the display string

    lcd_cmd(0x80);  // Move cursor to the first line
    sprintf(buffer, "Count: %d  ", count);  // Format and display count
    lcd_string(buffer);
}

void update_display(int count) {
    static int last_count = -1;

    // Only update the display if there is a change in count
    if (count != last_count) {
        display_count(count);
        last_count = count;
    }
}

void main() {
    unsigned int last_state1 = 0, last_state2 = 0;  // Track the last states of the IR sensors
    unsigned int current_state1, current_state2;
    int count = 0;  // Initialize the counter
    int block_sensor1 = 0;  // Flag to block sensor 1 for 100ms
    int block_sensor2 = 0;  // Flag to block sensor 2 for a specific time (2 seconds)

    lcd_init();  // Initialize LCD

    // Configure IR sensor pins as input
    IO0DIR &= ~(IR_PIN1 | IR_PIN2);  // Set P0.9 and P0.10 as input

    // Initial display message
    update_display(count);  // Display initial count

    while (1) {
        current_state1 = read_ir_sensor1();  // Read the current state of IR sensor 1
        current_state2 = read_ir_sensor2();  // Read the current state of IR sensor 2

        // If Sensor 1 is triggered and Sensor 2 is not triggered yet
        if (!block_sensor2 && current_state1 && !current_state2 && last_state1 != current_state1) {
            count++;  // Increment count
            update_display(count);  // Update LCD
            block_sensor2 = 1;  // Block sensor 2 for 2 seconds
            delay_ms(2000);  // 2-second delay
            block_sensor2 = 0;  // Unblock sensor 2 after the delay
        }

        // If Sensor 2 is triggered and Sensor 1 is not triggered yet
        if (!block_sensor1 && current_state2 && !current_state1 && last_state2 != current_state2) {
            if (count > 0) {  // Prevent count from going below zero
                count--;  // Decrement count
                update_display(count);  // Update LCD
                block_sensor1 = 1;  // Block sensor 1 for 100ms
                delay_ms(100);  // 100ms delay
                block_sensor1 = 0;  // Unblock sensor 1 after the delay
            }
        }

        // Update the last states for the next loop iteration
        last_state1 = current_state1;
        last_state2 = current_state2;

        // Maintain a stable display and reduce flickering
        delay(100000);  // Adjusted delay for smoother operation
    }
}
