## ideal in forth

OK, have a model worth looking at which gets the following results:

4821 LOAD
STEP in 4826 hides 634 ok
RUN
0 : 1 Null->R2 0 Frustrated
1 : 1 Null->R2 1 SelfSatisfied
2 : 1 Null->R2 2 Bored
3 : 2 E1->R1 0 Frustrated
4 : 2 E1->R1 1 SelfSatisfied
5 : 2 E1->R1 2 Bored
6 : 3 E2->R2 0 Frustrated
7 : 3 E2->R2 1 SelfSatisfied
8 : 3 E2->R2 2 Bored
9 : 2 E1->R1 1 SelfSatisfied
10 : 2 E1->R1 2 Bored
11 : 3 E2->R2 1 SelfSatisfied
12 : 3 E2->R2 2 Bored
13 : 2 E1->R1 1 SelfSatisfied
14 : 2 E1->R1 2 Bored
15 : 3 E2->R2 1 SelfSatisfied
16 : 3 E2->R2 2 Bored
17 : 2 E1->R1 1 SelfSatisfied
18 : 2 E1->R1 2 Bored
19 : 3 E2->R2 1 SelfSatisfied ok

One correction, comments on OTHEREXP in block 4825.  First version returns *lowest" ordinal not *highest*.  And once again, off to bed for me 😞