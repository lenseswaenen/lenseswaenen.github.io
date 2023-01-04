---
usemathjax: true
layout: post
title: "MegaTomBike 2022 Aftermath"
subtitle: "and the new Scipy MILP solver"
date: 2023-01-04
background: /img/posts/megatombike/simon-connellan-TQZlo6cC4s0-unsplash.jpg
---

Like [the 2021 post](/2021/11/11/megatombike2021-aftermath.html), we compute what would have been the optimal team for Megatombike 2022, a local fantasy cycling competition.

This year, I didn't get round to making a post on how I assembled my team. For the better, in hindsight, as it performed terribly. I hoped to get a team of overachievers as a combination of 
- up-and-coming talents who are still undervalued (like De Lie, Pidcock and Evenepoel)
- former big riders who had a poor season last year (Froome, Dumoulin, ...)

Especially the latter turned out to be a bad idea. Also, out of the expensive, high scoring riders, I kicked out Van Aert to make room for Alaphilippe at the last moment...

Anyways, on to more succesful teams. The winner of this year, once again, was *Hala Die Mannschaft* with a score of 2435, this year by a margin of almost 100 points over the second team. Very impressive!

| Rider                | Points |
|----------------------|--------|
| LÓPEZ Miguel Ángel     | 74  |
| JAKOBSEN Fabio         | 115 |
| VAN AERT Wout          | 401 |
| CAMPENAERTS Victor     | 74  |
| VINE Jay               | 74  |
| FOSS Tobias            | 45  |
| POGAČAR Tadej          | 473 |
| EVENEPOEL Remco        | 394 |
| VALVERDE Alejandro     | 176 |
| MOHORIČ Matej          | 125 |
| LAMPAERT Yves          | 22  |
| MARTÍNEZ Daniel Felipe | 191 |
| SHEFFIELD Magnus       | 80  |
| VADER Milan            | 0   |
| DE LIE Arnaud          | 191 |

Note that the overall scores this year are about 60% higher than last year, because of an extended calender incorporating many smaller races

# The optimal team
I have copied some math and code from previous year to show the mathematical model and how to solve that using the combination of the Pyomo optimization modelling languange along with open-source MILP (mixed integer linear programming) solver CBC.
However, we have a new kid on the block as well, stay tuned!

## Mathematical model
Let us repeat the mathematical model once more:

$$
\begin{align*}
\text{maximize}_{x_i} \quad & \sum_{i=1}^n p_i x_i \\
\text{subject to} \quad & \sum_{i=1}^n c_i x_i \leq C \\
& \sum_{i=1}^n x_i = N \\
& x_i \in \{0,1\}
\end{align*}
$$

where $p_i$ is the number of points collected by a rider, $c_i$ is the cost of a rider, $C$ is the total team cost limit (= 1.750.000), and $N$ is the total number of riders (= 15). $x_i$ is a binary variable which either selects or does not select rider $i$.

So, we are maximizing the total number of points, given all the constraints.

Constraint $$\sum_{i=1}^n x_i = N = 15$$ is how this problem differs from the standard 0-1 knapsack problem.

## Data


```python
%matplotlib notebook
```


```python
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
```


