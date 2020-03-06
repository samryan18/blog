---
layout: post
title: FiveThirtyEight Riddler Classic

last_modified_at: 2020-03-05T13:01:27-05:00
---



## The Question
> At your local barbershop, there are always four barbers working simultaneously. Each haircut takes exactly 15 minutes, and there’s almost always one or more customers waiting their turn on a first-come, first-served basis.
>
> Being a regular, you prefer to get your hair cut by the owner, Tiffany. If one of the other three chairs opens up, and it’s your turn, you’ll say, “No thanks, I’m waiting for Tiffany.” The person behind you in line will then be offered the open chair, and you’ll remain at the front of the line until Tiffany is available.
>
> Unfortunately, you’re not alone in requesting Tiffany — a quarter of the other customers will hold out for Tiffany, while no one will hold out for any of the other barbers.
>
> One Friday morning, you arrive at the barber shop to see that all four barbers are cutting hair, and there is one customer waiting. You have no idea how far along any of the barbers is in their haircuts, and you don’t know whether or not the customer in line will hold out for Tiffany.
>
> What is the expected wait time for getting a haircut from Tiffany?
>
>
> <cite><a href="https://fivethirtyeight.com/features/can-you-get-a-haircut-already/">Can You Get A Haircut Already?</a></cite>

## How to approach this?
This is pretty easy to solve via simulation. I wrote some (not well optimized) code (see bottom) to do that and then set about trying to find the solution analytically, which turns out to be harder than I would have expected. It is relatively straightforward to enumerate possible cases and assign probabilities to each, but determining E[wait_time] conditional on being in a certain leaf of the tree is tricky.

## Analytical Approach: The 4 (but really 3) cases
Before jumping into expectation in times, let's break it down into cases. There are two important unknowns:

1. Does Tiffany finish first? [probability (1/4)]
2. Is the customer ahead of us waiting for Tiffany? [probability (1/4)]

Feels safe to assume these are independent so we can create a 2x2 grid to represent four cases:

|                         	    | Tiffany finishes first 	| Tiffany does not finish first 	|
|-------------------------	    |------------------------	|-------------------------------	|
| **customer is waiting**     	| p=(1/4)*(1/4)         	| p=(3/4)*(1/4)               	|
| **customer is not waiting** 	| p=(1/4)*(3/4)         	| p=(3/4)*(3/4)               	|


The first column (Tiffany finishes first) can be treated as a single case since if Tiffany finishes first, the other customer will surely get seated by her.

So the three effective cases are:

1. Another barber finishes first (not Tiffany) and the other customer is not waiting for Tiffany
    
    $$\text{probability} = (3/4) * (3/4)$$
    
    $$\mathbb{E}[\text{wait_time}] = E[\text{tiffany finish time} | \text{tiffany does not finish first}] $$

2. Another barber finishes first (not Tiffany) and the other customer is waiting for Tiffany
    * this situation is kind of awkward because both of us pass up on this barber. if it were me, I probably wouldn't have the heart to pass them up but let's overlook that
    
    $$\text{probability} = (3/4) * (1/4)$$
    
    $$\mathbb{E}[\text{wait_time}] = E[\text{tiffany finish time} | \text{tiffany does not finish first}] + 15 $$

3. Tiffany finishes first
    
    $$\text{probability} = (1/4)$$
    
    $$\mathbb{E}[\text{wait_time}] = E[\text{tiffany finish time} | \text{tiffany finishes first}] + 15 $$


## Conditional expectations for wait time
This is the tricky part.

Let's digress for a minute and think about calculating expectation for the minimum of a set of i.i.d. uniform random variables. (Credit to a [blog post comment](https://danieltakeshi.github.io/2016/09/25/the-expectation-of-the-minimum-of-iid-uniform-random-variables/) for helping me think about this portion of the problem.)

