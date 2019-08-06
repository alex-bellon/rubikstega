# Scrambled: Rubik's Cube based steganography (from UTCTF 19)

*Disclaimer: I wrote this problem for UT CTF, but I did not come up with the idea (I’m not that smart). This entire system was proposed in the paper ‘Rubikstega: A Novel Noiseless Steganography Method in Rubik’s Cube’, and I just implemented it.*

## Prompt
```
B2 R U F' R' L' B B2 L F D D' R' F2 D' R R D2 B' L R

L' L B F2 R2 F2 R' L F' B' R D' D' F U2 B' U U D' U2 F'

L F' F2 R B R R F2 F' R2 D F' U L U' U' U F D F2 U R U' F U B2 B U2 D B F2 D2 L2 L2 B' F' D' L2 D U2 U2 D2 U B' F D R2 U2 R' B' F2 D' D B' U B' D B' F' U' R U U' L' L' U2 F2 R R F L2 B2 L2 B B' D R R' U L

Have fun!
```

## Solution
Since I made the problem, I'm going to cheat a little and use my God-like problem-writer powers to determine that the problem has to do with Rubikstega (for those not so clairvoyantly inclined, 'Rubikstega' was released as a hint later on in the CTF). Rubikstega is a steganography system that used Rubik's cube scramble notation to encode the data. There are 18 notations for Rubik’s Cube scrambles, but in Rubikstega they’re grouped into 9 pairs. There’s a default encoding table that maps the numbers 0-8 to a pair of notations, and one of the notations from the pair is randomly chosen to represent that number. To encode a message, you first generate a permutation of the default encoding table, with 0-8 now mapping to different notation pairs. This is encoded in the permutation header along with some random padding. To start encoding the message, you convert each character into its binary representation, then concatenate all of them into one large binary string. Next you get convert that long binary string to base 9, and use the permuted encoding table to convert the base 9 digits to scramble notation. Once the message is encoded, you make the length header by combining the length of the encoded message with more random padding. The final message is the permutation header, length header and finally the actual message. My explanation glossed over quite a few details, so if you're interested in the specifics you should check the paper. Now, in order to decode, you just do those steps in reverse:

### Step 0: Write some helper methods
```python
import math, random
nums = {0:('L', 'F'), 1:('R', 'B'), 2:('U', 'L2'), 3:('D', 'R2'), 4:('F2', 'U2'), 5:('B2', 'D2'), 6:('L\'', 'F\''), 7:('R\'', 'U\''), 8:('B\'', 'D\'')}
moves = {'L':0, 'F':0, 'R':1, 'B':1, 'U':2, 'L2':2, 'D':3, 'R2':3, 'F2':4, 'U2':4, 'B2':5, 'D2':5, 'L\'':6, 'F\'':6, 'R\'':7, 'U\'':7, 'B\'':8, 'D\'':8}
shuffleNums = dict()
# Convert a base 9 number to a string
def nineToStr(nine):
  return decToStr(int(str(nine), 9))
# Convert a decimal number to a string
def decToStr(dec):
  return bytes.fromhex(hex(dec).replace('L', '')[2:]).decode('utf-8')
# Convert scramble notation to base 9
def scrambleToNine(scramble, dict):
  if dict: result = ''.join(str(shuffleNums[moves[move]]) for move in scramble.split())
  else: result = ''.join(str(moves[move]) for move in scramble.split())
  return result
```
### Step 1: Decode the permutation header
```python
def decodePerm(head):
  decoded = str(int(str(scrambleToNine(head, 0)), 9))
  shuffle = list(decoded[int(decoded[0]) + 1 : int(decoded[0]) + 1 + 9])
  for key in shuffle: shuffleNums[int(key)] = shuffle.index(key)
```

### Step 2: Decode the length information
```python
def decodeLength(head):
  decoded = str(int(str(scrambleToNine(head, 1)), 9))
  return decoded[int(decoded[0]) + 2 : int(decoded[0]) + 2 + int(decoded[1])]
```

### Step 3: Decode the message!
```python
def decode(cipher):
  scrambles = cipher.split(',')
  decodePerm(scrambles[0])
  encoded = ''.join(scrambles[2:])
  decoded = nineToStr(str(scrambleToNine(encoded, 1))[:int(decodeLength(scrambles[1]))])
  return decoded
```
Add in a main method to get input and call `decode()`, pass in the 3 scrambles from the prompt, and we get the decoded message: 
`utflag{my_bra1n_1s_scrambl3d}`!