```python
df = pd.read_csv(r"C:\Users\swaenenl\OneDrive - Sioux Group B.V\Documents\Personal\website\notebooks\megatombike\2022_results\2022_results\riders.csv", delimiter=';')

df.head(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>price</th>
      <th>points</th>
      <th>id.1</th>
      <th>name.1</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>527</td>
      <td>POGAČAR Tadej</td>
      <td>437000</td>
      <td>473</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>322</td>
      <td>VAN AERT Wout</td>
      <td>320000</td>
      <td>401</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>395</td>
      <td>EVENEPOEL Remco</td>
      <td>152500</td>
      <td>394</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>328</td>
      <td>VINGEGAARD Jonas</td>
      <td>144000</td>
      <td>247</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>113</td>
      <td>VLASOV Aleksandr</td>
      <td>174000</td>
      <td>213</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>315</td>
      <td>LAPORTE Christophe</td>
      <td>96500</td>
      <td>213</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>96</td>
      <td>HIGUITA Sergio</td>
      <td>87500</td>
      <td>195</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>561</td>
      <td>VAN DER POEL Mathieu</td>
      <td>260000</td>
      <td>195</td>
      <td>19</td>
      <td>Alpecin-Fenix</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>219</td>
      <td>MARTÍNEZ Daniel Felipe</td>
      <td>75000</td>
      <td>191</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>335</td>
      <td>DE LIE Arnaud</td>
      <td>15000</td>
      <td>191</td>
      <td>12</td>
      <td>Lotto Soudal</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>189</td>
      <td>KÜNG Stefan</td>
      <td>121500</td>
      <td>187</td>
      <td>7</td>
      <td>Groupama - FDJ</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>61</td>
      <td>BILBAO Pello</td>
      <td>102500</td>
      <td>187</td>
      <td>3</td>
      <td>Bahrain - Victorious</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>548</td>
      <td>MERLIER Tim</td>
      <td>144000</td>
      <td>181</td>
      <td>19</td>
      <td>Alpecin-Fenix</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>383</td>
      <td>VALVERDE Alejandro</td>
      <td>159500</td>
      <td>176</td>
      <td>13</td>
      <td>Movistar Team</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>434</td>
      <td>MATTHEWS Michael</td>
      <td>150000</td>
      <td>153</td>
      <td>15</td>
      <td>Team BikeExchange - Jayco</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



The csv file, once again delivered to me by organizer Bob, has the riders already sorted according to their points. The table does have some duplicates which we need to fix.
We see 2 Belgian riders in the top 3! I believe one of the reasons of some of my Megatombike successes around 2012 were due to me not having a bias towards Belgian riders unlike most MTB participants. Back then, that was an advantage. Nowadays, not so much!


```python
df = df.drop_duplicates(['name'])
```


```python
df.sort_values('points', ascending=False).head(15).sum()
```




    id                                                       4826
    name        POGAČAR TadejVAN AERT WoutEVENEPOEL RemcoVINGE...
    price                                                 2439000
    points                                                   3597
    id.1                                                      169
    name.1      UAE Team EmiratesJumbo-VismaQuick-Step Alpha V...
    selected                                                  7.0
    dtype: object



Summing the values and points of the best 15 riders gives a team that scores 3597 points, but is way too expensive (2.4 million). Still, this doesn't exceed the 1.75 million budget as much as last year (then it was 2.85 million). And in fact, starting from the top, we would be able to buy the 9 highest scoring riders, for a point total of 2522. Enough to beat out *Hala Die Mannschaft*!  


```python
df.sort_values('points', ascending=False).head(9).sum()
```




    id                                                       2876
    name        POGAČAR TadejVAN AERT WoutEVENEPOEL RemcoVINGE...
    price                                                 1746500
    points                                                   2522
    id.1                                                      100
    name.1      UAE Team EmiratesJumbo-VismaQuick-Step Alpha V...
    selected                                                  5.0
    dtype: object



## Pyomo + CBC solve


```python
import pyomo.environ as pyo
```


```python
points = {}
cost = {}
count = {}
dups = []
for i, el in df.iterrows():
    name = el['name']
    
    points[name] = el['points']
    cost[name] = el['price']
    count[name] = 1.