Say we have n random variables $$(X_1 ... X_n)$$, each distributed Uniform(0,15) and we want the expecation for the minimum of these variables, i.e. $$\mathbb{E} min(X_1 ... X_n) ]$$.

Lets imagine an X_{n+1} added to the set. Two properties will be true: 
* $$Pr[X_{n+1} < min(X_1 ... X_n)] = 1/(N+1)$$ because any of these N+1 i.i.d. variables have equal probability of being minimum
* $$Pr[X_{n+1} < min(X_1 ... X_n)] = E[ min(X_1 ... X_n) ] / 15$$ because $$X_{n+1}$$ is another uniform RV from 0 to 15 so the probability it is below the minimum is simply the probability mass below $$\mathbb{E} min(X_1 ... X_n) ] / 15$$

We can rearange these two equations to get $$\mathbb{E} min(X_1 ... X_n) ] = \frac{15}{(N+1)}$$

So:

$$\mathbb{E}[\text{first finisher time}] = \frac{15}{(4+1)} = 3\text{ minutes}$$

By similar logic (with maybe a bit more work than I'm giving it credit for):

$$\mathbb{E}[\text{arbitrary not first finishing barber}] = 9\text{ minutes}$$

## Plugging in

Back to the three cases:
1. Another barber finishes first (not Tiffany) and the other customer is not waiting for Tiffany
    
    $$\text{probability} = (3/4) * (3/4)$$
    
    $$\mathbb{E}[\text{wait_time}] = E[\text{tiffany_finish_time} | \text{tiffany does not finish first}] = 9\text{ minutes} $$

2. Another barber finishes first (not Tiffany) and the other customer is waiting for Tiffany
    * this situation is kind of awkward because both of us pass up on this barber. if it were me, I probably wouldn't have the heart to pass them up but let's overlook that
    
    $$\text{probability} = (3/4) * (1/4)$$
    
    $$\mathbb{E}[\text{wait_time}] = E[\text{tiffany_finish_time} | \text{tiffany does not finish first}] + 15 = 9+15\text{ minutes} $$

3. Tiffany finishes first
    
    $$\text{probability} = (1/4)$$
    
    $$\mathbb{E}[\text{wait_time}] = E[\text{tiffany_finish_time} | \text{tiffany finishes first}] + 15 = 3+15\text{ minutes} $$


```python
probabilities = [
    (3 / 4) * (3 / 4),
    (3 / 4) * (1 / 4),
    (1 / 4)
]

expecations = [
    9,
    9+15,
    3+15
]

total_e_wait_time = sum([p*e for p,e in zip(probabilities, expecations)])

print(f"E[wait time] = {total_e_wait_time} minutes!")
```

#### Output:
```
E[wait time] = 14.0625 minutes!
```

## Confirm our answer via simulation:

```python
import numpy as np


def run_sim():
    customer_is_waiting_for_tiffany = np.random.binomial(1, 0.25)
    barber_remaining_time = np.random.uniform(0, 15, size=3)
    tiffany_remaining_time = np.random.uniform(0, 15, size=1)[0]

    if tiffany_remaining_time < min(barber_remaining_time):
        # tiffany finishes first!
        wait_time = tiffany_remaining_time + 15
    elif customer_is_waiting_for_tiffany:
        # another barber finishes first but customer is waiting for Tiffany!
        # slightly awkward because we both pass up this barber
        wait_time = tiffany_remaining_time + 15
    else:
        # another barber finishes first and customer is down for that barber!
        wait_time = tiffany_remaining_time

    return wait_time


wait_times = []
N_TRIALS = 10000000

for _ in range(N_TRIALS):
    wait_times.append(run_sim())

print(f"95% CI:")
print(
    f"{np.mean(wait_times):.4f} +/- {2*np.std(np.array(wait_times))/np.sqrt(len(wait_times)) :.4f}"
)
```

#### Output:
```
95% CI:
14.0637 +/- 0.0044
```
