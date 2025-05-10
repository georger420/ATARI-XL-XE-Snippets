## How to use Turbo 2000 Basoic Saver

If you have a tested program in your computer (running TOS 4.1), switch to the TOS menu using the DOS command and load the BASAVTOS program using the L function. Press Y to run it. The program's title will appear on the screen, along with the current LOMEM value and the program length (Length). Now you need to enter a new LOMEM address, using hexadecimal format.

If the program needs to run under an operating system (cassette or disk-based), it's recommended to set LOMEM to $2000. However, it can be higher (if the program's size allows it) or lower ($0600) if needed. If you simply press RETURN without entering an address, the program will use the existing LOMEM value.

📁 Note: The program checks if the entered address is a number, but it doesn’t verify if it's logically correct.

Next, enter the name of the program to be saved (up to 10 characters). It's common practice to use only uppercase letters and numbers, with a dot to separate the file extension. Other characters should not be used, including inverted ones. If you just press RETURN instead of entering a name, the program will exit and return to the TOS menu without saving the BASIC program.

After entering the name, the computer beeps twice, and when you press any key, it saves the BASIC program. If you want to save it again, press Y.

It is all.