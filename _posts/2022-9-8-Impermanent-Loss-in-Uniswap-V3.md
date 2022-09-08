---
toc: true
layout: post
description: The detailed derivation of the impermanent loss in Uniswap V3, prepared by Zelos, a forward-thinking blockchain lab.
categories: [markdown]
title: Impermanent Loss in Uniswap V3
---

# Impermanent Loss in Uniswap V3
### Zelos

Uniswap is a decentralized exchange with automated market making capabilities. Uniswap V2 extends the functionality to exchanges between any token pair. v2's liquidity provider provides liquidity at an exchange rate of $[0,∞)$. v3 allows the liquidity provider to set an arbitrary finite range of exchange rates and provide liquidity only within the range it sets. (also known as the liquidity function) to set the price of each transaction.

In this article, we will take the perspective of a liquidity provider(LP) and derive the mechanism for providing liquidity in V3 in detail, starting from the V2 liquidity function. Finally, we will derive the impermanent loss in V3.

## Uniswap V2 introduction
We start by entering the trading pool for coins X, Y. Use x, y to represent the number of both coins in the trading pool. We define $K$ as the liquidity of the trading pool and $P$ as the exchange rate. P is ranging from 0 to infinity.

$$
x\cdot y=K,\ \ \frac{y}{x} = P
$$

The formula can also be written as:

$$
x=\sqrt{\frac{K}{P}},\ \ y=\sqrt{KP}\tag{1}
$$
### Properties
V2 has the following two properties.
1. When coins are exchanged in a trading pool, P changes while liquidity remains constant, i.e., K stays constant. 
$$
\begin{cases}
    \Delta x=\Delta\frac{1}{\sqrt{P}}\cdot \sqrt{K}\\
    \Delta y=\Delta\sqrt{P}\cdot \sqrt{K}
\end{cases}\tag{2}
$$
2. When people provide liquidity or withdraw, $P$ remains the same while $K$ changes. Thus the change in position can be expressed as follows.
$$
\begin{cases}
    \Delta x=\frac{1}{\sqrt{P}}\cdot \Delta\sqrt{K}\\
    \Delta y=\sqrt{P}\cdot \Delta\sqrt{K}
\end{cases}\tag{3}
$$


These two properties also carry over to V3.

## V3 Mechanism
In V3, the liquidity provider can choose the scope of market making.

 $$P\in[P_L,P_H]$$

This means that all assets are X when the price reaches the lower bound and Y when the price is $P_H$. Thus the X real reserve and Y real reserve required for a V3 position can be represented by the orange line in the figure

See figure in: <https://www.desmos.com/calculator/shdzosurvx>

The image of V3 can be obtained by translating V2. For the offsets we know the following relations.
- The point $(x,0)$ will map to the point $P_L$ on the V2 curve.
- The point $(y,0)$ will map to the point $P_H$ on the V2 curve.

We use $x', y'$ represent the points on V2 and We can get the mapping function as:
$$
\begin{split}
x'=x+\sqrt{\frac{K}{P_H}}\\ 
y'=y+\sqrt{KP_L}
\end{split}
$$
Substituting into $(1)$ we obtain:
$$
\begin{split}
x=\sqrt{K}(\frac{1}{\sqrt{P}}-\frac{1}{\sqrt{P_H}})\\
y=\sqrt{K}(\sqrt{P}-\sqrt{P_L})\tag{4}
\end{split}
$$
Once the LP has selected the market making range, the ratio of the two coins to be provided by the LP is determined.
$$
r=\frac{y}{x} = \frac{\sqrt{P}-\sqrt{P_L}}{\frac{1}{\sqrt{P}}-\frac{1}{\sqrt{P_H}}}
$$
The current exchange rate $P$ is known. Three of the four variables $x, y, P_H, P_L$ are set, and the other one is determined accordingly.

## V3 impermanent loss
The impermanent loss is:
$$
Loss=\frac{V_{1}-V_{F}}{V_{0}}
$$
### The initial asset value $V_0$
We define the initial asset value as $V_0$ at $T_0$ with exchange rate $P$ as $S_0$. We assume that  $S_0$ is in $[P_L,P_H]$.
$$
V_0=y_0+S_0 \cdot x_0\\
$$
Substituting into $(2)$ we obtain:
$$
V_0=\sqrt{KS_0}(2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}})\tag{5}
$$
and:
$$
\sqrt{K} = \frac{V_0}{\sqrt{S_0}(2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}})\tag{6}}
$$
### The terminal asset value $V_T$
At $T_1$ moment, $P$ turns to $S_T$ from $S_0$. The asset value for LP is:
$$
V_T =\begin{cases}
   S_Tx_1 & S_T\leq P_L \\
   y_1+S_T\cdot x_1 & P_L < S_T< P_H \\
y_1 & P_H\leq S_T
\end{cases}\tag{7}
$$
When $S_T\leq P_L$, the price is sliding down past $P_L$, all of asset Y is transformed into asset X. Substituting into $(2)(4)$:
$$
\begin{split}
\Delta x &=x_1-x_0=\sqrt{K}(\frac{1}{\sqrt{P_L}}-\frac{1}{\sqrt{S_0}}) \\

