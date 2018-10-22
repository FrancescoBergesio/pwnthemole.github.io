---
title: Cryptowars2018- 0e,5e,12e Writeup
layout: post
author: "matpro98"
category: crypto ecc Smart-ASS
tags: crypto classic-crypto permutations
---

The challenge asks to break the ECC keys finding the private exponent $$k$$, i.e the smallest positive integer such that $$kP=Q$$. The elliptic curves here are defined by the equation $$y^2=x^3+Ax+B$$.
## 0e
We have the following parameters:

```
p=2^256-2^32-2^9-2^8-2^7-2^6-2^4-1
A=0
B=7
xP=55066263022277343669578718895168534326250603453777594175500187360389116729240
yP=32670510020758816978083085130507043184471273380659243275938904335757337482424
xQ=72488970228380509287422715226575535698893157273063074627791787432852706183111
yQ=62070622898698443831883535403436258712770888294397026493185421712108624767191
```

Let's try with an exhaustive search (the code is written in sage):

```
p=2^256-2^32-2^9-2^8-2^7-2^6-2^4-1
A=0
B=7
xP=55066263022277343669578718895168534326250603453777594175500187360389116729240
yP=32670510020758816978083085130507043184471273380659243275938904335757337482424
xQ=72488970228380509287422715226575535698893157273063074627791787432852706183111
yQ=62070622898698443831883535403436258712770888294397026493185421712108624767191

P=[xP,yP]
Q=[xQ,yQ]
F = FiniteField(p)
E = EllipticCurve(F,[A,B])
P = E.point(P)
Q = E.point(Q)
R=P
k=1
while R!=Q:
    k+=1
    R=k*P

print(k)
print(k*P==Q)
```

which returns:

```
10
True
```

Thus $$k=10$$ and the key is broken.

## 5e

We have

```
p=12506217790875063466368723611056175369923
A=12506217790875063466368723611052784275139
B=12506217790875063466368723533070038257347
xP=7493372729181057645036574086903590138065
yP=359098907392057890604329721532958479621
xQ=9505420031620208163682758801913524369821
yQ=5460936589331844194485299189975059431657
```

This time we are unlucky and we can't bruteforce the key. But if we look for the order of $$E$$ over the field $$F_p$$ we have:

```
print(E.order())
```

and the result is `12506217790875063466368723611056175369923`, which is exactly $$p$$. Hence we can find the private exponent with the additive transfer attack, aka the Smart-ASS attack.

The full code is then:

```
p=12506217790875063466368723611056175369923
A=12506217790875063466368723611052784275139
B=12506217790875063466368723533070038257347
xP=7493372729181057645036574086903590138065
yP=359098907392057890604329721532958479621
xQ=9505420031620208163682758801913524369821
yQ=5460936589331844194485299189975059431657
P=[xP,yP]
Q=[xQ,yQ]
F = FiniteField(p)
E = EllipticCurve(F,[A,B])
P = E.point(P)
Q = E.point(Q)
assert E.order() == p

Qp = Qp(p, 2)
Ep = EllipticCurve(Qp, [0, 0, 0, A, B])

yPp = sqrt(Qp(xP) ** 3 + Qp(A) * Qp(xP) + Qp(B))
Pp = Ep(Qp(xP), (-yPp, yPp)[yPp[0] == yP])

yQp = sqrt(Qp(xQ) ** 3 + Qp(A) * Qp(xQ) + Qp(B))
Qp = Ep(Qp(xQ), (-yQp, yQp)[yQp[0] == yQ])lQ = Ep.formal_group().log()(- (p * Qp)[0] // (p * Qp)[1]) / p
lP = Ep.formal_group().log()(- (p * Pp)[0] // (p * Pp)[1]) / p

e = lQ / lP

assert e[0] * E(xP, yP) == E(xQ, yQ)

print(e[0])
```

And we have $$k=11063983210789056064816501731025997778953$$.

## 12e

This time we have

```
p=30772300251428474897541404692968263716949812908443831592876046737810737208988156271014198502145416667717788718445610314549722607794124248272637226302317
A=30772300251428474897541404692968263716949812908443831592876046737810737208988156271014198502145416667717788718445610314549722607794121899994681666878317
B=30772300251428474897541404692968263716949812908443831592876046737810737208988156271014198502145416667717788718445610314549721222720722172186131393854317
xP=29561516241345269685600719840366915070758730476477194843156185445081418419687711726455154356975229698728353175026723190494273440744152320729175746030047
yP=6271596900388272418806185389049176826094544788645246439903066132798365469828717343340379070431849309644910923054338004606592187702166087195126836872974
xQ=9803494533476252078182975745467080970879117115916552756575328888804137399672877069174914310927425369894015322041756406128521349864943085492910753608888
yQ=13144336243557287405975237486494628680060717357027253199829545937735714808116560433321436824035889516510417399701805550140250708344883560551017907939240
```

Again, the order of $$E$$ over the field $$F_p$$ equals $$p$$ and the attack is the same as before. We have now $$k=4770968584243760060935329622525117398553912181733840860750148860803298370830880865702624434943891722451205710599936197775227009974950704028595510809166$$.