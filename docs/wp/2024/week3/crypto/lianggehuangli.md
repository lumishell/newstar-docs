---
titleTemplate: ":title | WriteUp - NewStar CTF 2024"
---

# 两个黄鹂鸣翠柳

整体就是一个稍加变形的关联信息攻击，只是一般的关联信息攻击给出的两个方程式信息比较明确，而在本题中各掺了一点点随机性的干扰小量。

所以其实在思路上只需要依整体代换思想把它改为一般的关联信息攻击，再对被代换的「整体」进行爆破求解即可。

此外由于本题对于较大的数据求最大公因子的需求，一般的 GCD 算法难以胜任，所以需要使用 Half-GCD 算法。关于这个算法的原理可以参考下面这篇文章：

- [多项式 gcd 的正确姿势：Half-GCD 算法](https://www.cnblogs.com/whx1003/p/16217087.html)

解题脚本如下：

```python
from Crypto.Util.number import *

def HGCD(a, b):
    if 2 * b.degree() <= a.degree() or a.degree() == 1:
        return 1, 0, 0, 1
    m = a.degree() // 2
    a_top, a_bot = a.quo_rem(x ^ m)
    b_top, b_bot = b.quo_rem(x ^ m)
    R00, R01, R10, R11 = HGCD(a_top, b_top)
    c = R00 * a + R01 * b
    d = R10 * a + R11 * b
    q, e = c.quo_rem(d)
    d_top, d_bot = d.quo_rem(x ^ (m // 2))
    e_top, e_bot = e.quo_rem(x ^ (m // 2))
    S00, S01, S10, S11 = HGCD(d_top, e_top)
    RET00 = S01 * R00 + (S00 - q * S01) * R10
    RET01 = S01 * R01 + (S00 - q * S01) * R11
    RET10 = S11 * R00 + (S10 - q * S11) * R10
    RET11 = S11 * R01 + (S10 - q * S11) * R11
    return RET00, RET01, RET10, RET11

def related_message_attack(a, b):
    q, r = a.quo_rem(b)
    if r == 0:
        return b
    R00, R01, R10, R11 = HGCD(a, b)
    c = R00 * a + R01 * b
    d = R10 * a + R11 * b
    if d == 0:
        return c.monic()
    q, r = c.quo_rem(d)
    if r == 0:
        return d
    return related_message_attack(d, r)

e =  683
c1 =  56853945083742777151835031127085909289912817644412648006229138906930565421892378967519263900695394136817683446007470305162870097813202468748688129362479266925957012681301414819970269973650684451738803658589294058625694805490606063729675884839653992735321514315629212636876171499519363523608999887425726764249
c2 =  89525609620932397106566856236086132400485172135214174799072934348236088959961943962724231813882442035846313820099772671290019212756417758068415966039157070499263567121772463544541730483766001321510822285099385342314147217002453558227066228845624286511538065701168003387942898754314450759220468473833228762416
N =  147146340154745985154200417058618375509429599847435251644724920667387711123859666574574555771448231548273485628643446732044692508506300681049465249342648733075298434604272203349484744618070620447136333438842371753842299030085718481197229655334445095544366125552367692411589662686093931538970765914004878579967
delta =  93400488537789082145777768934799642730988732687780405889371778084733689728835104694467426911976028935748405411688535952655119354582508139665395171450775071909328192306339433470956958987928467659858731316115874663323404280639312245482055741486933758398266423824044429533774224701791874211606968507262504865993

is_flag = False

for delt in range(-255, 255, 8):

    PR.<x> = PolynomialRing(Zmod(N))
    f = x ^ e - c1
    g1 = ((x + (delt + 0) * delta) ^ e - c2) * ((x + (delt + 1) * delta) ^ e - c2)
    g2 = ((x + (delt + 2) * delta) ^ e - c2) * ((x + (delt + 3) * delta) ^ e - c2)
    g3 = ((x + (delt + 4) * delta) ^ e - c2) * ((x + (delt + 5) * delta) ^ e - c2)
    g4 = ((x + (delt + 6) * delta) ^ e - c2) * ((x + (delt + 7) * delta) ^ e - c2)
    if delt == -7:
        g4 = ((x + (delt + 6) * delta) ^ e - c2)
    g = g1 * g2 * g3 * g4
    res = related_message_attack(f, g)
    m1 = int(-res.monic().coefficients()[0])
    for t1 in range(256):
        m = (m1 % N - t1 * delta) % N
        if m > 0:
            flag = long_to_bytes(m)
            if flag[:4] ==b'flag':
                print(flag)
                is_flag = True
                break

    if is_flag:
        break
```
