---
usemathjax: true
layout: post
title: "MegaTomBike 2021"
subtitle: "Aftermath"
date: 2021-11-11
background: /img/posts/megatombike/simon-connellan-TQZlo6cC4s0-unsplash.jpg
---
Well, that was a disappointing season of MegaTomBike...

This post is the successor to [this](/2021/03/27/megatombike2021.html) one, posted end of March 2021, right when the deadline for team submission was for the 2021 edition of MegaTomBike. MegaTomBike is a local fantasy cycling competition. The team I had submitted was generated through a mathematical optimization approach, namely as a type of knapsack problem. A mathematical approach had proven succesful for me in previous competition, but this year alas...

The last race on the calendar was on October 9th. This post contains some after*math*.
My team, *Poeske Scherens Fanclub* finished 62nd out of 87 teams, with a score of 1089 points. The final winning team was *Hala Die Mannschaft* with a score of 1564, winning out by a mere 4 points over the second placed team *Bekke - Algemeen*. The third team lagged quite a bit further behind.
The composition of the winning team is:

| Rider                | Points |
|----------------------|--------|
| Geraint Thomas       | 121    |
| Egan Bernal          | 132    |
| Tadej Pogačar        | 315    |
| Mathieu van der Poel | 160    |
| Wout van Aert        | 316    |
| Julian Alaphilippe   | 226    |
| Quinn Simmons        | 1      |
| Thomas Pidcock       | 86     |
| Mark Cavendish       | 54     |
| Remco Evenepoel      | 83     |
| Tim Merlier          | 47     |
| Gianni Vermeersch    | 19     |
| Arjen Livyns         | 0      |
| Mauri Vansevenant    | 3      |
| Heinrich Haussler    | 1      |

# Why did I not win?
The setup of MegaTomBike is to create a team of exactly 15 riders, with a cost not more than 1.750.000 euro. The team collects points during the season; the team with the most points wins.

The team I created was optimal in some mathematical sense, namely as the solution of a knapsack-type problem, where I searched for a team of 15 riders which satisfies the cost constraint, and maximizes the total number of points. The challenge of course is that the number of points collected by the end of the season is unknown, so one needs to make a prediction / use a proxy. The proxy I used were the UCI points from the season before. Now, we can conclude this was quite a poor proxy.

As was already discussed in the final section of the preceding blog post, the costs that the organizers of MegaTomBike have assigned to riders is not random, but actually also some kind rider level determined from cycling stats from the past few seasons. This in itself is a reasonable predictor, much like the weather of today is a pretty good predictor for the weather of tomorrow. Setting the costs like this, is exactly how a competition like MegaTomBike is made interesting. In the end, my way of team creation was kind of maximizing the disparity between both predictors.

# Highest scoring teams
Now the season is over, we can revisit the knapsack approach and use the actual points to determine what the optimal team for this season would have been, and see how close the MegaTomBike winner managed to get. Many thanks to organizer Bob for sharing a csv file with final points and costs to facilitate this analysis (and not having to do all kinds of string matching like in the previous blog post).

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
import re
```


```python
df = pd.read_csv(r"lense_riders.txt", delimiter=';')

df['name_'] = df['search_name'].map(lambda name: re.sub('\W+',' ', name)) #Fix Ben O'Connor error...

