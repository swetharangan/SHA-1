# SHA-1
# Register No.: 212223040221
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
```
#include <stdio.h>
#include <string.h>
#include <stdint.h>

// Define the SHA-1 circular left shift macro
#define LEFTROTATE(x, c) (((x) << (c)) | ((x) >> (32 - (c))))

// SHA-1 context structure
typedef struct {
    uint8_t data[64];
    uint32_t datalen;
    unsigned long long bitlen;
    uint32_t state[5];
} SHA1_CTX;

// SHA-1 initialization constants
uint32_t k[4] = { 0x5a827999, 0x6ed9eba1, 0x8f1bbcdc, 0xca62c1d6 };

// Functions for processing and updating the hash
void sha1_transform(SHA1_CTX *ctx, uint8_t data[]) {
    uint32_t a, b, c, d, e, i, j, t, m[80];
    
    for (i = 0, j = 0; i < 16; ++i, j += 4)
        m[i] = (data[j] << 24) | (data[j + 1] << 16) | (data[j + 2] << 8) | (data[j + 3]);
    
    for ( ; i < 80; ++i)
        m[i] = LEFTROTATE(m[i - 3] ^ m[i - 8] ^ m[i - 14] ^ m[i - 16], 1);
    
    a = ctx->state[0];
    b = ctx->state[1];
    c = ctx->state[2];
    d = ctx->state[3];
    e = ctx->state[4];
    
    for (i = 0; i < 80; ++i) {
        if (i < 20)
            t = LEFTROTATE(a, 5) + ((b & c) | (~b & d)) + e + k[0] + m[i];
        else if (i < 40)
            t = LEFTROTATE(a, 5) + (b ^ c ^ d) + e + k[1] + m[i];
        else if (i < 60)
            t = LEFTROTATE(a, 5) + ((b & c) | (b & d) | (c & d)) + e + k[2] + m[i];
        else
            t = LEFTROTATE(a, 5) + (b ^ c ^ d) + e + k[3] + m[i];
        
        e = d;
        d = c;
        c = LEFTROTATE(b, 30);
        b = a;
        a = t;
    }
    
    ctx->state[0] += a;
    ctx->state[1] += b;
    ctx->state[2] += c;
    ctx->state[3] += d;
    ctx->state[4] += e;
}

void sha1_init(SHA1_CTX *ctx) {
    ctx->datalen = 0;
    ctx->bitlen = 0;
    ctx->state[0] = 0x67452301;
    ctx->state[1] = 0xefcdab89;
    ctx->state[2] = 0x98badcfe;
    ctx->state[3] = 0x10325476;
    ctx->state[4] = 0xc3d2e1f0;
}

void sha1_update(SHA1_CTX *ctx, const uint8_t data[], size_t len) {
    for (unsigned int i = 0; i < len; ++i) {
        ctx->data[ctx->datalen] = data[i];
        ctx->datalen++;
        if (ctx->datalen == 64) {
            sha1_transform(ctx, ctx->data);
            ctx->bitlen += 512;
            ctx->datalen = 0;
        }
    }
}

void sha1_final(SHA1_CTX *ctx, uint8_t hash[]) {
    unsigned int i = ctx->datalen;
    
    if (ctx->datalen < 56) {
        ctx->data[i++] = 0x80;
        while (i < 56)
            ctx->data[i++] = 0x00;
    } else {
        ctx->data[i++] = 0x80;
        while (i < 64)
            ctx->data[i++] = 0x00;
        sha1_transform(ctx, ctx->data);
        memset(ctx->data, 0, 56);
    }
    
    ctx->bitlen += ctx->datalen * 8;
    ctx->data[63] = ctx->bitlen;
    ctx->data[62] = ctx->bitlen >> 8;
    ctx->data[61] = ctx->bitlen >> 16;
    ctx->data[60] = ctx->bitlen >> 24;
    ctx->data[59] = ctx->bitlen >> 32;
    ctx->data[58] = ctx->bitlen >> 40;
    ctx->data[57] = ctx->bitlen >> 48;
    ctx->data[56] = ctx->bitlen >> 56;
    sha1_transform(ctx, ctx->data);
    
    for (i = 0; i < 4; ++i) {
        hash[i]      = (ctx->state[0] >> (24 - i * 8)) & 0x000000ff;
        hash[i + 4]  = (ctx->state[1] >> (24 - i * 8)) & 0x000000ff;
        hash[i + 8]  = (ctx->state[2] >> (24 - i * 8)) & 0x000000ff;
        hash[i + 12] = (ctx->state[3] >> (24 - i * 8)) & 0x000000ff;
        hash[i + 16] = (ctx->state[4] >> (24 - i * 8)) & 0x000000ff;
    }
}

int main() {
    // initializing string
    char *str = "swetha";
    printf("Text: %s\n", str);
    
    // Create SHA1 context and hash buffer
    SHA1_CTX ctx;
    uint8_t hash[20];
    
    // Initialize, update, and finalize SHA-1 hash
    sha1_init(&ctx);
    sha1_update(&ctx, (unsigned char*)str, strlen(str));
    sha1_final(&ctx, hash);
    
    // Printing the equivalent hexadecimal value
    printf("\nThe hexadecimal equivalent of SHA1 is: ");
    for (int i = 0; i < 20; i++) {
        printf("%02x", hash[i]);
    }
    printf("\n");

    return 0;
}
```
# Output:
<img width="1910" height="733" alt="Screenshot 2025-10-25 085341" src="https://github.com/user-attachments/assets/1be44799-f637-4ea4-8ea0-96b4b63c41b3" />

# Result:

Thus the SHA-1 hashing technique had been implemented successfully