```


```python
cost_limit = 1750000
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=list(points.keys()))
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Objective(expr=sum(points[i]*M.x[i] for i in M.ITEMS), sense=pyo.maximize)
M.weight = pyo.Constraint(expr=sum(cost[i]*M.x[i] for i in M.ITEMS) <= cost_limit)
M.count = pyo.Constraint(expr=sum(count[i]*M.x[i] for i in M.ITEMS) == count_required)
```

Let's run this thing!


```python
opt = pyo.SolverFactory("cbc.exe")
results = opt.solve(M)
```


```python
print(results)
```

    
    Problem: 
    - Lower bound: -inf
      Upper bound: inf
      Number of objectives: 1
      Number of constraints: 2
      Number of variables: 963
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective -3263; 0 nodes, 19 iterations, 0.025 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.07400059700012207
    Solution: 
    - number of solutions: 0
      number of solutions displayed: 0
    
    

It never ceases to amaze how fast MILP solvers can solve (with guaranteed optimality) these NP-hard problems. If we take the point of view that the number of candidate solutions is dictated by choosing a *combination* of 15 elements out of 963, we can use the `scipy` implementation of $N$-choose-$k$ to get an impression of the size of the solution space and the NP-hardness of it:


```python
import scipy.special
```


```python
scipy.special.comb(963, 15)
```




    3.893159932009228e+32



The `results` structure output reports a computation time of ~65 milliseconds to find the optimal solution among $3.89 \times 10^{32}$ candidate solutions. Of course, it does so much faster than brute forcing its way through all those candidates.



```python
%%timeit
opt = pyo.SolverFactory("cbc.exe")
results = opt.solve(M)
```

    111 ms ± 412 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
    

If we do a custom timing of the solve, that computation time doubles to 120-130 milliseconds. CBC is included in Pyomo as an executable and the interfacing is file-system-based: Pyomo writes to files which get loaded by `cbc.exe` (which you have to put somewhere in the path, e.g. in the same directory as the Jupyter notebook I was using.

But you were interested in the optimal team, right?! Here goes:


```python
df['selected'] = [v.value for v in M.component_data_objects(pyo.Var)]
df[df['selected'] > 0.5].sort_values('points', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>price</th>
      <th>points</th>
      <th>id.1</th>
      <th>name.1</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>527</td>
      <td>POGAČAR Tadej</td>
      <td>437000</td>
      <td>473</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>322</td>
      <td>VAN AERT Wout</td>
      <td>320000</td>
      <td>401</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>395</td>
      <td>EVENEPOEL Remco</td>
      <td>152500</td>
      <td>394</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>328</td>
      <td>VINGEGAARD Jonas</td>
      <td>144000</td>
      <td>247</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>315</td>
      <td>LAPORTE Christophe</td>
      <td>96500</td>
      <td>213</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>96</td>
      <td>HIGUITA Sergio</td>
      <td>87500</td>
      <td>195</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>219</td>
      <td>MARTÍNEZ Daniel Felipe</td>
      <td>75000</td>
      <td>191</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>335</td>
      <td>DE LIE Arnaud</td>
      <td>15000</td>
      <td>191</td>
      <td>12</td>
      <td>Lotto Soudal</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>189</td>
      <td>KÜNG Stefan</td>
      <td>121500</td>
      <td>187</td>
      <td>7</td>
      <td>Groupama - FDJ</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>61</td>
      <td>BILBAO Pello</td>
      <td>102500</td>
      <td>187</td>
      <td>3</td>
      <td>Bahrain - Victorious</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>445</td>
      <td>ARENSMAN Thymen</td>
      <td>22000</td>
      <td>134</td>
      <td>16</td>
      <td>Team DSM</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>226</td>
      <td>RODRIGUEZ Carlos</td>
      <td>57500</td>
      <td>120</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>252</td>
      <td>MEINTJES Louis</td>
      <td>41500</td>
      <td>117</td>
      <td>9</td>
      <td>Intermarché - Wanty - Gobert Matériaux</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>507</td>
      <td>AYUSO Juan</td>
      <td>40000</td>
      <td>110</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>41</th>
      <td>247</td>
      <td>HERMANS Quinten</td>
      <td>30000</td>
      <td>103</td>
      <td>9</td>
      <td>Intermarché - Wanty - Gobert Matériaux</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df['selected'] > 0.5].sum()
