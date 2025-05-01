# SHA-1
# Date : 
# Register No.:
# Aim: To implement the SHA-I hashing technique using C program. 
# ALGORITHM:
Step1:Initialize Variables
Set five 32-bit words:
h0 = 0x67452301  
h1 = 0xEFCDAB89  
h2 = 0x98BADCFE  
h3 = 0x10325476  
h4 = 0xC3D2E1F0
Step2: Preprocessing (Padding)
a. Append a single 1 bit to the message
b. Append 0 bits until the length is 64 bits short of a multiple of 512
c. Append the original message length (in bits) as a 64-bit big-endian integer
Step3: Process the Message in 512-bit Blocks
For each 512-bit block: a. Divide block into 16 words W[0..15] (32 bits each)
b. Extend these into 80 words:
for i from 16 to 79:
W[i] = (W[i-3] XOR W[i-8] XOR W[i-14] XOR W[i-16]) left-rotated by 1
Step4: Initialize Working Variables
a = h0, b = h1, c = h2, d = h3, e = h4
Step5: Main Compression Loop (80 Rounds)
For i from 0 to 79:
Determine: f and k based on i:
i =  0–19: f = (b & c) | (~b & d), k = 0x5A827999  
i = 20–39: f = b ^ c ^ d,           k = 0x6ED9EBA1  
i = 40–59: f = (b & c) | (b & d) | (c & d), k = 0x8F1BBCDC  
i = 60–79: f = b ^ c ^ d,           k = 0xCA62C1D6  
Compute:
temp = ROTL(a,5) + f + e + k + W[i]  
e = d, d = c, c = ROTL(b,30), b = a, a = temp
Step6: Add This Block’s Hash to the Result
h0 = h0 + a  
h1 = h1 + b  
h2 = h2 + c  
h3 = h3 + d  
h4 = h4 + e
Final Output
Concatenate h0 || h1 || h2 || h3 || h4 to produce the final 160-bit hash.


# Program:

# Output:
# Result:

