
# description

we're given a `challenge.py` file
```py
import numpy as np
import random

polyc = [4,3,7]
poly = np.poly1d(polyc)


def generate_random_number():
    while True:
        num = random.randint(100, 999)
        first_digit = num // 100
        last_digit = num % 10
        if abs(first_digit - last_digit) > 1:
            return num


def generate_random_number_again():
    while True:
        num = random.randint(1000, 9999)
        if num % 1111 != 0:
            return num


def transform(num):
    number = random.randint(1, 100000)
    org = number
    number *= 2
    number += 15
    number *= 3
    number += 33
    number /= 6
    number -= org
    if number == 13:
        num1 = random.randint(1, 6)
        num2 = random.randint(1, 6)
        number = num1 * 2
        number += 5
        number *= 5
        number += num2
        number -= 25
        if int(number / 10) == num1 and number % 10 == num2:
            number = generate_random_number()
            num1 = int(''.join(sorted(str(number), reverse=True)))
            num2 = int(''.join(sorted(str(number))))
            diff = abs(num1 - num2)
            rev_diff = int(str(diff)[::-1])
            number = diff + rev_diff
            if number == 1088:
                org = num
                num *= 2
                num /= 3
                num += 5
                num *= 4
                num -= 9
                num -= org
                return num
            else:
                number = generate_random_number_again()
                i = 0
                while number != 6174:
                    digits = [int(d) for d in str(number)]
                    digits.sort()
                    smallest = int(''.join(map(str, digits)))
                    digits.reverse()
                    largest = int(''.join(map(str, digits)))
                    number = largest - smallest
                    i += 1

                if i <= 7:
                    org = num
                    num *= 2
                    num += 7
                    num += 5
                    num -= 12
                    num -= org
                    num += 4
                    num *= 2
                    num -= 8
                    num -= org
                    return num
                else:
                    org = num
                    num **= 4
                    num /= 9
                    num += 55
                    num *= 6
                    num += 5
                    num -= 23
                    num -= org
                    return num
        else:
            org = num
            num *= 10
            num += 12
            num **= 3
            num -= 6
            num += 5
            num -= org
            return num
    else:
        org = num
        num += 5
        num -= 10
        num *= 2
        num += 12
        num -= 20
        num -= org
        return num


def encrypt(p,key):
    return ''.join(chr(p(transform(i))) for i in key)


key = open('key.txt', 'rb').read()
enc = encrypt(poly,key)
print(enc)
```

it opens a `key.txt` file, encrypt it using a polynomial after applying the `transform` function.

i started by examining the code, at first look, the files given to us are `enc.txt` and `key.txt`, i thought the key was given to us, but if it given to us, where is the flag suppose to be in `challenge.py`, after much woundering and almost given-up, it seems that the `key.txt` file was changed to `encoded-key.txt`, now we know that the `challenge.py` script is used to encode the key, not the flag

# solution

experiencing with `transrform` function shows that it actually does nothing, so we just need to solve the second order equation on every character of the encoded key to recover the original key, after recovering it, we try multiple symitric key algorithm in order to recover the flag, the used algo was AES 128 in CBC mode

recovering the key:
```py
from z3 import *
from pwn import xor


x = Int('x')
encoded_key = open("encoded_key.txt", "r").read()

equation = 4*(x**2) + 3*x + 7

decoded_key = b''
for key in encoded_key:
    s = Solver()
    s.add(equation == ord(key))
    if s.check() == sat:
        model = s.model()
        solution = model[x]
        decoded_key += chr(solution).encode()

print(decoded_key)
```

decrypting the flag:
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import base64

def decrypt_aes(cipher_text, key):
    # Decode the cipher text from base64
    cipher_text = base64.b64decode(cipher_text)

    # Initialize AES cipher object with key and mode
    cipher = AES.new(key, AES.MODE_ECB)

    # Decrypt the cipher text and remove padding
    decrypted_text = unpad(cipher.decrypt(cipher_text), AES.block_size)

    return decrypted_text.decode('utf-8')

cipher_text = b'u5FUKxDUxH9y8yxvfaaU+GSXDwvJS6QxlN/3udOEzpU6fIVUExjDLsB3LKqUTz/x'
key = b'12345678910111213141516171819202'  # Example key

decrypted_text = decrypt_aes(cipher_text, key)
print("Decrypted Text:", decrypted_text)
```