```




    id                                                       4464
    name        POGAČAR TadejVAN AERT WoutEVENEPOEL RemcoVINGE...
    price                                                 1742500
    points                                                   3263
    id.1                                                      159
    name.1      UAE Team EmiratesJumbo-VismaQuick-Step Alpha V...
    selected                                                 15.0
    dtype: object



This team would have scored a whopping 3263 points: more than 800 points more than the winner. Easier said than done, of course!

A little bit more analysis:
We saw before how the total price of the top 15 scorers doesn't exceed the budget as much as last year. Now we see that the optimal team has 9 out of the 11 highest scoring riders in it. Only numbers 4 and 7 are not selected: Vlasov and van der Poel.

These observations may indicate that riders are relatively cheap this year, and one can buy almost all of the highest scoring riders. Once you get to the point that you can buy all 15 highest scoring riders with the given budget, the *knapsack* aspect of the fantasy cycling competition becomes a bit moot, and it becomes *only* about predicting who will be the highest scoring riders. 

However, if one looks at the 15 most expensive riders this season, only 2 of them make it to the optimal team, and there is quite a discrepancy between rider prices and scores. In fact, the 15 most expensive riders together would have scored in total 2226 points, which would not even have won the 2022 MTB competition!

Anyways, I'm not sure whether this all means I should recommend the organisers to make the budgets tighter again, or not...


```python
df.sort_values(['price'], ascending=False).head(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>price</th>
      <th>points</th>
      <th>id.1</th>
      <th>name.1</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>527</td>
      <td>POGAČAR Tadej</td>
      <td>437000</td>
      <td>473</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>318</td>
      <td>ROGLIČ Primož</td>
      <td>377000</td>
      <td>148</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>322</td>
      <td>VAN AERT Wout</td>
      <td>320000</td>
      <td>401</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>147</th>
      <td>385</td>
      <td>ALAPHILIPPE Julian</td>
      <td>280000</td>
      <td>32</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>561</td>
      <td>VAN DER POEL Mathieu</td>
      <td>260000</td>
      <td>195</td>
      <td>19</td>
      <td>Alpecin-Fenix</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>505</td>
      <td>ALMEIDA João</td>
      <td>219000</td>
      <td>118</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>541</th>
      <td>208</td>
      <td>BERNAL Egan</td>
      <td>193000</td>
      <td>0</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>209</td>
      <td>CARAPAZ Richard</td>
      <td>182500</td>
      <td>133</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>92</th>
      <td>504</td>
      <td>ACKERMANN Pascal</td>
      <td>175500</td>
      <td>47</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>113</td>
      <td>VLASOV Aleksandr</td>
      <td>174000</td>
      <td>213</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>141</th>
      <td>64</td>
      <td>COLBRELLI Sonny</td>
      <td>161000</td>
      <td>33</td>
      <td>3</td>
      <td>Bahrain - Victorious</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>237</td>
      <td>YATES Adam</td>
      <td>160500</td>
      <td>135</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>383</td>
      <td>VALVERDE Alejandro</td>
      <td>159500</td>
      <td>176</td>
      <td>13</td>
      <td>Movistar Team</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>55</th>
      <td>185</td>
      <td>GAUDU David</td>
      <td>154000</td>
      <td>76</td>
      <td>7</td>
      <td>Groupama - FDJ</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>98</th>
      <td>297</td>
      <td>WOODS Michael</td>
      <td>153000</td>
      <td>46</td>
      <td>10</td>
      <td>Israel - Premier Tech</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.sort_values(['price'], ascending=False).head(15).sum()
```




    id                                                       4818
    name        POGAČAR TadejROGLIČ PrimožVAN AERT WoutALAPHIL...
    price                                                 3406000
    points                                                   2226
    id.1                                                      170
    name.1      UAE Team EmiratesJumbo-VismaJumbo-VismaQuick-S...
    selected                                                  0.0
    dtype: object



A final bit of analysis: In 2021, the highest team scored 1564 points, while the maximum achievable was 2223. The ratio of these two numbers is 70%. In 2022, we have a ratio of $2435/3263  \approx 75\%$. So there seems to be some consistency on how optimal you have to be to win MTB.

# Auxiliary
## Cheapest team that could have won

We can also turn the knapsack problem on its head: What is the cheapest team, that could have won (= total score of 2436 or more). In the code, an objective and constraint need to be swapped.


```python
cost_limit = 1750000
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=points.keys())
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Constraint(expr=sum(points[i]*M.x[i] for i in M.ITEMS) >= 2436+1)
M.weight = pyo.Objective(expr=sum(cost[i]*M.x[i] for i in M.ITEMS), sense=pyo.minimize)
M.count = pyo.Constraint(expr=sum(count[i]*M.x[i] for i in M.ITEMS) == count_required)

