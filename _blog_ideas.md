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
- Game design, leveling the playing field (EN + NL summary)
- Roots & eig
- MegaTomBike 2022
- Dynobend youtube videos
- Unbiased variance estimation & sympy
- Linear inequality visualization with matplotlib and con2vert
- Radially symmetric PSD 2D kernel (and KKN)
- Forming shoulders

## Up next

- Wesley Smith matrix inverse
- DFO & composite
- Finite derivative
- Love letter to precision matrix

## Backlog

- Roadmap items:
	- Probabilistic programming & tracking: Whitepaper
	- Optimal control: Whitepaper, Vexar
	- Bayesian DFOGN
	- Belief propagation & critique: https://gaussianbp.github.io/ ***
	- GaBP & Blocked linear algebra
- Projects:
	- ASML and patent: https://www.asml.com/en/news/stories/2023/novel-lens-manipulator?utm_source=social&utm_medium=TWITTER&utm_content=100003968663288&utm_campaign=Always%20On&s=09
	- Kadaster
	- ASML & tensor contractions
- Elements from projects:
	- Anisotropic stitching
	- Generalized Tikhonov solver
	- Orthogonal integrators** (https://thenumb.at/Exponential-Rotations/)
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
	- Matrix that crashed SVD computation...***
	- Unconstrained variables in a QP (2 methods)**
	- Sparse direct solvers!
	- Kadaster & updating covariance matrices
	- EKF failing on sine fitting** (https://dsp.stackexchange.com/questions/79451/kalman-filter-on-sinusoidal-signal)
- Puzzles/Recreational:
	- Dots & MILP (???)
	- Pyomo & overhang*
	- Sprinkler pattern
	- Garage door shapes
	- Pi Day & Matt Parker (2022)
	- sum(x) = 1: What does the 3d slice in 4d space look like?
	- Variational physics, optimal solutions in nature
	- Numberphile regular polyhedra & linear inequalities
- Sports/games:
	- Rating systems. Love letter to TrueSkill. Ranking the sports
		- Comparison of TrueSkill and Glicko?
		- TrueSkill in other prob. progr. languages (Pyro, ForneyLab)
		- Comparison/review of rating systems in practise.
		- Racketlon: ratings & point win differences
		- Table tennis: rating difference & win point/match win probability
	- Decision tree / flow chart on toy, puzzle, activity, game, sport
	- Table tennis ELO handicap
	- Table tennis bounce analysis
- Instructive:
	- Viewpoints of Kalman filters
	- Viewpoints of regularization (tikhonov, ridge regression, MAP, rubber banding, ...)
	- Message passing, belief propagation, ...?
	- DFOGN vs LM. Newton vs regula falsi
	- Complexities of Poisson solvers + typical computation times?
	- Precision matrices
- Lore:
	- Gembicki
- Hall of shame
	- Wesley Smiths rectangular matrix inverse***
	- 1000 variable Non-negative linear least squares with fminsearch (Nelder-Mead)
	- min ||a - s*b||
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
0. %matplotlib ipympl --> %matplotlib inline (& rerun notebook)
1. ipynb --> md (CL jupyter nbconvert foo.ipynb --to markdown) + img folder 
2. md -> _posts + rename (with date)
3. pngs -> assets/images/ 
4. Replace "foo_files" with "/assets/images/foo_files" for output images
5. Search for image on unsplash?
6. Add header to md (+ mast photo?)
7. Add footer
8. Replace mast image?
9. Delete output of some Python cells
10. `bundle exec jekyll serve` in git bash @ repo & debug/modify at http://127.0.0.1:4000/ (or localhost:4000)
11. Fix internal links
12. Related posts updates
13. Update _blog_ideas.md
14. Git Push
15. (Tweet)
16. (Footer: update with tweet reference)
17. Archive .ipynb

## Setting up a new development environment
1. Install Ruby (preferably 2.7 i.s.o. 3.0 (Github-pages support...))
2. Install Jekyll and Bundler (`gem install jekyll bundler`)
3. Run `bundle`
4. `bundle exec jekyll serve`

# Errata/improvements
- More acknowledgements overall?
- Unbiased variance estimation: Jake's video is one [I] frequently...
- Unbiased variance estimation: ...data are [is] a consistent estimator to mu....
- Unbiased variance estimation: ...assymetry...
- Unbiased variance estimation: [?!] Unfortunately this is not the case
- A new spatial covariance kernel: Could I have used this as well?: https://docs.gpytorch.ai/en/stable/kernels.html#gpytorch.kernels.PiecewisePolynomialKernel
