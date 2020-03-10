---
layout: post
title: FiveThirtyEight Riddler Classic

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

# One Player Optimal strategy
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

# But guess who is two player!?
Turns out, the above aside is a nice thought experiment but doesn't actually return an optimal strategy in the two player game.

This is trickier because we no longer trying to minimize total guesses — we are trying to beat an opponent. There are analogies here to the coin flipping riddler from two weeks ago. When you are behind (e.g. if the opponent is 1 move away from winning), it is advantageous to take more risk.

(wish I had more non-sports analogies) but this is just like:
* pulling the goalie in hockey when your team is down at the end of the game
* throwing a hail mary in football

Interestingly, this only applies in situations where "doing your best" is not the goal but rather being above some arbitrary limit (by any amount) is the goal. This seems like an interesting distinction between sports and real life.

Let's work backwards from endgame. If either player is down to 1 remaining candidate, the other player must guess, because on the following turn they will lose.

# Dynamic Programming Approach
```python
"""Guess Who Strategy — Value Iteration (sorta)

Link to riddle: https://fivethirtyeight.com/features/how-good-are-you-at-guess-who/
"""

from typing import Dict, Tuple

from tdx.cli import fire_cli


def learn_guess_who(N: int) -> Tuple[Dict[Tuple[int], float]]:
    V = {}  # state-value table (can think of this as p(win))
    A = {}  # state-action table

    for sumij in range(2, N * 2 + 1):
        for i in range(1, sumij):
            j = sumij - i
            """This for loop looks a little funny.
            
            We need to update in diagonal snaking pattern.

            If our value table is indexed like this:
            11 12 13 14
            21 22 23 24
            31 32 33 34
            41 42 43 44

            Want iteration order like:
            11, 12, 21 13, 22, 31, 14, 23, ...

            This was my best effort at a clever way to go about this.

            Looking at the pattern the numbers in a diagonal add up to 
            the same value. Hence "sumij"

            Variables:
                i - current player's n possible characters
                j - non-current player's n possible characters 
                    remaining
            """

            if i == 1:
                # Base case (if you're up with 1 remaining, you should win)
                V[(i, j)] = 1
                A[(i, j)] = 0

            else:
                # initialize strategy to guess (represented by '0' action)
                # in this case you're win prob is 1/i
                V[(i, j)] = 1 / i
                A[(i, j)] = 0

                # check for better options
                for n_to_elim in range(1, int(i / 2) + 1):
                    p = n_to_elim / i
                    new_value = p * (1 - V[(j, n_to_elim)]) + (1 - p) * (
                        1 - V[(j, i - n_to_elim)]
                    )
                    if new_value > V[(i, j)]:
                        # if an action represents an improvement, make the update
                        V[(i, j)] = new_value
                        A[(i, j)] = n_to_elim

    return V, A


def main(N: int = 24, print_all: bool = False, print_538_answers: bool = True, print_list: bool=False):
    """Learn optimal strategies and values.

    Example how to run with fire_cli:
        $ python guess_who.py main --N=24 --print_all=True --print_538_answers=True
    
    Keyword Arguments:
        N {int} -- Number of characters to learn (default: {24})
        print_all {bool} -- Print values and optimal strategies for all situations (default: {False})
        print_538_answers {bool} -- Print values for 538 question (default: {True})

    """
    V, A = learn_guess_who(N)

    if print_all:
        for i in range(1, N + 1):
            for j in range(1, N + 1):
                print(f"({i},{j}): \t\t{V[(i,j)]:.3f}, {A[(i,j)]}")

    if print_538_answers:
        if N >= 24:
            for i in [4, 14, 24]:
                print(f"N: {i} \t Pr[player0 wins] = {V[(i,i)]:.5f}")
        else:
            print(
                f"N must be greater than 24 to print 538 answers! You ran with N={N}."
            )

    if print_list:
        print([V[(i,i)] for i in range(1, N + 1)])


if __name__ == "__main__":
    fire_cli({"main": main})()

```