df = df[['name_', 'value', 'points']]
```

Let us look at the riders who scored the most points. Fellow countryman Wout van Aert did best! On the UCI ranking, he finished second after Tadej Pogacar, but obviously the MegaTomBike ranking is much more prestigious.


```python
df.sort_values('points', ascending=False).head(15)
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>465</th>
      <td>Wout van Aert</td>
      <td>242000</td>
      <td>316</td>
    </tr>
    <tr>
      <th>549</th>
      <td>Tadej Pogacar</td>
      <td>324000</td>
      <td>315</td>
    </tr>
    <tr>
      <th>461</th>
      <td>Primoz Roglic</td>
      <td>400000</td>
      <td>256</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Julian Alaphilippe</td>
      <td>291000</td>
      <td>226</td>
    </tr>
    <tr>
      <th>240</th>
      <td>Richard Carapaz</td>
      <td>156000</td>
      <td>195</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Sonny Colbrelli</td>
      <td>140500</td>
      <td>160</td>
    </tr>
    <tr>
      <th>582</th>
      <td>Mathieu van der Poel</td>
      <td>249000</td>
      <td>160</td>
    </tr>
    <tr>
      <th>267</th>
      <td>Adam Yates</td>
      <td>143000</td>
      <td>149</td>
    </tr>
    <tr>
      <th>326</th>
      <td>Michael Woods</td>
      <td>145500</td>
      <td>142</td>
    </tr>
    <tr>
      <th>382</th>
      <td>Alejandro Valverde</td>
      <td>198500</td>
      <td>136</td>
    </tr>
    <tr>
      <th>239</th>
      <td>Egan Bernal</td>
      <td>174500</td>
      <td>132</td>
    </tr>
    <tr>
      <th>523</th>
      <td>Jasper Stuyven</td>
      <td>140000</td>
      <td>130</td>
    </tr>
    <tr>
      <th>468</th>
      <td>Jonas Vingegaard</td>
      <td>40000</td>
      <td>126</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Matej Mohoric</td>
      <td>80500</td>
      <td>126</td>
    </tr>
    <tr>
      <th>256</th>
      <td>Richie Porte</td>
      <td>135000</td>
      <td>124</td>
    </tr>
  </tbody>
</table>
</div>



Summing the values and points of the best 15 riders gives that scores 2693 points, but is way too expensive (2.85 million). But now we do know an upperbound of the maximum knapsack score: 2693 points.


```python
df.sort_values('points', ascending=False).head(15).sum()
```




    name_       Wout van AertTadej PogacarPrimoz RoglicJulian ...
    value                                                 2859500
    points                                                   2693
    selected                                                  8.0
    dtype: object



