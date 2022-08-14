## lior@wildboar:/2021-crypto-ctf/triplets# cd ..

[Go Back](../index.md)

---------

# Triplets [50 solves, 91pt]

## Challenge Description

> Fun with RSA, [three times](./Triplet_b00247f67afefeb5b305957e5605dd70a3b8f5ca.txz)!
>
> nc 07.cr.yp.toc.tf 18010

## Solution

### Preface

Downloading and extracting the attached archive results in a python script file [Triplet.py](./Triplet.py)

```py
#!/usr/bin/env python3

from Crypto.Util.number import *
from random import randint
import sys
from flag import FLAG

def die(*args):
	pr(*args)
	quit()

def pr(*args):
	s = " ".join(map(str, args))
	sys.stdout.write(s + "\n")
	sys.stdout.flush()

def sc():
	return sys.stdin.readline().strip()

def main():
	border = "+"
	pr(border*72)
	pr(border, " hi talented cryptographers, the mission is to find the three RSA   ", border)
	pr(border, " modulus with the same public and private exponent! Try your chance!", border)
	pr(border*72)

	nbit = 160

	while True:
		pr("| Options: \n|\t[S]end the three nbit prime pairs \n|\t[Q]uit")
		ans = sc().lower()
		order = ['first', 'second', 'third']
		if ans == 's':
			P, N = [], []
			for i in range(3):
				pr("| Send the " + order[i] + " RSA primes such that nbit >= " + str(nbit) + ": p_" + str(i+1) + ", q_" + str(i+1) + " ")
				params = sc()
				try:
					p, q = params.split(',')
					p, q = int(p), int(q)
				except:
					die("| your primes are not valid!!")
				if isPrime(p) and isPrime(q) and len(bin(p)[2:]) >= nbit and len(bin(q)[2:]) >= nbit:
					P.append((p, q))
					n = p * q
					N.append(n)
				else:
					die("| your input is not desired prime, Bye!")
			if len(set(N)) == 3:
				pr("| Send the public and private exponent: e, d ")
				params = sc()
				try:
					e, d = params.split(',')
					e, d = int(e), int(d)
				except:
					die("| your parameters are not valid!! Bye!!!")
				phi_1 = (P[0][0] - 1)*(P[0][1] - 1)
				phi_2 = (P[1][0] - 1)*(P[1][1] - 1)
				phi_3 = (P[2][0] - 1)*(P[2][1] - 1)
				if 1 < e < min([phi_1, phi_2, phi_3]) and 1 < d < min([phi_1, phi_2, phi_3]):
					b = (e * d % phi_1 == 1) and (e * d % phi_2 == 1) and (e * d % phi_3 == 1)
					if b:
						die("| You got the flag:", FLAG)
					else:
						die("| invalid exponents, bye!!!")
				else:
					die("| the exponents are too small or too large!")
			else:
				die("| kidding me?!!, bye!")
		elif ans == 'q':
			die("Quitting ...")
		else:
			die("Bye ...")

if __name__ == '__main__':
	main()
```

We observe that the challenge is finding an RSA public key ($e$), private ($d$) key and $3$ RSA moduli ($3$ pairs of primes $p_i, q_i$) where $\log_2(p) \ge 160$ & $\log_2(q) \ge 160$. The difficult part in the challenge is that it enforces $e,d \le \min\limits_{i}(\phi(n_i))$.

### CRT is your best friend, utilize it in a creative way!

We know:

$ed \equiv 1 \pmod{(p_1-1)(q_1-1)}$

$ed \equiv 1 \pmod{(p_2-1)(q_2-1)}$

$ed \equiv 1 \pmod{(p_3-1)(q_3-1)}$


Chinese Remainder Theorem implies that for this conqurence system there exists a unique solution $N=ed \pmod{\text{lcm}((p1-1)(q1-1), (p2-1)(q2-1), (p3-1)(q3-1))}$

So, we want the LCM to be relatively small, and let $ed = \text{lcm}(\dots) + 1$

The idea: if we pick three primes $p, q, r$

$(p_1, q_1) = p, q$

$(p_2, q_2) = q, r$

$(p_3, q_3) = r, p$

$$
\text{lcm}{((p1-1)(q1-1), (p2-1)(q2-1), (p3-1)(q3-1))} = \\
= \text{lcm}{((p-1)(q-1), (q-1)(r-1), (p-1)(r-1))} \le\\ 
\le (p-1)(r-1)(q-1),
$$
Which is roughly ~$O(N_i^{1.5})$

for the LCM to be even smaller, we find primes of the form $2^{\text{big}} \cdot k + 1$ (let $\text{big} = 160$) and based on dirichlet distribution there should be many of them.

```py
from Crypto.Util.number import getPrime, isPrime
from Crypto.Util.number import GCD
from functools import reduce
import math
NBIT = 160


def phi(p, q):
    return (p-1) * (q-1)

def lcm(x, y):
    return (x*y) // GCD(x,y)

def main():

    start = 1 << NBIT
    k = 1
    res = []
    while len(res) < 3:
        if(isPrime(k*start + 1)):
            res.append(k*start+1)
        k += 1

    p, q, r = res
    print("p:", p)
    print("q:", q)
    print("r:", r)

    p1, q1, p2, q2, p3, q3 = [p, q, p, r, q, r]
    print("p1:", p1)
    print("q1:", q1)
    print("p2:", p2)
    print("q2:", q2)
    print("p3:", p3)
    print("q3:", q3)
    phi1 = phi(q1, p1)
    phi2 = phi(q2, p2)
    phi3 = phi(q3, p3)

    print(min(math.log2(p1 * q1), math.log2(q2 * p2), math.log2(q3 * p3)))
    print(phi1, phi2, phi3)
    print("\n\n\n")
    ed = lcm(lcm(phi1, phi2), phi3) + 1
    print(ed)

if __name__ == "__main__":
	main()
```

factorize $ed$ via factordb.com and pick $2$ factors, and task DONE!

Flag:

`CCTF{7HrE3_b4Bie5_c4rRi3d_dUr1nG_0Ne_pr39naNcY_Ar3_triplets}`

