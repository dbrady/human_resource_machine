# Human Resource Machine Emulator

Human Resource Machine: http://tomorrowcorporation.com/humanresourcemachine

HRM is a pretty cool way of gamifying a turing machine (but a TM with a few
bytes of RAM). Basically if BrainFuck had 24 bytes of RAM, the concept of
pointers, we'd be there. (Also HRM stats some levels with constants prerecorded
into RAM, which technically is doable through the existing commands but it gets
tricky&sup1;.)

&sup1; You can capture a zero by reading from INBOX, COPYTO a cell, SUB that
cell, then COPYTO the "zero" cell. You can then use a hardcoded series of BUMP+
to create any other constant.

Unfortunately for me, I find fiddling on my iPad very tedious. I would like very
much to be able to save and load from my favorite editor. Even if I can't have
method calls, I at least want to be able to save and load snippets like "LOAD A
STRING FROM INBOX TO FLOOR STARTING AT TILE n"

## Commands

### **INBOX**

Get a box from INBOX.

* Discard anything in your hands.
* Walk to INBOX.
* If INBOX is empty: PROGRAM ENDS
* Move the topmost box from INBOX and hold it in your hands.

### **OUTBOX**

Put your box in the OUTBOX. Empties your hands.

* Walk to outbox.
* If your hands are empty: ERROR "Empty value! You can't OUTBOX with empty hands!"
* Move the box in your hands to the OUTBOX.
* If the box was wrong: ERROR "Bad outbox! Management expected N, but you
  outboxed M."