## Solving the knapsack problem
Like before, we use [Pyomo](http://www.pyomo.org/) as the optimization modeling environment.


```python
import pyomo.environ as pyo
```


```python
points = {}
cost = {}
count = {}
for i, el in df.iterrows():
    points[el['name_']] = el['points']
    cost[el['name_']] = el['value']
    count[el['name_']] = 1.
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
      Number of variables: 979
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective -2223; 4 nodes, 90 iterations, 0.054 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.1037588119506836
    Solution: 
    - number of solutions: 0
      number of solutions displayed: 0
    
    

We got our solution with 979 variables in the blink of an eye: 0.054 seconds. Even though the knapsack problem is NP-hard, it is a common misconception that reasonably sized problems can not be solved to optimality pretty fast. More famous is the Traveling Salesman Problem of which the [Traveling Sam Problem](https://github.com/Forceflow/Ambiance_TSP/blob/main/README.md) formulated by Nerdland is a fun instance with 24 variables. Even though there are $24! \approx 6.2 \times 10^{23}$ solutions, which is way too big to be brute forced, good algorithms exists which can solve a 24 city problem to (provable) optimality in less than a second. In fact, the current world record for a TSP having been solved to proven optimality stands at [57,912 stops](https://www.math.uwaterloo.ca/tsp/nl/index.html) and is a Dutch cycling route along national monuments. 

So what does NP-hardness then mean? NP-hardness is a statement about asymptotic complexity behavior for the number of cities $N \to \infty$, and that in this limit, the computation times grow exponentially fast (kind of). But it might be that fast growing behavior only appears beyond $N > 10^{100}$. Not saying that this is the case for TSPs or knapsack problems, but the statement about reasonably sized problems being solvable in reasonable time stands.

Apologies for this intermezzo, as you are obviously more interested in the optimal team, so here it is:


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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>465</th>
      <td>Wout van Aert</td>
      <td>242000</td>
      <td>316</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>549</th>
      <td>Tadej Pogacar</td>
      <td>324000</td>
      <td>315</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Julian Alaphilippe</td>
      <td>291000</td>
      <td>226</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>240</th>
      <td>Richard Carapaz</td>
      <td>156000</td>
      <td>195</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Sonny Colbrelli</td>
      <td>140500</td>
      <td>160</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>267</th>
      <td>Adam Yates</td>
      <td>143000</td>
      <td>149</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Matej Mohoric</td>
      <td>80500</td>
      <td>126</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>468</th>
      <td>Jonas Vingegaard</td>
      <td>40000</td>
      <td>126</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Kasper Asgreen</td>
      <td>114500</td>
      <td>118</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>265</th>
      <td>Dylan van Baarle</td>
      <td>59000</td>
      <td>103</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Ben O Connor</td>
      <td>34000</td>
      <td>98</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>255</th>
      <td>Thomas Pidcock</td>
      <td>10500</td>
      <td>86</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>66</th>
      <td>Damiano Caruso</td>
      <td>70000</td>
      <td>81</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>199</th>
      <td>Neilson Powless</td>
      <td>29000</td>
      <td>70</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Mark Cavendish</td>
      <td>2500</td>
      <td>54</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df['selected'] > 0.5].sort_values('points', ascending=False).sum()
```




    name_       Wout van AertTadej PogacarJulian AlaphilippeRi...
    value                                                 1736500
    points                                                   2223
    selected                                                 15.0
    dtype: object



This team would have made 2223 points, which is 659 points more than the current winner!

Many of the big names and big scorers are selected. The highest scorers which are not selected are interestingly Primoz Roglic and Mathieu van der Poel.


```python
df.sort_values('points', ascending=False).head(10)
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>465</th>
      <td>Wout van Aert</td>
      <td>242000</td>
      <td>316</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>549</th>
      <td>Tadej Pogacar</td>
      <td>324000</td>
      <td>315</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>461</th>
      <td>Primoz Roglic</td>
      <td>400000</td>
      <td>256</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Julian Alaphilippe</td>
      <td>291000</td>
      <td>226</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>240</th>
      <td>Richard Carapaz</td>
      <td>156000</td>
      <td>195</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Sonny Colbrelli</td>
      <td>140500</td>
      <td>160</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>582</th>
      <td>Mathieu van der Poel</td>
      <td>249000</td>
      <td>160</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>267</th>
      <td>Adam Yates</td>
      <td>143000</td>
      <td>149</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>326</th>
      <td>Michael Woods</td>
      <td>145500</td>
      <td>142</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>382</th>
      <td>Alejandro Valverde</td>
      <td>198500</td>
      <td>136</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['efficiency'] = 1000*df['points']/df['value']
```


```python
df[df['selected']>0].sort_values('efficiency', ascending=False)
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>156</th>
      <td>Mark Cavendish</td>
      <td>2500</td>
      <td>54</td>
      <td>1.0</td>
      <td>21.600000</td>
    </tr>
    <tr>
      <th>255</th>
      <td>Thomas Pidcock</td>
      <td>10500</td>
      <td>86</td>
      <td>1.0</td>
      <td>8.190476</td>
    </tr>
    <tr>
      <th>468</th>
      <td>Jonas Vingegaard</td>
      <td>40000</td>
      <td>126</td>
      <td>1.0</td>
      <td>3.150000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Ben O Connor</td>
      <td>34000</td>
      <td>98</td>
      <td>1.0</td>
      <td>2.882353</td>
    </tr>
    <tr>
      <th>199</th>
      <td>Neilson Powless</td>
      <td>29000</td>
      <td>70</td>
      <td>1.0</td>
      <td>2.413793</td>
    </tr>
    <tr>
      <th>265</th>
      <td>Dylan van Baarle</td>
      <td>59000</td>
      <td>103</td>
      <td>1.0</td>
      <td>1.745763</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Matej Mohoric</td>
      <td>80500</td>
      <td>126</td>
      <td>1.0</td>
      <td>1.565217</td>
    </tr>
    <tr>
      <th>465</th>
      <td>Wout van Aert</td>
      <td>242000</td>
      <td>316</td>
      <td>1.0</td>
      <td>1.305785</td>
    </tr>
    <tr>
      <th>240</th>
      <td>Richard Carapaz</td>
      <td>156000</td>
      <td>195</td>
      <td>1.0</td>
      <td>1.250000</td>
    </tr>
    <tr>
      <th>66</th>
      <td>Damiano Caruso</td>
      <td>70000</td>
      <td>81</td>
      <td>1.0</td>
      <td>1.157143</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Sonny Colbrelli</td>
      <td>140500</td>
      <td>160</td>
      <td>1.0</td>
      <td>1.138790</td>
    </tr>
    <tr>
      <th>267</th>
      <td>Adam Yates</td>
      <td>143000</td>
      <td>149</td>
      <td>1.0</td>
      <td>1.041958</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Kasper Asgreen</td>
      <td>114500</td>
      <td>118</td>
      <td>1.0</td>
      <td>1.030568</td>
    </tr>
    <tr>
      <th>549</th>
      <td>Tadej Pogacar</td>
      <td>324000</td>
      <td>315</td>
      <td>1.0</td>
      <td>0.972222</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Julian Alaphilippe</td>
      <td>291000</td>
      <td>226</td>
      <td>1.0</td>
      <td>0.776632</td>
    </tr>
  </tbody>
</table>
</div>



### Popular riders
The following seven riders below proved to be among the most popular riders throughout the competition, and all followed through with excellent performances too. Five of them actually are in the optimal team too. Note by the way, how I closed my previous blog post with the observation that Mark Cavendish is a steal, but was not selected by my optimization due to him not having ridden and scored UCI points the year before. I decided not to swap him out for Dutch youngster Olav Kooij. Unfortunaly, Olav only scored 2 points, versus Cavendish' 54 points.


```python
popular = ['Wout van Aert', 'Mathieu van der Poel', 'Julian Alaphilippe',
           'Tadej Pogacar', 'Primoz Roglic',
           'Mark Cavendish', 'Thomas Pidcock']
```


```python
sum(points[p] for p in popular)
```




    1413




```python
sum(cost[p] for p in popular)
```




    1519000



A team consisting of all those riders would have already scored 1413 points, much better than I did, enough for a 15th spot in the ranking, and still leaving over 200.000 euro to spend. Let's see how far we could get with a team having all these riders. In the code, we add equality constraints in which we set the binary selection variable for those riders to 1.


```python
cost_limit = 1750000
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=points.keys())
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Objective(expr=sum(points[i]*M.x[i] for i in M.ITEMS), sense=pyo.maximize)
M.weight = pyo.Constraint(expr=sum(cost[i]*M.x[i] for i in M.ITEMS) <= cost_limit)
M.count = pyo.Constraint(expr=sum(count[i]*M.x[i] for i in M.ITEMS) == count_required)