opt = pyo.SolverFactory("cbc.exe")
results = opt.solve(M)
print(results)
```

    
    Problem: 
    - Lower bound: -inf
      Upper bound: inf
      Number of objectives: 1
      Number of constraints: 2
      Number of variables: 963
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective 908000; 0 nodes, 50 iterations, 0.072 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.11903929710388184
    Solution: 
    - number of solutions: 0
      number of solutions displayed: 0
    
    


```python
df['selected'] = [v.value for v in M.component_data_objects(pyo.Var)]
df[df['selected'] > 0.5].sort_values('points', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>price</th>
      <th>points</th>
      <th>id.1</th>
      <th>name.1</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>395</td>
      <td>EVENEPOEL Remco</td>
      <td>152500</td>
      <td>394</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>328</td>
      <td>VINGEGAARD Jonas</td>
      <td>144000</td>
      <td>247</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>315</td>
      <td>LAPORTE Christophe</td>
      <td>96500</td>
      <td>213</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>96</td>
      <td>HIGUITA Sergio</td>
      <td>87500</td>
      <td>195</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>219</td>
      <td>MARTÍNEZ Daniel Felipe</td>
      <td>75000</td>
      <td>191</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>335</td>
      <td>DE LIE Arnaud</td>
      <td>15000</td>
      <td>191</td>
      <td>12</td>
      <td>Lotto Soudal</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>61</td>
      <td>BILBAO Pello</td>
      <td>102500</td>
      <td>187</td>
      <td>3</td>
      <td>Bahrain - Victorious</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>445</td>
      <td>ARENSMAN Thymen</td>
      <td>22000</td>
      <td>134</td>
      <td>16</td>
      <td>Team DSM</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>226</td>
      <td>RODRIGUEZ Carlos</td>
      <td>57500</td>
      <td>120</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>252</td>
      <td>MEINTJES Louis</td>
      <td>41500</td>
      <td>117</td>
      <td>9</td>
      <td>Intermarché - Wanty - Gobert Matériaux</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>507</td>
      <td>AYUSO Juan</td>
      <td>40000</td>
      <td>110</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>312</td>
      <td>KOOIJ Olav</td>
      <td>41500</td>
      <td>109</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>41</th>
      <td>247</td>
      <td>HERMANS Quinten</td>
      <td>30000</td>
      <td>103</td>
      <td>9</td>
      <td>Intermarché - Wanty - Gobert Matériaux</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>53</th>
      <td>228</td>
      <td>SHEFFIELD Magnus</td>
      <td>2500</td>
      <td>80</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>78</th>
      <td>980</td>
      <td>POZZOVIVO Domenico</td>
      <td>0</td>
      <td>55</td>
      <td>9</td>
      <td>Intermarché - Wanty - Gobert Matériaux</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df['selected'] > 0.5].sum()
```




    id                                                       4946
    name        EVENEPOEL RemcoVINGEGAARD JonasLAPORTE Christo...
    price                                                  908000
    points                                                   2446
    id.1                                                      151
    name.1      Quick-Step Alpha Vinyl TeamJumbo-VismaJumbo-Vi...
    selected                                                 15.0
    dtype: object



The cheapest team that could have won, costs 908.000 euro, which means that only about 50 percent of the budget is used! This team shares 12 riders with the optimal team, and swaps out 
- Pogacar
- Van Aert
- Kung 

for 
- Kooij
- Sheffield
- Pozzovivo

This corroborates the discrepancy between prices and scores this year: Many riders that scored high were quite cheap. And vice versa many expensive riders scored below par. Only Pogacar and Van Aert lived up to the expectations.

## Worst team
Finally, the worst team is a trivial problem: just select 15 (cheap?) riders with no points. One way to have a more interesting worst team, could be to set a lower limit on the team cost, e.g. 1.745.000 euro. 


```python
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=points.keys())
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Objective(expr=sum(points[i]*M.x[i] for i in M.ITEMS), sense=pyo.minimize)
M.weight = pyo.Constraint(expr=sum(cost[i]*M.x[i] for i in M.ITEMS) >= cost_limit - 5000)
M.count = pyo.Constraint(expr=sum(count[i]*M.x[i] for i in M.ITEMS) == count_required)

