
## 逆元

> 如果 ax≡1（mod M)，就称x为在模M下 a的 逆元
> 简单地说，如果一个数x满足 ax%M=1，那么x就称为在模M下 a的 逆元！


## 模运算

模运算与基本四则运算有些相似，但是除法除外。其规则如下：

```
(a + b) % p = (a % p + b % p) % p
(a - b) % p = (a % p - b % p) % p
(a * b) % p = (a % p * b % p) % p
a ^ b % p = ((a % p) ^ b) % p
结合律
((a + b) % p + c) = (a + (b + c) % p) % p
((a * b) % p * c) = (a * (b * c) % p) % p
交换律
(a + b) % p = (b + a) % p
(a * b) % p = (b * a) % p
分配律
(a + b) % p = (a % p + b % p) % p
((a + b) % p * c) % p = ((a * c) % p + (b * c) % p
重要定理
若 a ≡ b (mod p)，则对于任意的 c，都有(a + c) ≡ (b + c) (mod p)
若 a ≡ b (mod p)，则对于任意的 c，都有(a * c) ≡ (b * c) (mod p)
若 a ≡ b (mod p)，c ≡ d (mod p)，则
(a + c) ≡ (b + d) (mod p)
(a - c) ≡ (b - d) (mod p)
(a * c) ≡ (b * d) (mod p)
(a / c) ≡ (b / d) (mod p) 
```

在模运算下是没有除法运算的，比如:

$$\begin{aligned} 
a * b \equiv c \ mod \ n  \\
c * \frac{1}{a} \equiv b \mod\ n 是错误的 \\
c * a^{-1} \equiv \ b \mod\ n 才是对的
\end{aligned} $$
## 定理

| 定理       | 条件/描述                                                                  | 公式                                                                                                                            |
| -------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| 费马小定理    | 对于质数 p，当 a 是一个与 p互质的整数时                                                | a^p-1≡1(mod p)；即可化为a * a^p-2 ≡1(mod p)                                                                                        |
| 欧几里得算法   |                                                                        | gcd(a,b)=gcd(b,a)=gcd(−a,b)=gcd(∣a∣,∣b∣)=gcd(b, a mod b)                                                                      |
| 扩展欧几里得   | 对于不完全为 0 的整数 a，b，gcd（a，b）表示 a，b 的最大公约数，必然存在整数对 x，y ，使得 gcd（a，b）=ax+by。 | gcd（a，b）=ax+by;可以使用gmpy2.gcdext(a,b)求解x、y的组合；并使用`x+i*b`的方式进行爆破；对于ax≡1(mod p)可以化成`ax-kp=1`(k为整数)，令k = -k，且a，p互质，那么就可用上扩展欧几里得了。 |
| 威尔逊定理    | 对于素数p                                                                  | (p-1)!≡-1(mod p)                                                                                                              |
| 欧拉定理     | 若n,a为正整数，且n,a互质                                                        | `a^φ(n)≡1 mod n `                                                                                                             |
| 中国剩余定理   |                                                                        |                                                                                                                               |
| 欧拉准则     | p为素数                                                                   | a^((p-1)/2)  =   {a/p}mod p                                                                                                   |
| 二次剩余定理   | p是素数、a!=kp、存在x，使得x^2 = a mod p ，那么我们称a是模p的二次剩余，记为QR，否则记NR              | QR，{a/p}=1 ；NR {a/p=-1}                                                                                                       |
| 二次剩余定理拓展 | 如果一个数是模p的二次剩余，另外一个不是                                                   | 那么这2个数的积不是模p的二次剩余                                                                                                             |
|          |                                                                        |                                                                                                                               |

### 补充式子

1. a和n互质：`a^(-y) % n = invert(a,n)^y % n`
2. $\phi(p^k*q)=p^(k-1)*(p-1)*(q-1)$
3. $$(a+b)^n = \sum_{k=0}^{n} \binom{n}{k} a^{n-k} (b[可正可负])^k$$


### 欧拉函数

>欧拉函数（Euler's totient function），即phi_n，表示的是小于等于n和n互质的数的个数。特别的，当n是质数时，phi_n = n-1


### 欧拉定理

>若n,a为正整数，且n,a互质，则:`a^φ(n)≡1 mod n `


### 欧几里得算法

> `gcd(a,b)=gcd(b,a)=gcd(−a,b)=gcd(∣a∣,∣b∣)`
> `gcd(a,b)=gcd(b, a mod b)`


### 扩展欧几里得

>对于不完全为 0 的非负整数 a，b，gcd（a，b）表示 a，b 的最大公约数，必然存在整数对 x，y ，使得 gcd（a，b）=ax+by。
>`gcd（a，b）=ax+by`;

```
>可以使用gmpy2.gcdext(a,b)求解x、y的组合；并使用`x+i*b`的方式进行爆破
```


### 费马小定理

>对于质数 p，当 a 是一个与 p互质的整数时有：`a**(p-1)≡1(mod p)`；
>即可化为`a * a**(p-2) ≡1(mod p)`


### 威尔逊定理

>对于素数p,`(p-1)!≡-1(mod p)`



### 中国剩余定理

> 物不知数」问题：有物不知其数，三三数之剩二，五五数之剩三，七七数之剩二。问物几何？

![[attachments/Pasted image 20231106224355.png]]
解题

```python

import gmpy2
from Crypto.Util.number import long_to_bytes

def CRT(k_count, a_result_pre_mod, n_mod):
    n = 1; ans = 0
    for i in range(0, k_count ):
        n = n * n_mod[i]
    for i in range(0, k_count ):
        m = n // n_mod[i]; b = y = 0
        b = gmpy2.invert(m, n_mod[i]) # b * m mod r[i] = 1
        ans = (ans + a_result_pre_mod[i] * m * b % n) % n
    return (ans % n + n) % n

```



### 欧拉准则+二次剩余

![[attachments/Pasted image 20231201154721.png]]