M.popular = pyo.ConstraintList()
for p in popular:
    M.popular.add(M.x[p] == 1)

opt = pyo.SolverFactory("cbc.exe")

results = opt.solve(M)

print(results)
```

    
    Problem: 
    - Lower bound: -inf
      Upper bound: inf
      Number of objectives: 1
      Number of constraints: 9
      Number of variables: 979
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective -1957; 0 nodes, 22 iterations, 0.035 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.09378385543823242
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>465</th>
      <td>Wout van Aert</td>
      <td>242000</td>
      <td>316</td>
      <td>1.0</td>
      <td>1.305785</td>
    </tr>
    <tr>
      <th>549</th>
      <td>Tadej Pogacar</td>
      <td>324000</td>
      <td>315</td>
      <td>1.0</td>
      <td>0.972222</td>
    </tr>
    <tr>
      <th>461</th>
      <td>Primoz Roglic</td>
      <td>400000</td>
      <td>256</td>
      <td>1.0</td>
      <td>0.640000</td>
    </tr>
    <tr>
      <th>147</th>
      <td>Julian Alaphilippe</td>
      <td>291000</td>
      <td>226</td>
      <td>1.0</td>
      <td>0.776632</td>
    </tr>
    <tr>
      <th>582</th>
      <td>Mathieu van der Poel</td>
      <td>249000</td>
      <td>160</td>
      <td>1.0</td>
      <td>0.642570</td>
    </tr>
    <tr>
      <th>468</th>
      <td>Jonas Vingegaard</td>
      <td>40000</td>
      <td>126</td>
      <td>1.0</td>
      <td>3.150000</td>
    </tr>
    <tr>
      <th>265</th>
      <td>Dylan van Baarle</td>
      <td>59000</td>
      <td>103</td>
      <td>1.0</td>
      <td>1.745763</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Ben O Connor</td>
      <td>34000</td>
      <td>98</td>
      <td>1.0</td>
      <td>2.882353</td>
    </tr>
    <tr>
      <th>255</th>
      <td>Thomas Pidcock</td>
      <td>10500</td>
      <td>86</td>
      <td>1.0</td>
      <td>8.190476</td>
    </tr>
    <tr>
      <th>199</th>
      <td>Neilson Powless</td>
      <td>29000</td>
      <td>70</td>
      <td>1.0</td>
      <td>2.413793</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Mark Cavendish</td>
      <td>2500</td>
      <td>54</td>
      <td>1.0</td>
      <td>21.600000</td>
    </tr>
    <tr>
      <th>163</th>
      <td>Mikkel Frolich Honore</td>
      <td>25500</td>
      <td>53</td>
      <td>1.0</td>
      <td>2.078431</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Gino Mader</td>
      <td>18500</td>
      <td>46</td>
      <td>1.0</td>
      <td>2.486486</td>
    </tr>
    <tr>
      <th>976</th>
      <td>Gianni Marchand</td>
      <td>2500</td>
      <td>26</td>
      <td>1.0</td>
      <td>10.400000</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Ide Schelling</td>
      <td>14000</td>
      <td>22</td>
      <td>1.0</td>
      <td>1.571429</td>
    </tr>
  </tbody>
</table>
</div>



With these 7 popular riders, the maximum achievable score is 1957 which is very good, but also quite a drop from 2223 points.

### Without any of the popular riders
We can also ask the opposite question: What is the best team we could have created without any of these 7 popular riders. In the code, now the binary variables of these riders are put to 0.


```python
cost_limit = 1750000
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=points.keys())
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Objective(expr=sum(points[i]*M.x[i] for i in M.ITEMS), sense=pyo.maximize)
M.weight = pyo.Constraint(expr=sum(cost[i]*M.x[i] for i in M.ITEMS) <= cost_limit)
M.count = pyo.Constraint(expr=sum(count[i]*M.x[i] for i in M.ITEMS) == count_required)

