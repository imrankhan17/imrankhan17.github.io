# Solving the secretary problem with Python

The secretary problem is a hypothetical problem regarding how best to choose a candidate for a job position.  The formulation and mathematical solution is outlined quite clearly [here](https://en.wikipedia.org/wiki/Secretary_problem).  This post explores how to solve the problem numerically in Python.

In short, an known number of candidates are interviewed one at a time in a random order to fill one position.  The interviewer knows nothing about the abilities of the candidates to come but can rank those that have been seen from best to worst.  After each interview, the candidate is either rejected or accepted and this decision is final.  The solution to this problem will maximise the probability of selecting the best candidate.   

We start by forming a list of candidates ordered from 1 to n where 1 is the best candidate.  

```python
import numpy as np
candidates = np.arange(1, n+1)
# simulate random order of candidates being interviewed
np.random.shuffle(candidates)
```

The optimal solution is to reject the first `n/e` applicants (e ~ [2.718](https://en.wikipedia.org/wiki/E_(mathematical_constant)) and choose the first candidate who is better than the best candidate from the `n/e` rejected candidates.  If there is no candidate who is better then choose the last candidate.

For our n candidates we reject the first `int(round(n/np.e))` candidates.  The best candidates from this list will be the minimum value (since lower is better).

```python
stop = int(round(n/np.e)) 
best_from_rejected = np.min(candidates[:stop])
rest = candidates[stop:]
# Index zero since we choose the first candidate who is better
chosen_candidate = rest[rest < best_from_rejected][0]
```

However, the last line will return an error if there is no candidate in `rest` who is better than `best_from_rejected`.  Hence we catch this error and return the last candidate.

```python
stop = int(round(n/np.e)) 
best_from_rejected = np.min(candidates[:stop])
rest = candidates[stop:]
try:
	return rest[rest < best_from_rejected][0]
except IndexError:
    return candidates[-1]
```

We can wrap all of this in a function.
```python
def choose_candidate(n):
    candidates = np.arange(1, n+1)
    np.random.shuffle(candidates)
    
    stop = int(round(n/np.e)) 
    best_from_rejected = np.min(candidates[:stop])
    rest = candidates[stop:]
    
    try:
        return rest[rest < best_from_rejected][0]
    except IndexError:
        return candidates[-1]
```

Now let's see how 










