# Pin-the-tail-on-the-moon-maggot
I have ridden the mighty moon worm! 

![image](https://github.com/user-attachments/assets/4f8dc1e8-83fe-4b1b-ac82-6b588faed1ab)

![image](https://github.com/user-attachments/assets/1da215c2-d7fb-4136-acff-ad82446f8f94)

```
:I `Init: Creating array` 
   [ 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ] m !
   `Init: Complete` /N
;

:S `Set: Starting` /N
   p ! q ! r ! s !      // Store coordinates and value
   `Set: Coordinates x:` p . ` y:` q . ` z:` r . ` val:` s . /N
   p 16 * q 4 * + r + t !  // Calculate offset
   `Set: Offset calculated:` t . /N
   s m t ? !            // Store value in array
   `Set: Value stored` /N
;

:G `Get: Starting` /N
   d ! e ! f !          // Store coordinates
   `Get: Reading x:` d . ` y:` e . ` z:` f . /N
   d 16 * e 4 * + f + g !  // Calculate offset
   `Get: Offset calculated:` g . /N
   `Get: Value is:` m g ? . /N
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

# ASM code

```
; 3D Space Mapping Robot Control
; Memory layout:
; 4x4x4 array starts at ARRAY_BASE
; Each cell is 1 byte

        ORG     8000h           ; Program start

ARRAY_BASE:      EQU 9000h      ; Array location
SERVO_X:         EQU 80h        ; Servo X port
SERVO_Y:         EQU 81h        ; Servo Y port
SERVO_Z:         EQU 82h        ; Servo Z port
WHISKER_PORT:    EQU 83h        ; Whisker input port

; Initialize array
init:   LD      HL,ARRAY_BASE   ; Point to array start
        LD      B,64            ; 64 bytes to clear
        XOR     A               ; A = 0
init_loop:
        LD      (HL),A          ; Clear byte
        INC     HL              ; Next position
        DJNZ    init_loop       ; Repeat 64 times
        RET

; Calculate array offset
; Input: B=X, C=Y, D=Z
; Output: HL=offset
calc_offset:
        LD      A,D             ; Get Z
        LD      H,0
        LD      L,A
        ADD     HL,HL           ; *2
        ADD     HL,HL           ; *4
        ADD     HL,HL           ; *8
        ADD     HL,HL           ; *16 (Z*16)
        
        LD      A,C             ; Get Y
        LD      E,A
        LD      D,0
        ADD     HL,DE
        ADD     HL,HL           ; Y*4
        ADD     HL,HL
        
        LD      A,B             ; Get X
        LD      E,A
        LD      D,0
        ADD     HL,DE           ; Add X
        
        LD      DE,ARRAY_BASE
        ADD     HL,DE           ; Add base address
        RET

; Set value in array
; Input: B=X, C=Y, D=Z, E=value
set_value:
        PUSH    DE
        CALL    calc_offset
        POP     DE
        LD      (HL),E
        RET

; Get value from array
; Input: B=X, C=Y, D=Z
; Output: A=value
get_value:
        CALL    calc_offset
        LD      A,(HL)
        RET

; Move servos
; Input: B=X, C=Y, D=Z
move_servos:
        ; X servo
        LD      A,B
        CALL    scale_pos       ; Scale 0-3 to 0-180
        OUT     (SERVO_X),A
        
        ; Y servo
        LD      A,C
        CALL    scale_pos
        OUT     (SERVO_Y),A
        
        ; Z servo
        LD      A,D
        CALL    scale_pos
        OUT     (SERVO_Z),A
        
        CALL    delay
        RET

; Scale position (0-3) to servo range (0-180)
scale_pos:
        LD      H,45            ; Multiply by 45
        CALL    multiply
        RET

; Multiply A by H
multiply:
        LD      B,A             ; Save multiplier
        LD      C,0             ; Initialize result
        LD      A,H             ; Get multiplicand
mult_loop:
        OR      A               ; Check if done
        RET     Z
        ADD     A,C             ; Add to result
        LD      C,A             ; Save result
        DJNZ    mult_loop       ; Decrement counter
        RET

; Check whiskers
check_whiskers:
        IN      A,(WHISKER_PORT)
        AND     1               ; Mask bit 0
        RET

; Scan single position
scan_pos:
        PUSH    BC
        PUSH    DE
        CALL    move_servos
        CALL    delay
        CALL    check_whiskers
        OR      A
        JR      Z,scan_end
        LD      E,1             ; Mark as occupied
        CALL    set_value
scan_end:
        POP     DE
        POP     BC
        RET

; Delay routine
delay:  LD      BC,5000
delay_loop:
        DEC     BC
        LD      A,B
        OR      C
        JR      NZ,delay_loop
        RET

; Main scan loop
scan_all:
        LD      D,0             ; Z=0
z_loop: LD      C,0             ; Y=0
y_loop: LD      B,0             ; X=0
x_loop: CALL    scan_pos
        INC     B
        LD      A,B
        CP      4
        JR      NZ,x_loop
        INC     C
        LD      A,C
        CP      4
        JR      NZ,y_loop
        INC     D
        LD      A,D
        CP      4
        JR      NZ,z_loop
        RET

; Main program
main:   CALL    init
main_loop:
        CALL    scan_all
        CALL    delay
        JR      main_loop
```

Key differences from MINT:

Direct memory access instead of array operators
Hardware I/O using IN/OUT instructions
Register-based calculations instead of stack operations
Nested loops using labels instead of counters
Binary arithmetic instead of decimal