M.popular = pyo.ConstraintList()
for p in popular:
    M.popular.add(M.x[p] == 0)

opt = pyo.SolverFactory("cbc.exe")
results = opt.solve(M)
print(results)
```

    
    Problem: 
    - Lower bound: -inf
      Upper bound: inf
      Number of objectives: 1
      Number of constraints: 9
      Number of variables: 979
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective -1940; 0 nodes, 6 iterations, 0.031 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.08577203750610352
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>240</th>
      <td>Richard Carapaz</td>
      <td>156000</td>
      <td>195</td>
      <td>1.0</td>
      <td>1.250000</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Sonny Colbrelli</td>
      <td>140500</td>
      <td>160</td>
      <td>1.0</td>
      <td>1.138790</td>
    </tr>
    <tr>
      <th>267</th>
      <td>Adam Yates</td>
      <td>143000</td>
      <td>149</td>
      <td>1.0</td>
      <td>1.041958</td>
    </tr>
    <tr>
      <th>326</th>
      <td>Michael Woods</td>
      <td>145500</td>
      <td>142</td>
      <td>1.0</td>
      <td>0.975945</td>
    </tr>
    <tr>
      <th>239</th>
      <td>Egan Bernal</td>
      <td>174500</td>
      <td>132</td>
      <td>1.0</td>
      <td>0.756447</td>
    </tr>
    <tr>
      <th>523</th>
      <td>Jasper Stuyven</td>
      <td>140000</td>
      <td>130</td>
      <td>1.0</td>
      <td>0.928571</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Matej Mohoric</td>
      <td>80500</td>
      <td>126</td>
      <td>1.0</td>
      <td>1.565217</td>
    </tr>
    <tr>
      <th>468</th>
      <td>Jonas Vingegaard</td>
      <td>40000</td>
      <td>126</td>
      <td>1.0</td>
      <td>3.150000</td>
    </tr>
    <tr>
      <th>256</th>
      <td>Richie Porte</td>
      <td>135000</td>
      <td>124</td>
      <td>1.0</td>
      <td>0.918519</td>
    </tr>
    <tr>
      <th>264</th>
      <td>Geraint Thomas</td>
      <td>133500</td>
      <td>121</td>
      <td>1.0</td>
      <td>0.906367</td>
    </tr>
    <tr>
      <th>215</th>
      <td>David Gaudu</td>
      <td>128500</td>
      <td>120</td>
      <td>1.0</td>
      <td>0.933852</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Kasper Asgreen</td>
      <td>114500</td>
      <td>118</td>
      <td>1.0</td>
      <td>1.030568</td>
    </tr>
    <tr>
      <th>265</th>
      <td>Dylan van Baarle</td>
      <td>59000</td>
      <td>103</td>
      <td>1.0</td>
      <td>1.745763</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Ben O Connor</td>
      <td>34000</td>
      <td>98</td>
      <td>1.0</td>
      <td>2.882353</td>
    </tr>
    <tr>
      <th>100</th>
      <td>Wilco Kelderman</td>
      <td>121000</td>
      <td>96</td>
      <td>1.0</td>
      <td>0.793388</td>
    </tr>
  </tbody>
</table>
</div>



The maximum achievable score then is 1940, which is slightly less than the best team having all those riders. 

# Cheapest team that could have won

We can also turn the knapsack problem on its head: What is the cheapest team, that could have won (= total score of 1564 or more). In the code, the objective and the cost constraint need to be interchanged.


```python
cost_limit = 1750000
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=points.keys())
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Constraint(expr=sum(points[i]*M.x[i] for i in M.ITEMS) >= 1564+1)
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
      Number of variables: 979
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective 936500; 2 nodes, 47 iterations, 0.125 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.17966532707214355
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>465</th>
      <td>Wout van Aert</td>
      <td>242000</td>
      <td>316</td>
      <td>1.0</td>
      <td>1.305785</td>
    </tr>
    <tr>
      <th>240</th>
      <td>Richard Carapaz</td>
      <td>156000</td>
      <td>195</td>
      <td>1.0</td>
      <td>1.250000</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Sonny Colbrelli</td>
      <td>140500</td>
      <td>160</td>
      <td>1.0</td>
      <td>1.138790</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Matej Mohoric</td>
      <td>80500</td>
      <td>126</td>
      <td>1.0</td>
      <td>1.565217</td>
    </tr>
    <tr>
      <th>468</th>
      <td>Jonas Vingegaard</td>
      <td>40000</td>
      <td>126</td>
      <td>1.0</td>
      <td>3.150000</td>
    </tr>
    <tr>
      <th>265</th>
      <td>Dylan van Baarle</td>
      <td>59000</td>
      <td>103</td>
      <td>1.0</td>
      <td>1.745763</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Ben O Connor</td>
      <td>34000</td>
      <td>98</td>
      <td>1.0</td>
      <td>2.882353</td>
    </tr>
    <tr>
      <th>202</th>
      <td>Rigoberto Uran</td>
      <td>93500</td>
      <td>90</td>
      <td>1.0</td>
      <td>0.962567</td>
    </tr>
    <tr>
      <th>255</th>
      <td>Thomas Pidcock</td>
      <td>10500</td>
      <td>86</td>
      <td>1.0</td>
      <td>8.190476</td>
    </tr>
    <tr>
      <th>199</th>
      <td>Neilson Powless</td>
      <td>29000</td>
      <td>70</td>
      <td>1.0</td>
      <td>2.413793</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Mark Cavendish</td>
      <td>2500</td>
      <td>54</td>
      <td>1.0</td>
      <td>21.600000</td>
    </tr>
    <tr>
      <th>163</th>
      <td>Mikkel Frolich Honore</td>
      <td>25500</td>
      <td>53</td>
      <td>1.0</td>
      <td>2.078431</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Gino Mader</td>
      <td>18500</td>
      <td>46</td>
      <td>1.0</td>
      <td>2.486486</td>
    </tr>
    <tr>
      <th>976</th>
      <td>Gianni Marchand</td>
      <td>2500</td>
      <td>26</td>
      <td>1.0</td>
      <td>10.400000</td>
    </tr>
    <tr>
      <th>977</th>
      <td>Finn Fisher Black</td>
      <td>2500</td>
      <td>21</td>
      <td>1.0</td>
      <td>8.400000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df['selected'] > 0.5].sum()
