---
layout: post
title: Guess Who (Riddler Classic)

last_modified_at: 2020-03-07T13:01:27-05:00
---

## The Question
<details>
<summary>See question</summary>
<q>

Sticking with the board game theme, from Andrew Lin comes a closer examination of a classic game of reasoning and elimination:

In the game of “Guess Who,” each player first randomly (and independently of their opponent) selects one of N character tiles. While it’s unlikely, both players can choose the same character. Each of the N characters is distinct in appearance — for example, characters have different skin tones, hair color, hair length and accessories like hats or glasses.

Each player also has access to a board with images of all N characters. The players alternate taking turns, and during each turn a player has two options:

Make a specific guess as to their opponent’s selected character. If correct, the player who made the guess immediately wins. Otherwise, that player immediately loses.
Ask a yes-or-no question about their opponent’s chosen character, in order to eliminate some of the candidates. Importantly, if only one possible character is left after the question, the player must still wait until their next turn to officially guess that character.
Assume both players are highly skilled at choosing yes-or-no questions, so that they can always craft a question to potentially rule out (or in) any desired number of candidates. Also, both are playing to maximize their own probability of winning.

Let’s keep things (relatively) simple, and suppose that N = 4. How likely is it that the player who goes first will win?

Extra credit: If N is instead 24 (the number of characters in the original “Guess Who” game), now how likely is it that the player who goes first will win?

Extra extra credit: If N is instead 14, now how likely is it that the player who goes first will win?
<cite><a href="https://fivethirtyeight.com/features/how-good-are-you-at-guess-who/">How Good Are You At Guess Who?</a></cite>
</q>
</details>

## One Player Optimal strategy
Before bringing in the game theory side of things, lets figure out the optimal strategy to minimize guesses in the absence of an opponent. Asking a question which eliminates people in this case removes some proportion of the people.

Let's start to build an intuition with an example before jumping into some math.

There are N characters, each has a probability of 1/N of being the selected character. We can ask a question which has the possibility of eliminating any number of them from 1...N-1.

Our goal is to narrow it down quickly. There are lots of ways to frame this:
* binary search
* maximizing information gain (ask high entropy questions)

##### Assertion:
> If we have X characters remaining, and we ask a binary question which applies to M characters, we have a M/X chance of eliminating X-M characters and an (X-M)/X chance of eliminating M characters.

I'll skip confirming this fact (pretty easy to see on paper), but we can see a nice corollary:

$$\begin{eqnarray}
    E[\text{n_elim} | M] =& (M/X)(X-M)  + ((X-M)/X)M \nonumber \\ \\
    =& (1/X)(2M(X-M)) \nonumber \\ \\
    =& (1/X)(2MX - 2M^2)
\end{eqnarray}$$

To find the optimal value of a function, we can set the derivative to zero:

$$\begin{eqnarray}
    \frac{E[\text{n_elim} | M]}{dM} =& \frac{d((1/X)(2MX - 2M^2))}{dM} \nonumber \\ \\
                        =& (1/X)(2X - 4M) \nonumber \\ \\
    \rightarrow (1/X) * (2X - 4M^*) =& 0 \nonumber \\ \\
    X-2M^* =& 0 \nonumber \\ \\
    \rightarrow M^* =& \frac{X}{2} \nonumber \\ \\
\end{eqnarray}$$

To confirm our critical point is a maximum, check the sign of the second derivative:

$$\begin{eqnarray}
    \frac{d^2 E[\text{n_elim} | M]}{dM^2} =& \frac{d((1/X)(2X - 4M))}{dM} \nonumber \\ \\
                                    =& \frac{-4}{X} \nonumber \\ \\
                                    & \text{This is strictly negative so we have a max!} \nonumber \\ \\
\end{eqnarray}$$

So the optimal result here is to always choose M = X/2.

## But guess who is two player!?
Turns out, the above aside is a nice thought experiment but doesn't actually return an optimal strategy in the two player game.

This is trickier because we no longer trying to minimize total guesses — we are trying to beat an opponent. There are analogies here to the coin flipping riddler from two weeks ago. When you are behind (e.g. if the opponent is 1 move away from winning), it is advantageous to take more risk.

(wish I had more non-sports analogies) but this is just like:
* pulling the goalie in hockey when your team is down at the end of the game
* throwing a hail mary in football

Aside—
Interestingly, this high risk framework only applies in situations where "doing your best" is not the goal but rather being above some arbitrary limit (by any amount) at the end of some arbitrary amount of time is the goal. This seems like an interesting distinction between sports and real life.

Let's work backwards from endgame. If either player is down to 1 remaining candidate, the other player must guess, because on the following turn they will lose.


## Some math that looks fancier than it is
Let's introduce the concepts of states, actions, rewards, policies and value functions.

##### State
A **state** is a data structure encapsulating everything we need to know about a game.

In this case, a game state can be captured by the following:
1. who's turn is it?
2. How many characters has player 1 narrowed it down to?
3. How many characters has player 2 narrowed it down to?

In the code, I capture states with a tuple (X_1, X_2) representing how many characters the two players have narrowed it down to.

I swap the ordering if player 2 is up.

##### Action
An **action** is just... well an action that a user can take.

In this case, users can ask about a binary trait. Let's break this down a bit. 

> **Example situation:** There are 10 characters left and 1 of them has pink hair, and the user asks "does your character have pink hair?" This is an action. I am encoding actions by how many characters their question applied to. Since they asked about pink hair, this applies to 1 character. 

Furthermore, since they can only ever get the same information from asking something like "does your character *NOT* have pink hair?" (which applies to 9 charaters), I treat actions M and X-M as the same. So I capture the bottom half of the action space $$a \in 1...X/2$$. There is also one other action, which is making an actual guess. I encode this as action 0. So the action space can be represented as $$a \in 0...X/2$$

##### Reward
A **reward** is a tangible benefit to an agent. In this case, there are two possible rewards: winning and losing. We encode these as +1 and -1.

##### Policy
A mapping of states to actions. Call this $$A$$.

##### Value Function
Yay, we made it to the fun part. In this type of problem (Markov Decision Process), value functions exist w.r.t. a policies.

We define a **value function under a policy** as the long term expectation of reward following that policy. Call this $$V_{A}$$.

I'm going to skip a few steps ahead and make the assumption that we're following an optimal policy (as is our opponent), which was stated in the riddle.

Let's call $$A^*$$ the optimal policy and $$V^* = V_{A^*}$$ the value function under that policy. (In the code and henceforth, I'm just going to refer to these as A and V).

##### Back to the math

$$ E[R_t | a] = E(R|a) + \sum $$



<!-- 
$$\begin{eqnarray}
    E[\text{n_elim} | M] =& (M/X)(X-M)  + ((X-M)/X)M \nonumber \\ \\
    =& (1/X)(2M(X-M)) \nonumber \\ \\
    =& (1/X)(2MX - 2M^2)
\end{eqnarray}$$ -->

## Dynamic Programming Approach
<script src="https://gist.github.com/samryan18/ee1a06108aea97c9b28b9674802e2103.js"></script>
