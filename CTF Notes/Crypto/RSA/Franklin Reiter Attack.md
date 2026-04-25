Challenges:
- Dreamhack fl and ag

This attack is when two messages are linearly related. Meaning if:
$$M_1^e = C_1\ mod\ N$$ $$(KM_1 + r)^e = C_2\ mod\ N$$
Then $M_1$ is recoverable.

Sagemath code from [here](https://crypto.stackexchange.com/questions/30884/help-understanding-basic-franklin-reiter-related-message-attack).

```python
p = random_prime(2^512)
q = random_prime(2^512)
n = p * q # 1024-bit modulus

m = randint(0, n) # some message we want to recover
r = randint(0, n) # random padding

c1 = pow(m + 0, 3, n)
c2 = pow(m + r, 3, n)

R.<X> = Zmod(n)[]
f1 = X^3 - c1
f2 = (X + r)^3 - c2

# GCD is not implemented for rings over composite modulus in Sage
# so we'll do it ourselves. Might fail in rare cases, but we
# don't care.
def my_gcd(a, b): 
    return a.monic() if b == 0 else my_gcd(b, a % b)

print(m)
print(my_gcd(f1, f2).coefficients()[0]) # coefficient 0 = -m
```
