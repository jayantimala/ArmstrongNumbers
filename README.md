# Armstrong Numbers - Efficient Generation Algorithms

## What is that all about?

Armstrong number (aka Narcissistic number) of length N digits is a number which is equal to the sum of its digits each in power of *N*. For example: 153 = 1^3 + 5^3 + 3^3 = 3 + 125 + 27 = 153

More info at [wiki](https://en.wikipedia.org/wiki/Narcissistic_number)

## Why do we need to optimize?

Let's say that we need to generate Armstrong Numbers from 1 to K (of length N digits). There is an obvious bruteforce algorithms that:

1. For each integer *i* from 1 to *K*
2. Divides *i* by digits
3. Calculate power of each digit
4. Sum up those powers
5. If this sum is equal to *i* - add it to the result list
 
Such approach works reasonable time only for int numbers (N<10). For long''s (N<20) it works more than a night, which is not OK.

## Better approach - multi-sets

We may note that for each multi-set of digits, like [1, 1, 2, 4, 5, 7, 7] there is only one sum of powers, which in its turn may either be or be not represented by the digits from set. In the example 1^7 + 1^7 + 2^7 + 4^7 + 5^7 + 7^7 + 7^7 = 1741725, which can be represented by the digits and thus is an Armstrong number.

We may build an algorighm basing on this consideration.

1. For each number length from 1 to N
2. Generate all possible multi-sets of N digits
3. For each multi-set calculate sum of digits^N
4. Check if it''s possible to represent the number we got on step 4 with the digits from the multi-set
5. If so - add the number to the result list

Implementation of the algorithm with optimizations:
* For long: [ArmstrongNumbersLong.java](https://github.com/shamily/ArmstrongNumbers/blob/master/ArmstrongNumbersLong.java)
* For BigInteger: [ArmstrongNumbersBigInteger](https://github.com/shamily/ArmstrongNumbers/blob/master/ArmstrongNumbersBigInteger.java)

## Another approach - Hash

There is another interesting idea of bruteforce approach improvement. 

1. Divide a number for two equal parts. In case of an odd *N* first part will be a bit longer. For example, if *N*=7, the number will be divide like XXXXYYY, where XXXX the first part (4 decimal digits), and YYY the second part with 3 digits.
2. Generate all integers *i* of the second part (in our example there will be integers from 001 to 999).
3. Calculate *p* equal to sum of digits in power of *N*.
4. Add to some hash the following pair {p-i, i}. For example, for *i*=725, *p*=7^7+2^7+5^7=901796. We add pair {901071, 725}.
5. Generate all integers *i* of the first part without leading zeros (in our example there will be integers from 1000 to 9999).
6. Calculate *p* equal to sum of digits in power of *N*.
7. Check if hash has a key of (i\*10^(N/2)-p). For example, *i*=1741, thus *p*=1^7 + 7^7 + 4^7 + 1^7=839929. We look for key (1741000 - 839929) = (901071). OMG! It exists!!!
8. In case that key exists we unite the Armstrong number from two parts and add it to the result list. 1741000 + 725 = 1741725

One addition, is that we cannot store simply (key, value), we need to store multiple values, for example to be able to generate 370 and 371.

## Benchmarking

Let's compare the algorithms performance for different numbers of length *N*. I did the tests with my MacBook Pro.

| Algorithm   | N           | Performance  | Comments |
| ------------- |:-------------:| :-----:| -----|
| Brute Force   | int (N<10)  | seconds | |
| Brute Force   | long (N<20) |   ~200 years | Just wait! |
| Optimized - Hash | int (N<10)  |    50 ms | |
| Optimized - Hash | long (N<20)  |    minutes | Hash consumes quite a lot of memory and if we leave algorithm as is it will throw OutOfMemory |
| Optimized - Multi-set | int (N<10)  |    11 ms | before all the optimizations: 15 ms |
| Optimized - Multi-set | long (N<10)  |    ~550 ms | before optimizations ~ 1s |
| Optimized - Multi-set | BigInteger (N<40)  |    ~ 0.5 hours | All 88 decimal Armstrong numbers   |

Clear win of the multi-set algorithm!