* If the outbox is now too big (program should've stopped already): ERROR "TOO
  MANY THINGS IN THE OUTBOX! Your outbox was correct up intil now, but
  management expected only N items, not M!"

### **copyfrom ADDR**

Copy box from the floor to your hands. Overwrites anything you're carrying.

* Walk to ADDR.
* If tile is empty, ERROR: "Empty value! You can't COPYFROM with an empty tile
  on the floor! Try writing something to that tile first."
* Pick up a COPY of the box on the floor.

### **copyto ADDR**

Copy box from your hands to the floor. Does not empty your hands.

* Walk to ADDR.
* Put a copy the box in your hands on the floor.

### **add ADDR**

Add the box on the floor to the box in your hands.

* Walk to ADDR.
* If tile is empty, ERROR
* If you are holding a letter or tile contains a letter, ERROR "You can't ADD with a letter! What would
  that even mean?!"
* * Add the box on the floor to the box in your hands.

Example:
* Before:
  * Tile 3 contains "6"
  * You are holding "3"
* ADD 3
* After:
  * Tile 3 contains "6"
  * You are holding "9"


### **sub ADDR**

Add the box on the floor to the box in your hands.

* Walk to ADDR.
* If tile is empty, ERROR
* If tile contains a letter and you are holding a number OR tile contains a
  number and you are holding a letter, ERROR "You can't SUB with mixed operands!
  SUB'ing between one letter and one number is invalid. Only nice respectable
  pairs of two letters or two numbers is allowed."
* Subtract the box on the floor from the box in your hands.

Example 1:
* Before:
  * Tile 3 contains "6"
  * You are holding "4"
* SUB 3
* After:
  * Tile 3 contains "6"
  * You are holding "2"

Exmaple 2: Works with letters, too!

Example 1:
* Before:
  * Tile 3 contains "C"
  * You are holding "E"
* SUB 3
* After:
  * Tile 3 contains "C"
  * You are holding "-2"

### **bump+ ADDR**

Increment the value in the box on the floor, then copy the box to your hands.

* Walk to ADDR.
* If tile is empty, ERROR "Empty value! You can't BUMP+ with an empty tile or
  the floor! Try writing something to that tile first."
* If tile contains a letter, ERROR "You can't BUMP+ with a letter! What would
  that even mean?!"
* Increment the value in the box on the floor.
* Copy the box to your hands.

### **bump- ADDR**

Decrement the value in the box on the floor, then copy the box to your hands.

* Walk to ADDR.
* If tile is empty, ERROR
* If tile is empty, ERROR "Empty value! You can't BUMP- with an empty tile or
  the floor! Try writing something to that tile first."
* If tile contains a letter, ERROR "You can't BUMP- with a letter! What would
  that even mean?!"
* Decrement the value in the box on the floor.
* Copy the box to your hands.

### **jumptarget TARGET**

Not really a command in the game, but jumps all go to a specific target, which
take up a spot on the program tape but do not have line numbers.

### **jump TARGET**

Move the instruction pointer to TARGET.

Note: Technically jumps go to "jump targets" which are positions between
instructions, labels, and other jump targets. E.g. you can have a program like

    LABEL Copy INBOX to OUTBOX
    jumptarget 1
    INBOX
    OUTBOX
    jump 1

### **jump if zero LINE**

If you have a zero in your hands, move the instruction pointer to
TARGET. Otherwise, proceed to the next instruction as normal.

* If your hands are empty, ERROR "Empty value! You can't JUMP IF ZERO with empty
  hands!"

Fun fact: if you are holding a letter, JUMP IF ZERO will fall through. It won't
jump, but it also won't raise an error. Why? Because you are not holding a zero!

### **jump if negative LINE**

If you have a negative number in your hands, move the instruction pointer to
TARGET. Otherwise, proceed to the next instruction as normal.

* If your hands are empty, ERROR "Empty value! You can't JUMP IF NEGATIVE with
  empty hands!"

Fun fact: if you are holding a letter, JUMP IF NEGATIVE will fall through. It
won't jump, but it also won't raise an error. Why? Because you are not holding a
negative number!

### **LABEL**

A comment. Does not count as an instruction.

### **name TILE NAME**

Before starting a program, tiles can be give specific labels. In the HRM game,
you draw labels on tiles while writing your program. Once the program is
running, they cannot be changed. To do this in text, set them up before the
first instruction. (Arguably this could also go along with the setup of the
INBOX and the OUTBOX.)

The HMR does not allow you to use a named tile until you have named it, and
changing its name or removing it will update any commands that use that
name. That's part of the "IDE" portion of the game and doesn't really need
supporting. For this emulator, however, we'll need to catch any errors where
players try to use a name that doesn't exist.

## NON-ERROR DETAILS AND NOTES

### **Line Numbers**

Each instruction has a line number. LABELs do not.

### **ADDR**

The address of a tile on the floor, OR the location of a tile that contains a
location of another tile. E.g. if tile 16 contains a 4, then "bump+ 4" and
"bump+ [16]" will both increment tile 4. So 4 and [16] both mean the Address of
tile 4.

### **ERROR**

* Display error message
* Program aborts

### **PROGRAM WINS**
* Display Win message
* Show

### **EXECUTION STOP**
* Execution stops (Error or success message has already been displayed)

### **PROGRAM ENDS**
* ERROR "Not enough stuff in the OUTBOX! Management expected a total of N items,
not M!" if outbox too small
* PROGRAM WINS if outbox correct

# Syntax

* HMR programs end with ".hmr". Maybe. Or .txt. Or .md. I dunno. Actually, I do
  know: we'll end them with ".hmr" but they'll secretly be ruby programs. I
  don't want to write a parser for the emulator; I think I'd rather just write
  support for it all in ruby, e.g. `def add(addr)`. The hmr "emulator" will
  simply do a require "hmr" and then run the program, displaying the program and
  then showing the workspace as the little Instruction Follower executes the
  program.

* Text execution simply journals the execution as it goes along, without trying
  to display the work room or the instruction pointer. E.g. here's a simple
  program, and its output:

```
    $ hmr adder.hmr
    # Program start:
    INBOX: 3 14 7 5
    FLOOR:
    0:
    1:
    2:
    3:
    Program has 6 instructions.
    > JUMPTARGET start
    Jump target "start" is here.
    > INBOX
    Picked up 3.
    > COPYTO 0
    wrote 3 to tile 0.
    > INBOX
    Picked up 14.
    > ADD 0
    Adding from 0. 3 + 14 = 17.
    > OUTBOX
    Put 17 in outbox.
    > JUMP start
    Arrived at jump target "start".
    > INBOX
    Picked up 7.
    > COPYTO 0
    wrote 7 to tile 0.
    > INBOX
    Picked up 5.
    > ADD 0
    Adding from 0. 7 + 5 = 12.
    > OUTBOX
    Put 12 in outbox.
    > JUMP start
    Arrived at jump target "start".
    > INBOX
    Nothing to pick up!
    Outbox looks good!
    WIN!
    Program executed in 12 operations.
```