opt = pyo.SolverFactory("cbc.exe")
results = opt.solve(M)
print(results)
```

    
    Problem: 
    - Lower bound: -inf
      Upper bound: inf
      Number of objectives: 1
      Number of constraints: 2
      Number of variables: 963
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective 70; 0 nodes, 19 iterations, 0.031 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.0669410228729248
    Solution: 
    - number of solutions: 0
      number of solutions displayed: 0
    
    


```python
df['selected'] = [v.value for v in M.component_data_objects(pyo.Var)]
df[df['selected'] > 0.5].sort_values('points', ascending=False)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>price</th>
      <th>points</th>
      <th>id.1</th>
      <th>name.1</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>147</th>
      <td>385</td>
      <td>ALAPHILIPPE Julian</td>
      <td>280000</td>
      <td>32</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>232</th>
      <td>235</td>
      <td>VIVIANI Elia</td>
      <td>146500</td>
      <td>14</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>245</th>
      <td>396</td>
      <td>HONORÉ Mikkel Frølich</td>
      <td>115000</td>
      <td>11</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>308</th>
      <td>99</td>
      <td>KELDERMAN Wilco</td>
      <td>109500</td>
      <td>6</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>348</th>
      <td>109</td>
      <td>SCHACHMANN Maximilian</td>
      <td>137500</td>
      <td>4</td>
      <td>4</td>
      <td>BORA - hansgrohe</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>391</th>
      <td>401</td>
      <td>MASNADA Fausto</td>
      <td>78500</td>
      <td>1</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>413</th>
      <td>305</td>
      <td>DUMOULIN Tom</td>
      <td>127500</td>
      <td>1</td>
      <td>11</td>
      <td>Jumbo-Visma</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>419</th>
      <td>17</td>
      <td>PARET-PEINTRE Aurélien</td>
      <td>88000</td>
      <td>1</td>
      <td>1</td>
      <td>AG2R Citroën Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>541</th>
      <td>208</td>
      <td>BERNAL Egan</td>
      <td>193000</td>
      <td>0</td>
      <td>8</td>
      <td>INEOS Grenadiers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>542</th>
      <td>508</td>
      <td>BENNETT George</td>
      <td>86500</td>
      <td>0</td>
      <td>18</td>
      <td>UAE Team Emirates</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>544</th>
      <td>784</td>
      <td>ZAKARIN Ilnur</td>
      <td>71500</td>
      <td>0</td>
      <td>29</td>
      <td>Gazprom - RusVelo</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>545</th>
      <td>78</td>
      <td>POELS Wout</td>
      <td>80000</td>
      <td>0</td>
      <td>3</td>
      <td>Bahrain - Victorious</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>546</th>
      <td>389</td>
      <td>CATTANEO Mattia</td>
      <td>69500</td>
      <td>0</td>
      <td>14</td>
      <td>Quick-Step Alpha Vinyl Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>548</th>
      <td>47</td>
      <td>MOSCON Gianni</td>
      <td>85000</td>
      <td>0</td>
      <td>2</td>
      <td>Astana Qazaqstan Team</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>605</th>
      <td>167</td>
      <td>PADUN Mark</td>
      <td>80000</td>
      <td>0</td>
      <td>6</td>
      <td>EF Education-EasyPost</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df['selected'] > 0.5].sum()
