# Pin-the-tail-on-the-moon-maggot
I have ridden the mighty moon worm! 

![image](https://github.com/user-attachments/assets/4f8dc1e8-83fe-4b1b-ac82-6b588faed1ab)

![image](https://github.com/user-attachments/assets/1da215c2-d7fb-4136-acff-ad82446f8f94)

```
:I `Init: Creating array` 
   [ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] m b!
   `Init: Complete` /N
;

:S `Set: Starting` /N
   p b! q b! r b! s b!
   `Set: Coordinates x:` p . ` y:` q . ` z:` r . ` val:` s . /N
   p 16 * q 4 * + r + t b! 
   `Set: Offset calculated:` t . /N
   s m t ? u b!  // Use ? for array access
   `Set: Value stored` /N
;

:G `Get: Starting` /N
   d b! e b! f b!
   `Get: Reading x:` d . ` y:` e . ` z:` f . /N
   d 16 * e 4 * + f + g b!
   `Get: Offset calculated:` g . /N
   `Get: Value is:` m g ? . /N  // Use ? for array access
;

:M `Move: Starting servo move` /N
   h b! i b! j b!
   `Move: Raw coords x:` h . ` y:` i . ` z:` j . /N
   h 45 * k b!
   i 45 * l b!
   j 45 * m b!
   `Move: Servo positions:` /N
   `X servo:` k . /N
   `Y servo:` l . /N 
   `Z servo:` m . /N
   k #80 /O
   l #81 /O
   m #82 /O
   `Move: Servos updated, delaying` /N
   100()
   `Move: Complete` /N
;

:W `Whisker: Reading port` /N
   #83 /I 1 & n b! n .
   `Whisker: State is:` n . /N
;

:P `Pos Scan: Starting` /N
   h b! i b! j b!
   `Pos Scan: Moving to x:` h . ` y:` i . ` z:` j . /N
   h i j M
   `Pos Scan: Moved, waiting` /N
   100()
   `Pos Scan: Checking whisker` /N
   W (
      `Pos Scan: Contact! Marking position` /N
      1 h i j S
   ) /E (
      `Pos Scan: No contact` /N
   )
;

:F `Full Scan: Starting` /N
   4( /i o b!
      `Full Scan: Layer ` o . /N
      4( /i p b!
         `Full Scan: Row ` p . /N
         4( /i q b!
            `Full Scan: Column ` q . /N
            q p o P
         )
      )
   )
   `Full Scan: Complete` /N
;

:L `Loop: Starting` /N
   I
   `Loop: Initialized` /N
   /U(
      `Loop: Starting scan` /N
      F
      `Loop: Scan complete, delaying` /N
      1000()
   )
;
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