```




    name_         Ben O ConnorSonny ColbrelliGino MaderMatej Moh...
    value                                                    936500
    points                                                     1570
    selected                                                   15.0
    efficiency                                            69.569662
    dtype: object



The cheapest team that could have won, costs 936.500 euro, which means that only 53 percent of the budget is used. There are a lot of riders shared with the optimal team, among who again Wout van Aert. What a guy!

# Worst team
Finally, the worst team is a trivial problem: just select 15 (cheap?) riders with no points. One way to have a more interesting worst team, could be to set a lower limit on the team cost, e.g. 1.700.000 euro. 


```python
count_required = 15

M = pyo.ConcreteModel()
M.ITEMS = pyo.Set(initialize=points.keys())
M.x = pyo.Var(M.ITEMS, within=pyo.Binary)

M.points = pyo.Objective(expr=sum(points[i]*M.x[i] for i in M.ITEMS), sense=pyo.minimize)
M.weight = pyo.Constraint(expr=sum(cost[i]*M.x[i] for i in M.ITEMS) >= 1700000)
M.count = pyo.Constraint(expr=sum(count[i]*M.x[i] for i in M.ITEMS) == count_required)

M.popular = pyo.ConstraintList()
for p in popular:
    M.popular.add(M.x[p] == 0)