```




    id                                                       3902
    name        ALAPHILIPPE JulianVIVIANI EliaHONORÉ Mikkel Fr...
    price                                                 1759500
    points                                                     71
    id.1                                                      142
    name.1      Quick-Step Alpha Vinyl TeamINEOS GrenadiersQui...
    selected                                                 15.0
    dtype: object



Last year, we could find a team that used all the money and scored no points. This year, if we spent nearly all the money, we're still getting about 70 points. Julien Alaphilippe and Tom Dumoulin are two of the worst riders one could have chosen... which I did...

# Scipy solution
This final section is for those interested in the mathematical optimization aspect.

I feel like there is quite a lot of movement in the MILP solver landscape. In the land of commercial solvers, some new, mostly Chinese, solvers are speeding their ways up to the tops of the [benchmarks](http://plato.asu.edu/bench.html). And some other contenders are pulling out of the benchmarks due to some Gurobi scandal. The developers of the Chinese COPT seem to be confident that they will overtake Gurobi in the next years: https://www.youtube.com/watch?v=iqiBXoJQVD8&t=258s  

In open source land, [HiGHs](https://highs.dev/) is the big newcomer. A major development with respect to last years post, is that in the summer of 2022, with Scipy version 1.9.0, the `scipy.optimize` submodule got extended with a `milp` routine, which runs HiGHs in the backend. I think this is quite a big deal. Scipy is a much more common Python library/dependency than Pyomo. For example, the Anaconda conda-forge channel has (at the time of writing) 600k installs for Pyomo, while Scipy has 31M, which is 50 times more! Moreover you don't need to install an open source MILP solver like CBC separately. This makes MILP solvers much more broadly accessible. The main current downside is that the interface is much more barebones, and you don't get all the nice modeling tools (named variables etc.) that you get with Pyomo.

Below, we work out the MTB knapsack problem with `scipy.optimize.milp`. This is really easy to do, as the Scipy documentation already has a knapsack example: https://scipy.github.io/devdocs/tutorial/optimize.html#knapsack-problem-example
All we need to add is the constraint that the number of riders needs to equal 15.


```python
import scipy.optimize as sciopt
```


```python
scipy.__version__
```




    '1.9.3'




```python
points_arr = np.array(list(points.values()))
cost_arr = np.array(list(cost.values()))
count_arr = np.array(list(count.values()))
```


```python
bounds = sciopt.Bounds(0, 1)  # 0 <= x_i <= 1
integrality = np.full_like(points_arr, True)  # x_i are integers

cost_constraints = sciopt.LinearConstraint(A=cost_arr, lb=0, ub=cost_limit)
count_constraints = sciopt.LinearConstraint(A=count_arr, lb=count_required, ub=count_required)

res = sciopt.milp(c=-points_arr, constraints=[cost_constraints, count_constraints],
           integrality=integrality, bounds=bounds)
```


```python
res.fun
```




    -3263.0




```python
sel = np.array(df['selected'])
np.max(np.abs(res.x - sel))
```

The solver finds the exact same solution with the same maximum point total of 3263.


```python
%%timeit
res = sciopt.milp(c=-points_arr, constraints=[cost_constraints, count_constraints],
           integrality=integrality, bounds=bounds)
```

    20.9 ms ± 716 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
    

If we benchmark it, we see that on this problem Scipy+HiGHs is 3 - 6 times faster than the Pyomo+CBC combination (depending on which timing number you take)! Part of it is probably due to the solver being interfaced as a normal library rather than a seperate executable running in a different process.

# SCIP
Finally, I'd like to report some related news on the [SCIP](https://www.scipopt.org/) MILP solver, which has always dwelled a bit in between open source and commercial solvers (both in license and in benchmark results). In November 2022, SCIP switched to the fully open source Apache 2.0 license. SCIP can be used from Python through the [PySCIPOpt](https://github.com/scipopt/PySCIPOpt) package. This has 130k downloads through conda-forge so seems the least popular option at the moment, though that may change due to the recent license changes. I haven't gotten round to giving it a go. I wouldn't be surprised if it turns out to be the fastest open source solver on this knapsack problem.


-----

## Related posts
- [MegaTomBike 2022 - aftermath](/2021/11/11/megatombike2021-aftermath.html)
- [MegaTomBike 2021 - team creation](/2021/03/27/megatombike2021.html)


## Want to leave a comment?
Very interested in your comments but still figuring out the most suited approach to this. For now, feel free to send me an [email](mailto:lenseswaenen@gmail.com).