x_1&=x_0+\Delta x \\
&=\sqrt{K}(\frac{1}{\sqrt{S_0}}-\frac{1}{\sqrt{P_H}})+\sqrt{K}(\frac{1}{\sqrt{P_L}}-\frac{1}{\sqrt{S_0}}) 
\end{split}
$$
Thus we have:
$$
x_1 = \sqrt{K}(\frac{1}{\sqrt{P_L}}-\frac{1}{\sqrt{P_H}})\tag{8}
$$
When $P_H\leq S_T$, the price is sliding upward past $P_H$, all of asset X is transformed into asset Y. Substituting into $(2)(4)$, Similarly we have:
$$
\Delta y =y_1-y_0=\sqrt K(\sqrt {P_H}- \sqrt{S_0})
$$
$$
y_1 = y_0 +\Delta y = \sqrt K(\sqrt P_H -\sqrt P_L)\tag{9}
$$
Substituting $(4)(8)(9)$ into $(6)$ we get:
$$
V_T =
\begin{cases}
   S_T\sqrt{K}(\frac{1}{\sqrt{P_L}}-\frac{1}{\sqrt{P_H}})  & S_T\leq P_L \\
   \sqrt{KS_T}(2-\sqrt{\frac{P_L}{S_T}}-\sqrt{\frac{S_T}{P_H}})
   & P_L < S_T< P_H \\
\sqrt K(\sqrt P_H -\sqrt P_L) & P_H\leq S_T
\end{cases} \tag{10}
$$

Subtituting$(6)$ into $(10)$ to eliminate $\sqrt{K}$. 
As a LP, given $V_0$, the asset value at $T_0$ are shown below:

$$
V_T =
\begin{cases}
   S_T\frac{V_0(\frac{1}{\sqrt{P_L}}-\frac{1}{\sqrt{P_H}})  }{\sqrt{S_0}(2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}})}
   & S_T\leq P_L \\

   \frac{V_0\sqrt{S_T}(2-\sqrt{\frac{P_L}{S_T}}-\sqrt{\frac{S_T}{P_H}}) }{\sqrt{S_0}(2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}})}
   & P_L < S_T< P_H \\

   \frac{V_0(\sqrt P_H -\sqrt P_L) }{\sqrt{S_0}(2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}})}
   & P_H\leq S_T
\end{cases} \tag{11}
$$

See figure in: <https://www.desmos.com/calculator/ugi0mwotsc>


### The free asset value $V_F$
Then considering the opportunity cost up to $T_{1}$ moment, i.e. we do not participate in providing liquidity, but hold a fixed number of assets in the portfolio. our asset value is:
$$
V_{F}=y_{0}+S_T\cdot x_{0}
$$

### The impermanent loss
Process the molecules first use equation$(10)$.
$$
\begin {split}
V_T-V_F=\begin{cases}
   \sqrt{KS_T}(\sqrt{\frac{P_L}{S_T}}+\sqrt{\frac{S_T}{P_L}}-\sqrt{\frac{S_T}{S_0}}-\sqrt{\frac{S_0}{S_T}})
   & S_T\leq P_L \\

   \sqrt{KS_T}(2-\sqrt{\frac{S_0}{S_T}}-\sqrt{\frac{S_T}{S_0}})
   & P_L < S_T< P_H \\

   \sqrt{KS_T}(\sqrt{\frac{P_H}{S_T}}+\sqrt{\frac{S_T}{P_H}}-\sqrt{\frac{S_T}{S_0}}-\sqrt{\frac{S_0}{S_T}})
   & P_H\leq S_T
\end{cases}
\end {split}
$$
We get:
$$
\begin {split}
Loss=\begin{cases}
   \sqrt{\frac{S_T}{S_0}}\cdot\frac{\sqrt{\frac{P_L}{S_T}}+\sqrt{\frac{S_T}{P_L}}-\sqrt{\frac{S_T}{S_0}}-\sqrt{\frac{S_0}{S_T}}}{2-\sqrt{\frac{S_0}{P_H}}-\sqrt{\frac{P_L}{S_0}}}
   & S_T\leq P_L \\

   \sqrt{\frac{S_T}{S_0}}\cdot\frac{2-\sqrt{\frac{S_0}{S_T}}-\sqrt{\frac{S_T}{S_0}}}{2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}}}
   & P_L < S_T< P_H \\

   \sqrt{\frac{S_T}{S_0}}\cdot\frac{\sqrt{\frac{P_H}{S_T}}+\sqrt{\frac{S_T}{P_H}}-\sqrt{\frac{S_T}{S_0}}-\sqrt{\frac{S_0}{S_T}}}{2-\sqrt{\frac{P_L}{S_0}}-\sqrt{\frac{S_0}{P_H}}}
   & P_H\leq S_T
\end{cases}\tag{11}
\end {split}
$$
Let $R=\sqrt{\frac{S_T}{S_0}}$. We assume that the geometric mean of our market making range $P_L$ and $P_H$ is $S_0$. Then we set $a=\sqrt{\frac{P_L}{S_0}}=\sqrt{\frac{S_0}{P_H}}$. tte larger a, the wider the market making range. We have:
$$
\begin {split}
Loss=\begin{cases}
   \frac{R^2-a}{2a}
   & S_T\leq P_L \\

   \frac{(R-1)^2}{2(a-1)}
   & P_L < S_T< P_H \\

   \frac{1-aR^2}{2a}
   & P_H\leq S_T
\end{cases}\tag{12}
\end {split}
$$
See Figure of Loss in:
<https://www.desmos.com/calculator/ivd2d0rems>


