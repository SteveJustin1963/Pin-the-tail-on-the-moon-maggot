# Pin-the-tail-on-the-moon-maggot
I have ridden the mighty moon worm! 

![image](https://github.com/user-attachments/assets/4f8dc1e8-83fe-4b1b-ac82-6b588faed1ab)

![image](https://github.com/user-attachments/assets/1da215c2-d7fb-4136-acff-ad82446f8f94)

```
// Initialize 4x4x4 array in memory (64 bytes total for 4 layers)
:I 64 /A m! 
   64( 0 m /i /? /! )  // Fill array with zeros
;

// Set value in 3D array: value x y z -- 
:S z! y! x! v!
   z 16 * y 4 * + x + // Calculate offset
   v $ m $ /? /!      // Store value at offset
;

// Get value from 3D array: x y z -- value
:G z! y! x!
   z 16 * y 4 * + x + // Calculate offset
   m $ /? .           // Get and print value
;

// Move servos to position: x y z --
:M z! y! x!
   // Map 0-3 to servo ranges (0-180)
   x 45 * s1!         // X servo position
   y 45 * s2!         // Y servo position
   z 45 * s3!         // Z servo position
   
   // Output to servo ports
   s1 #80 /O          // Servo 1 control
   s2 #81 /O          // Servo 2 control
   s3 #82 /O          // Servo 3 control
   100()              // Delay for movement
;

// Check whiskers: -- bool
:W #83 /I 1 & .      // Read whisker port and mask bit 0
;

// Scan single position: x y z --
:P z! y! x!
   x y z M            // Move to position
   100()              // Wait for movement
   W (                // If whisker contact
      1 x y z S       // Mark position as occupied
   )
;

// Full scan sequence
:F 4( i z!           // For each z layer
     4( i y!         // For each y row
        4( i x!      // For each x column
           x y z P   // Scan position
        )
     )
  )
;

// Main loop
:L I                 // Initialize array
   /U(               // Infinite loop
      F              // Do full scan
      1000()         // Wait 5 seconds
   )
;

// To run: L
```


The MINT code above implements a 3D space mapping system with the following functions:

- `I`: Initializes a 64-byte array (4x4x4 = 64 positions) to zero
- `S`: Sets a value in the 3D array at specified coordinates
- `G`: Gets and prints a value from the 3D array at specified coordinates
- `M`: Moves servos to mapped positions (0-3 mapped to 0-180 degrees)
- `W`: Checks whisker contact sensors
- `P`: Scans a single position and updates map if contact detected
- `F`: Performs full scan of all positions
- `L`: Main loop that initializes and continuously scans

Key adaptations from the original code:

1. Uses RPN notation as required by MINT
2. Uses single-letter variables (m for map array, s1-s3 for servo positions)
3. Implements 3D array using linear memory with offset calculations
4. Uses MINT's port I/O for servo control and whisker input
5. Uses MINT's delay mechanism with `()` for timing
6. Leverages MINT's stack operations for parameter passing

To run the program, simply enter `L` at the MINT prompt. The robot will initialize the map and continuously scan the space, updating the map when contact is detected.

Let me know if you'd like me to explain any part in more detail or make modifications to the code.