opt = pyo.SolverFactory("cbc.exe")
results = opt.solve(M)
print(results)
```

    
    Problem: 
    - Lower bound: -inf
      Upper bound: inf
      Number of objectives: 1
      Number of constraints: 9
      Number of variables: 979
      Sense: unknown
    Solver: 
    - Status: ok
      Message: CBC 2.10.3 optimal, objective 0; 0 nodes, 0 iterations, 0.013 seconds
      Termination condition: optimal
      Id: 0
      Error rc: 0
      Time: 0.058875083923339844
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
      <th>name_</th>
      <th>value</th>
      <th>points</th>
      <th>selected</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>15</th>
      <td>Bob Jungels</td>
      <td>91500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Emanuel Buchmann</td>
      <td>116000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Rudy Molard</td>
      <td>67500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Thibaut Pinot</td>
      <td>172500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>284</th>
      <td>Andrea Pasqualon</td>
      <td>77000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>308</th>
      <td>Chris Froome</td>
      <td>131500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>333</th>
      <td>John Degenkolb</td>
      <td>99000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>450</th>
      <td>Dylan Groenewegen</td>
      <td>135000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>471</th>
      <td>Fabio Aru</td>
      <td>88000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>515</th>
      <td>Vincenzo Nibali</td>
      <td>158000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>550</th>
      <td>Jan Polanc</td>
      <td>68500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>735</th>
      <td>Eduard Prades</td>
      <td>114500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>819</th>
      <td>Ilnur Zakarin</td>
      <td>99500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>879</th>
      <td>Nairo Quintana</td>
      <td>207500</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>925</th>
      <td>Niki Terpstra</td>
      <td>79000</td>
      <td>0</td>
      <td>1.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df['selected'] > 0.5].sum()
```




    name_         Bob JungelsEmanuel BuchmannRudy MolardThibaut ...
    value                                                   1705000
    points                                                        0
    selected                                                   15.0
    efficiency                                                  0.0
    dtype: object



We managed to find a team that uses nearly all the money and still scores no points. This result could also have been achieved without a MILP solver, and by selection all riders with no points, and then sorting them from expensive to cheap and take the 15 most expensive.

# Closing thoughts

As the bottleneck is in the quality of predicting scores (when combined with knapsack), perhaps I should consider building a machine-learning model to predict scores? Or perhaps apply a cool Bayesian rating system like Trueskill to the cycling data? Maybe this can be inspiration for extending Trueskill, which can handle both individual competition as well as team competition, but not a hybrid. The hybrid one is very typical in modern cycling where riders have both individual and team goals, which can sometimes be [conflicting](https://www.cyclingnews.com/news/dutch-tactics-gone-wrong-in-womens-road-race-at-the-world-championships/).

-----

## Related posts
- [MegaTomBike 2021 - team creation](/2021/03/27/megatombike2021.html)

## Want to leave a comment?
Very interested in your comments but still figuring out the most suited approach to this. For now, feel free to send me an [email](mailto:lenseswaenen@gmail.com).