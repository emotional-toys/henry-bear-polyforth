## What needs to happen

We are stuck with VOL06 in thinking that external GPIO pins are the answer when direct connection via J8-J13 suffices.

For the IDE, use this [repo](https://github.com/volatco/saneForth).

### Task definition

Walk through the code and see if you can operate eyes and mouth by hand over IDE.
 ============
COLLABORATION PROCEDURES
TRANSIENT, UNDER DEVELOPMENT
BETWEEN CHRIS AND GREG
As of 250510 Session call
============================

If we had net access between your XP box and my office, we could do this easily and in real time by mapping a file here into each of our saneFORTH systems.  The file would then be available to both of our polyFORTH systems running on the EVBs, and the connection could easily be secured using IPSec. This is how we oldtimers usually work together; it is as tight as can be.

#### Collaboration set-up

1.  Make minimal file system on a USB flash using  the XP machine.  Directory is D:\evb.  Copy  one of my 4800 block CARTHEUR-back.src files  into it.  Move flash to XP machine if this was  done on another.
2.  On XP machine make projects\henry   Duplicate of DEFAULT directory.
3.  Do all the replacements in file names, shortcut  command line and custom.txt file content for the new directory name.
4.  Verify the shortcut works.  Say BYE.
5.  Change content of custom.txt so that 5 UNIT  aka Projback in comment has the full path to  the new file on the USB flash.
6.  Run aF-3 using same shortcut.
7.  DISKING LOAD  4800 QX  to see differences   between the default application source and the  "backup" as received from me.

#### HENRY HARDWARE DEBUG SETUP

1.  EVB ports A and B connected and properly configured 
2.  Insert jumper J26.
3.  Fire up voltmeter, 10V scale good.
4.  Have DB014 EVB pinouts handy (p.26)
5.  Have DB013 section 5 (p.31) handy for reference on the external IDE.
6.  Have DB001 page 10 handy for F18 register addresses.
7.  Have DB002 2.3 page 10 handy for IO port addressing.

#### DEBUG HENRY HARDWARE
1.  Insert J26 no-boot jumper.
2.  Check paths 0PA 1PA 2PA if you've been changing them
3.  Say BRIDGE LOAD  TALK  to connect to host chip
4.  Say SPAN  to append target chip to host
5.  --- We will be using path 0 which can reach target.
6.  Say 0 10007 HOOK  to connect with node 7 on target,   which controls 18 bit data bus via its UP register
7.  Say SEE  to display stack and RAM in that node
8.  Say 1. LIT  then 2. LIT  then 3. LIT  to push those  numbers onto the node's stack and see display update
    (Steps 7 and 8 are not necessary, just good to get the feel of being actually in that node.)
9.  Notice from DB002 that the parallel bus register is  reached as UP.  It would be nice to read UP at this
    point, however read and write of UP blocks waits for signals from node 10008.  Use DATA instead.
10. To see what should be on the data pins right now,  say HEX 1.5555 IO R!  to set default value including bus direction (out).   0 DATA R@ D.  to display output register.  Note we're still in HEX, don't forget to say DECIMAL  before issuing any IDE functions like HOOK.
11. Say  0. DATA R!  which should drive all D pins to  ground.  Check that all the pins with signals that  originate from the d bus are coming out of the level shifter at ground, and if not buzz back to find the reason.  Assuming that is all grokked and fixed if necessary, proceed: 
12. Say  1. DATA R!  to drive d00 high.  With voltmeter,  start at J30 pin 0.  That's d00 which should be 1.8V
    follow to U13.A3, to U13.B3 which should be high at the 3.3V-ish rail, to motor driver pin BI1A and produce appropriate 5V signals on its A pins per your drawing.  Switch back and forth if desired, storing  1. and 0. into UP.  Notice if any other motor driver input or output pins change when doing this.
13. Repeat 12 using 0. and 2. for values stored to DATA.
14. Say  DECIMAL 0 10009 HOOK and repeat relevant steps  from above starting at 6 and using proper pins for  the d00 and d01 signals.

 ============