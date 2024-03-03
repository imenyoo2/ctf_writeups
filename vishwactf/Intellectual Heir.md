

# description

```
# my secret to hide the combination of my safe in fornt of all without anyone getting a clue what it is ;)

#some boring python function for conversion nothing new
def str_to_ass(input_string):
    ass_values = []
    for char in input_string:
        ass_values.append(str(ord(char)))
    ass_str = ''.join(ass_values)
    return ass_str

input_string = input("Enter the Combination: ")
result = str_to_ass(input_string)
msg = int(result)

#not that easy, you figure out yourself what the freck is a & z
a = 
z = 

f = (? * ?) #cant remember what goes in the question mark
e = #what is usually used

#ohh yaa!! now you cant figure out $h!t
encrypted = pow(msg, e, f)
print(str(encrypted))

#bamm!! protection for primes
number = 
bin = bin(number)[2:]

#bamm!! bamm!! double protection for primes
bin_arr = np.array(list(bin), dtype=int)
result = np.sin(bin_arr)
result = np.cos(bin_arr)
np.savetxt("file1", result)
np.savetxt("file2", result)
```

reading the obove we see that the RSA algo is used here (`pow(msg, e, f)`)

# solution

first we see that they took `a` and `z` (p and q), convert them to binary, and then store very bit in `file1` and `file2` after applying eather `cos` or `sin` operation.

since we know the result of `cos(0)` and `cos(1)` (and the same for `sin`), we can recover `a` and `z`

```py
from sympy import *
import math
from tqdm import tqdm
from sove_rsa import solve_rsa
from recover import recover_init_flag

cos = 0

with open("file1.txt", "r") as f:
	lines = f.readlines()[::-1]
	for i, line in enumerate(lines):
		if line[0] == '5':
			cos |= (1 << i)

print(cos)

sin = 0

with open("file2.txt", "r") as f:
	lines = f.readlines()[::-1]
	for i, line in enumerate(lines):
		if line[0] == '8':
			sin |= (1 << i)

print(sin)
```

this script should return 2 prime numbers

after that we only need `e` to be able to decrypte the flag, looking back at the file we're given, they said this `e = #what is usually used`, after searching we find that [65537](https://en.wikipedia.org/wiki/65,537) is what mostly used in RSA, so what left is just to decrypt

```py
def solve_rsa(p, q, e, n, c):
    phi = (p - 1) * (q - 1)
    d = pow(e, -1, phi)  # Extended Euclidean

    m = pow(c, d, n)
    int_m = int(m)
    return (int_m)
```

we apply this to reverse the RSA operation, then we use the function bellow to reverse the `str_to_ass` function

```py
# assuming only 2 digit characters are used
def ass_to_str(input_string):
    ords = []
    for i in range(0, len(input_string), 2):
        print(i)
        ords.append(chr(int(input_string[i] + input_string[i + 1])))
    return "".join(ords)
```
