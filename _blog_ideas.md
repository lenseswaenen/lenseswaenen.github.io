# Inspiration 
- Cleve's corner
- Towards Data Science
- Medium
- jorisbaan.nl =  jsbaan.github.io
- https://karpathy.ai/
- http://jakevdp.github.io/blog/2014/06/06/frequentism-and-bayesianism-2-when-results-differ/
--> Own blog or on those websites?

# Practical

## Done
- github pages + jekyll + markdown: check
- improved theme: check
- reactions: twitter & e-mail
- Google analytics
- Fix layout on mobile devices
- Decreased reading time
- SEO
- Moved test page
- Copyright + attribution Slimoentjes
- Discuss with Robbert
- More compact table formatting

## TODO:

- Internship portfolio?
- Formatting: Output blocks in grey? Inline ```stuff``` nicer?
- Check unsplash license conditions?
- Check out Medium?!
- Tags
- Sioux background
- Prioritize backlog
- Fix ico, fix CNAME
- Duplicate on medium/tds?
- Reactions section
- Calibrate reading time
- Clean up layout (minima + then some)
- Upload notebooks? 
- Downgrade to Ruby 2.7 on Dell Precision
- favicon.png fix

# Contents

## Done

- Nerd sniping
	- Extension: way of estimating/calculating Pi
- MegaTomBike 2021 & knapsack problem
- Love letter to matrix logarithm
- Table tennis home advantage NL
- Table tennis 1000 wins - NL
- MegaTomBike 2021: final results
- Linear depenency of linear inequality constraints & prevalence (e.g. in poly PVAJ constraints)
- clip(x, l, u) vs clip(l, x, u)

## Up next

- DFO & new paradigm
- MegaTomBike 2022
- Wesley Smith matrix inverse
- Finite derivative
- Love letter to precision matrix
- Dynobend youtube videos


## Backlog

- Roadmap items:
	- Probabilistic programming & tracking: Whitepaper
	- Optimal control: Whitepaper, Vexar
	- Bayesian DFOGN:
	- Belief propagation & critique: https://gaussianbp.github.io/ ***
	- GaBP & Blocked linear algebra
- Projects:
	- Dynobend Youtube videos
	- ASML and patent
	- ASML & tensor contractions
- Elements from projects:
	- Anisotropic stitching
	- Generalized Tikhonov solver
	- Orthogonal integrators**
	- CasADi & trajectory generation
	- Minimum time & least squares approach
	- Gauss Seidel and random matrices
	- Constraint pruning / simplification
	- Gamma processess for filling/degradation
	- Unconstrained subspace --> Splitting or AS/IP internal optimization (e.g. cholesky)
	- Generic finite differences***
	- Tensor contractions & applications/speedups
	- AMRs and CasADi
	- 1D bang-bang snippets
	- QR vs Cholesky (speed difference)
	- BFGS vs BOBYQA Hessian convergence
- Puzzles/Recreational:
	- Dots & MILP
	- Pyomo & overhang*
	- Sprinkler pattern
	- Garage door shapes
	- Traveling Sam Problem (w TSP Concorde)
	- Pi Day & Matt Parker (2022)
	- sum(x) = 1: 3d slice in 4d space
	- Comic: Trolleying Salesman Problem
	- Variational physics, optimal solutions in nature
	- Numberphile regular polyhedra & linear inequalities
- Sports/games:
	- Game design: leveling the playing field (EN+NL summary?)***
	- Rating systems. Love letter to TrueSkill. Ranking the sports
		- Comparison of TrueSkill and Glicko?
		- TrueSkill in other prob. progr. languages (Pyro, ForneyLab)
		- Comparison/review of rating systems in practise.
	- MegaTomBike 2022:
		- MegaTomBike distinct teams?
- Instructive:
	- Viewpoints of Kalman filters
	- Viewpoints of regularization
	- Message passing, belief propagation, ...?
	- DFOGN vs LM. Newton vs regula falsi
	- Complexities of Poisson solvers + typical computation times?
	- Precision matrices
	- Root finding & QR algorithm (MATLAB roots)
- Lore:
	- Gembicki
- Hall of shame
	- Wesley Smiths rectangular matrix inverse**
	- 1000 variable Non-negative linear least squares with fminsearch (Nelder-Mead)
	- min ||a - s*b||
	- clip(x, l, u) vs clip(l, x, u)
	- MATLAB positive/negative zeros, max and infs, ...
- Throwback thursday --> KULeuven
	- Multigrid
	- Numerical linear algebra homework on direct sparse matrices
	- Master physics?
- Internships
	- Multigrid for LSA? Multigrid for stitching. Yin tat lee
	- Arjan (electron optics?), Monique, Paul, Jasper, Gerrit, Arno, Abby, Milan, Angelique, Lieke, Greta!
	

## Tags/categories?
- Sports
- Fun
- Loveletters
- Sioux/Work
- Throwback thursday
- Spotlight?

# Tutorials

## Making a new post
1. ipynb --> md (with images embedded?!) -> _posts + rename
2. Search for image on unsplash?
3. Add header to md (+ mast photo?)
4. Add footer
5. Replace mast image?
6. `bundle exec jekyll serve` in git bash @ repo & debug/modify at http://127.0.0.1:4000/ (or localhost:4000)
7. Git Push
8. Tweet
9. Footer: update with tweet reference

## Setting up a new development environment
1. Install Ruby (preferably 2.7 i.s.o. 3.0 (Github-pages support...))
2. Install Jekyll and Bundler (`gem install jekyll bundler`)
3. Run `bundle`
4. `bundle exec jekyll serve`