A	Atomicity
C	Consistency
I	 Isolation
D  Durability

C	Consistency
A	Availability
P	Partition Tolerance

BA	Basically Available
S	 Soft State
E	 Eventually Consistent

TC		Transaction Coordinator
TM		Transaction Manager
RM		Resource Manager

XA   -->  X/Open DTP, Distributed Transaction Processing
AT	 -->  undo-log


|     |      |     |      |     |      |
| --- | ---- | --- | ---- | --- | ---- |
| 1   | 0001 | 7   | 0111 | 3   | 0011 |
| 2   | 0010 | b   | 1011 | c   | 1100 |
| 4   | 0100 | d   | 1101 |     |      |
| 8   | 1000 | e   | 1110 |     |      |
|     |      |     |      |     |      |
| 5   | 0101 |     |      | 6   | 0110 |
| a   | 1010 |     |      | 9   | 1001 |




