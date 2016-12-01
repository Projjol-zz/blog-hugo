+++

title = "Understanding RSA"
date = "2016-11-24T12:09:47+05:30"

+++

# What is Public Key Cryptography?


Public Key Cryptography (AKA asymmetric cryptography) is a [cryptosystem][1] which consists of a pair of keys, i.e public and private. 
The public key can be widely disseminated whilst the private key should be known to the owner only. Public Key Cryptography(PKC from hereonforth) is also known as asymmetric cryptography because the public and private keys are different from each other and do not match. 
With PKC, to send a message to the recipient of the private key, one merely requires the public key. The public key can be used to sign and encrypt a message which can only be decrypted by the owner of the corresponding private key. 
The strength of a PKC cryptosystem depends on how hard it is to determine the private key by working backwards from the public key.

# What is RSA?

The RSA cryptosystem is named after it's creators _Rivest_, _Shamir_ & _Adleman_ and is one of the first implementations of a PKC cryptosystem.

It has 4 steps:

* Key generation
* Key distribution
* Encryption
* Decryption
 
Let's start with **Key Generation**
# Key Generation
Take two prime numbers (in proper implementations, large prime numbers are chosen, however for the sake of this post we'll choose smaller, easier to follow primes). Let _p_ & q be the prime numbers in question. A third number _n_ is a product of the chosen primes.


    n = p*q


_n_ is considered a modulus. Modulo arithmetic is used to calculate the remainder of a division operation. The idea behind using modulo is to find numbers in the same group, i.e 

    3 mod 7 = 3

    10 mod 7 = 3

This relationship can also be expressed as `10 ≡ 3 mod 7`, where '≡' is to represent congruence. In simpler terms, `10 ≡ 3 mod 7` is a shorter way of expressing `10 mod 7 = 3 mod 7`
Therefore, 3 and 10 are a part of the group where the remainder is 3. **Why this is important, I'll show later.**
While we're still on key generation it'll be a good idea to check out Euler’s totient formula

# Euler’s Totient

Euler’s totient, represented as φ, is a function on a number that returns the number of positive integers upto the number itself that are co-prime to it.

Lolwat. Let's break this down. Firstly, what's co-prime? Any two numbers whose [GCD][3] is 1 are considered to be co-prime to each other. So φ(n) finds all the numbers from 1 to _n_-1 that are co-prime to n. An example of this would be:

    φ(5) = 1,2,3,4  [here 1,2,3,4 have a GCD of 1 with 5]
    φ(6) = 1,5      

Euler’s totient has the property of being commutative i.e :
    
    φ(n) = φ(p)*φ(q)

This property will be of use once we re-vist the Key Generation process.

# Key Generation (contd.)

So, till this point we have calculated _n_, which is the product of two random prime numbers that we have selected. As mentioned earlier, _n_ is the modulus and is used to determine the length of the the public and private keys. This can be determined from the fact that the length of _n_ in bits is the length of the key itself. So for example someone uses a 1024 bit RSA key, the value of _n_ would be in the range of _0-2^1023_. 
Euler’s totient that we saw in the previous section comes of use now, especially the commutative property:

     φ(n) = φ(p)*φ(q) = (p − 1)(q − 1) = n − (p + q − 1)                 

For prime numbers φ(n) is 1-(_n_-1), check φ(5) above.

Considering _p_ & _q_ to be 7 & 13, _n_ = 91

Therefore, φ(_n_) = 91 - (7 + 13 - 1) =  72

This value calculated for Euler's totient is kept a secret.

Now, an integer _e_ needs to be obtained such that 1 > _e_ < φ(_n_) and gcd(_e_, φ(_n_)) =1, i.e _e_ & φ(_n_) are coprime.
Let, 
    
    e = 11

Next step is to compute the modular multiplicative inverse of e. 

Lolwat?


# Modular Multiplicative Inverse
Let’s say we have a number _n_. The multiplicative inverse of _n_ would be _n_^-1, such that _n_*_n_^-1 = 1
In modular mathematics, the modular inverse of _n_(mod _c_) is _n_^-1 for some number _c_. 

We have,
    
    n * n^-1 = 1 (mod c) = 1 
Therefore, 
    
    n * n(mod c) = 1

All of this is fine, but how do we calculate how to actually calculate the modular inverse?
    
Here's the long method:

* Calculate _n_ * _x_ ( x has values from 0- _c_-1)
* Modular inverse is the value of _x_ where _n_*_x_ mod _c_ = 1

Let n = 4, c = 7,

Values of x = {0,1,2,3,4,5,6}

* 4 * 0 ≡ 0 ≡ 0 (mod 7)
* 4 * 1 ≡ 4 ≡ 4 (mod 7)
* 4 * 2 ≡ 8 ≡ 1 (mod 7) —> bingo 


Let's see it in isolation:

    4(n) * 2(x) mod 7( c ) = 1 [This satisfies the condition we were looking for]

Note that this is a slow method to calculate the modular inverse of a number and using the [Extended Euclidean Algorithm][4], better performance can be achieved. 

# Key Generation (contd.)

Alright, so we have,

    e = 11, φ(n) = 72, 
And we need to calculate their modular multiplicative inverse such that:

    e*d mod φ(n) = 1 mod φ(n)

Let’s find the value of d!

* 11 * 0 ≡ 0 ≡ 0 (mod 72)
* 11 * 1 ≡ 11 ≡ 11 (mod 72)
* 11 * 2 ≡ 22 ≡ 22 (mod 72)
* ........
* 11 * 59 ≡ 649 ≡ 1 (mod 72) —> gotcha


Therefore,
    
    d = 59 
  
With this computation done, the public key is made with the values in _n_ and _e_ and the private key is made using values in _n_ and _d_. 

One question does arise though, why did we calculate the modular inverse of _e_ ? Consider the following:

    E(M) = M^e mod n
    C ≡ M^e mod n
    D(C) = C^d mod n
    C^d ≡ (M^e)^d mod n
    C^d ≡ (M^(e*d)) mod n
    e*d ≡ 1 mod n 
    e*d ≡ kϕ(n)+1
    => D(C) = M^(kϕ(n)+1) mod n
    => D(C) = M * (M^ϕ(n))^k
    => D(C) = M * (1 mod n)^k
    => D(C) = M * 1^k
    => D(C) = M         [original message]

By finding the inverse of e, we find the value which when the message is raised to gives back the original message, thereby acheiving a working public-private key cryptosystem. 

# Simple Example

    e = 11
    d = 59
    n = 91

Let the message be _9_

Encryption:
    
    E(M) = M^e mod n
    => 9^11 mod 91
    => 81
    D(C) = C^d mod n
    => 81^59 mod 91
    => 9 [the original message]
# Errata

The idea behind the public key is to have a prime number between 3 and φ(n). Normally, values behind public key cryptography involves large numbers (if you remember, the size of the key could be as high as 2^1023 or 2^2023 but we’re mostly pushing it at key lengths above 2024 bits), so popular implementations set it as [65537][2] as a good tradeoff between sufficiently high and being a decent enough cost to raise a number to. 
A good point that might be made is whether or not the fact that everyone knows if 65537 is a public key exponent. This is the public key that is being shared with people anyway and as long as one cannot deduce the private key from it, there should not be a problem.  


[1]: https://en.wikipedia.org/wiki/Cryptosystem
[2]: http://crypto.stackexchange.com/questions/3110/impacts-of-not-using-rsa-exponent-of-65537 
[3]: https://www.khanacademy.org/math/pre-algebra/pre-algebra-factors-multiples/pre-algebra-greatest-common-divisor/v/greatest-common-divisor
[4]: https://www.math.utah.edu/~fguevara/ACCESS2013/Euclid.pdf
