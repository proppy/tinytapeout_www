---
hidden: true
title: "42 Base-10 grey counter counts from zero to a trillion"
weight: 43
---

## 42 : Base-10 grey counter counts from zero to a trillion

* Author: Daniel Wisehart
* Description: Change only one output bit per count, but count with decimal digits instead of the usual reverse bit order grey counter.
* [GitHub repository](https://github.com/dwisehart/tt03-submission)
* [Most recent GDS build](https://github.com/dwisehart/tt03-submission/actions/runs/4780044080)
* HDL project
* [Extra docs]()
* Clock: any Hz
* External hardware: 



### How it works

Like a standard grey counter, this counter will only change one bit per time, but the bits are grouped into
decimal digits.  (This also makes for easy bits -> decimal decoding, which is done in the test.py file.)

Each decimal digit uses five bits.  As with all grey counters, you have to scan the counter output fast
enough that either the output value is the same or it has only increased by 1 between scans.  If you scan
too slowly and the value changes by 2 or more, then the grey counter can and will give you bad counts.

As an exampe of how this grey counter works, this is the progression of grey bits when counting from from
decimal 0 to 12:
\begin{tabular}{rrl}
10's  & 1's   & count \\
10001 & 10001 & 0     \\
10001 & 00001 & 1     \\
10001 & 00011 & 2     \\
10001 & 00010 & 3     \\
10001 & 00110 & 4     \\
10001 & 00100 & 5     \\
10001 & 01100 & 6     \\
10001 & 01000 & 7     \\
10001 & 11000 & 8     \\
10001 & 10000 & 9     \\
00001 & 10001 & 10    \\
00001 & 00001 & 11    \\
00001 & 00011 & 12
\end{tabular}

But wait, you say, at decimal value 10, two bits change at once.  This is where your deciphering of the bits
has to be smart.

The bits in the 10's digit change 10x more slowly than the 1's digit.  So when you decipher the combined
value, you look first at the 10's digit.  If the 10's digit has changed, then you can ignore the 1's digit,
because you know the combined value is 10 or 20 or ... 90 or back to 00.  If the 10's digit has not changed,
then you look at both digits to decipher the combined value: 1 or 2 or ... 98 or 99.

Some work has been done to insure that the 10's digit changes before the 1's digit, but if the physical
routes between the outputs of this circuit and the inputs to your scanning circuit make the 10's bits
arrive at your inputs later than the 1's bits, you can and will receive 1's bits that change from 9 ('b10000)
to 0 ('b00000) even though you have not yet seen the 10's bits change.  That can be worked around because you
can deduce that the 10's digit has rolled over, so you update your internal copy of the 10's bits.  It is
probably better to keep the input paths the same length or make the 1's bits use a little longer path than
the 10's bits.

Note that in this implementation, the TinyTapeout PCB hardware--which only has 8 outputs--outputs a
combination of eight bits depending on the input selection.  This is the purpose of the input selection: to
determine which counter values you can see on the outputs.  To see all of the digits at once, you have to run
this in the simulator or with cocotb.  Here are the hardware output selections that are available.  The
selection uses input bits 7 to 2, in that order.

\begin{tabular}{lrrr}
   select [5:0] & output [7:0] &                 &               \\
   000101       & rollover     &  100B{[}5:0{]}  &  10B{[}5:4{]} \\
   000110       & 100B{[}0{]}  &    10B{[}5:0{]} &   1B{[}5:4{]} \\
   000111       &  10B{[}0{]}  &     1B{[}5:0{]} & 100M{[}5:4{]} \\
   001001       &   1B{[}0{]}  &   100M{[}5:0{]} &  10M{[}5:4{]} \\
   001010       & 100M{[}0{]}  &    10M{[}5:0{]} &   1M{[}5:4{]} \\
   001011       &  10M{[}0{]}  &     1M{[}5:0{]} & 100T{[}5:4{]} \\
   010001       &   1M{[}0{]}  &   100T{[}5:0{]} &  10T{[}5:4{]} \\
   010010       & 100T{[}0{]}  &    10T{[}5:0{]} &   1T{[}5:4{]} \\
   010011       &  10T{[}0{]}  &     1T{[}5:0{]} &  100{[}5:4{]} \\
   100001       &   1T{[}0{]}  &    100{[}5:0{]} &   10{[}5:4{]} \\
   100010       &  100{[}0{]}  &     10{[}5:0{]} &    1{[}5:4{]} \\
   default      & 10{[}1:0{]}  &      1{[}5:0{]} & half\_clk
\end{tabular}

Here are the meanings of the abbreviations
\begin{tabular}{rr}
     abbrev &              meaning  \\
       100B &   100 billions digit  \\
        10B &    10 billions digit  \\
         1B &       billions digit  \\
       100M &   100 millions digit  \\
        10M &    10 millions digit  \\
         1M &       millions digit  \\
       100T &  100 thousands digit  \\
        10T &   10 thousands digit  \\
         1T &      thousands digit  \\
        100 &       hundreds digit  \\
         10 &           tens digit  \\
          1 &        singles digit  \\
   rollover &         one trillion  \\
  half\_clk & clock divided by two
\end{tabular}


### How to test

After reset goes low, the counter should increase by one with each rising edge of the clock.
You are encouraged to try different clock rates.  If you load a 60 bit value into the init input before you
assert reset, that bit pattern will be loaded into the grey counter, as shown in test.py.

If you enter `make` from the src directory, a cocotb test will run and report the results.  Inside of test.py
there is a RANGE constant at the top of the file which you can use to tell cocotb how many different values
to check.  Here are some representative test times:

\begin{tabular}{lr}
RANGE =     10,000 &    2 sec \\
RANGE =    100,000 &   19 sec \\
RANGE =  1,000,000 &  189 sec \\
RANGE = 10,000,000 & 2193 sec \\
\end{tabular}


### IO

| # | Input        | Output       |
|---|--------------|--------------|
| 0 | clock  | output[7] |
| 1 | reset  | output[6] |
| 2 | select[5]  | output[5] |
| 3 | select[4]  | output[4] |
| 4 | select[3]  | output[3] |
| 5 | select[2]  | output[2] |
| 6 | select[1]  | output[1] |
| 7 | select[0]  | output[0] |
