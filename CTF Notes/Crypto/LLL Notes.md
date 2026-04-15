Standard shape:  
```  
[x, 1, 0, 0]  
[y, 0, 1, 0]  
[z, 0, 0, 1]  
```  
  
This solves for `a, b, c` in $ax + by = cz$. (You might have to flip all the signs). It solves it by looking for the **shortest vector**. The "length" of a vector is calculated by $\sqrt{v_0^2 + \ldots + v_i^2}$ where $v_i$ is the i-th element in the v  
ector.  
  
Thus, in most LLL challenges, the goal is to embed information to minimise the size of the vector.  
  
We can predict the size of the shortest vector using **Minowski's Theorem**.  
  
### To add a modulus:  
  
Add a vector `[p, 0, 0, 0]`. This lets LLL use p an arbitrary amount of times without any weight penalty. This lets modular arithmetic under p still work.  
  
### To force the resulting vector to sum to evaluate to 0:  
  
Multiply the first column by a huge weight W. This will make any error very large. This is because the first column is equals to $M(ax + by + cz)$. When M is large, the first element in the vector is large, and LLL will suffer a huge penalty.  
  
### To force an element to be used once:  
  
Multiply the element's identity by a huge weight. (This might make the element be used 0 times).  
  
### To weight elements:  
  
Multiply the element's identity by 1/W where W is the expected size of the target element we're trying to find.  
  
### Double modulus:  
  
Unlike with a single modulo, we should not have two rows of 0s if there is a double modulus. This is because with two coprime integers, by Bezout's Identity, it can find a vector where the first element is 0.  
  
Instead, weigh the larger modulus.  
  
### Two equations, with shared coeffs  
  
Sometimes, we are given more than one equation, for example:  
  
$$ax + by = cz$$  
$$ai + bj = ck$$  
  
Where $a$, $b$ and $c$ are shared amongst the two equations.  
  
We can embed this information in the lattice by simply adding another row  
  
```  
[x, i, 1, 0, 0],  
[y, j, 0, 1, 0],  
[z, k, 0, 0, 1],  
```  
  
Let's say that $c$ is not shared amonst the two equations. In this case, we simply make another row for c.  
  
```  
[x, i, 1, 0, 0, 0],  
[y, j, 0, 1, 0, 0],  
[z, 0, 0, 0, 1, 0],  
[0, k, 0, 0, 0, 1],  
```  
  
### Low density subset-sum attack (Merkle-Hellman knapsack)  
  
READ: https://web.stevens.edu/algebraic/Files/SubsetSum/p229-lagarias.pdf  
  
Construct the matrix as so:  
  
```  
[2 * r_0, 2, 0, 0],  
[2 * r_1, 0, 2, 0],  
[2 * r_2, 0, 0, 2],  
[2 * c,   1, 1, 1],  
```  
  
Where we are trying to find $x_i$ in    
  
$$\sum_{i=0}^n x_i r_i = c$$  
  
This results in the vector LLL finds being in the form:  
  
`[0, 1, -1, 1]`  
  
Each element in the vector (other than the first row) is equals to $2x_i - 1$. (or $-(2x_i - 1)$)  
  
This shows us that information doesn't have to be embedded in the identity form i introduced earlier. Information about `c` is embedded in the last row of 1s. The goal is for LLL to find the vector where the last element is `c` coeff is -1, making a pa  
rticularly short vector.