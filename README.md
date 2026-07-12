# Introduction to Quantum computing
With the recent advancements in Quantum computing, I think it's about time that I write something about the subject.  
I am quite hesitant - this is not my area of expertise, so if I write something incorrect here - please message me and I will correct it.  
My goal in this blogpost series is to get to the major Quantum computing algorithms - [Grover's algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm) and [Shor's algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm), while completely skipping the physical parts - which I know even less about.  
I will warn that there is some mathematical background necessary - particularly [https://en.wikipedia.org/wiki/Linear_algebra](Linear algebra) and [complex numbers](https://en.wikipedia.org/wiki/Complex_number) - but I will try to explain what I can.

## Background
Apparently, on the smallest scales, the world behaves very differently than what we're used to. This part is extremely hard to accept as it's substantially different from our day-to-day experiences, and while (in my opinion) we still do not know what *truly* happens at those scales - we have good mathematical models for those behaviors.  
Interestingly, those behaviors and\or the mathematical model could be used for actual computation that would have certain advantages to classical computing.  
Instead of rambling on - let me write out those behaviors:
1. Quantum particles (e.g. [Photons](https://en.wikipedia.org/wiki/Photon), [Electrons](https://en.wikipedia.org/wiki/Electron) and so on) have certain properties that do not have definite values before measurements. Some of those properties could be the particle's position, [momentum](https://en.wikipedia.org/wiki/Momentum), [spin](https://en.wikipedia.org/wiki/Spin_(physics)) (intrinsic [angular momentum](https://en.wikipedia.org/wiki/Angular_momentum)) and so on. We say that those properties are in a **[superposition](https://en.wikipedia.org/wiki/Superposition_principle)** - not because of lack of information about the property, but because that's how nature works. For example, a particle's [spin](https://en.wikipedia.org/wiki/Spin_(physics)) can be measured as "up" or "down", but is normally in a superposition of "up" and "down" - kind of a "mix" of them. We mark that as $\ket{\varphi} = a \ket{\uparrow} + b \ket{\downarrow}$ - and we say that the spin of the particle has a coefficient of "a" in the "up" state, and a coefficient of "b" to be in the "down" state.
2. Measuring a particle "collapses" the property into a base state - randomly (but not uniformly). For example, in the spin example, we asserted $\ket{\varphi} = a\ket{\uparrow} + b\ket{\downarrow}$. If we measure the spin - it would collapse into the state $\ket{\uparrow}$ with a probability of $\left|a\right|^2$, and collapse into the state $\ket{\downarrow}$ with a probability of $\left|b\right|^2$. This rule is known as [the Born rule](https://en.wikipedia.org/wiki/Born_rule), and dictates a strong assumption on the coefficients - **their values squared must sum up to 1**, since they are probabilities. In mathematical notation: $a^2 + b^2 = 1$, and generally for $n$ coefficients that I will be marking as $c_i$: $\sum_{i=1}^{n}{\left|c_i\right|}^2 = 1$. Note that **we do not control the result** - nature decides on its own based on the coefficients and the Born rule. Note that after a measurement, follow-up measurements result in the same value always - that's why we call this phenomenon a "collapse". I do not want to get into the engineering or physics aspects, but maintaining quantum particles in superpositions without making the collapse due to unintentional measurements (I will keep the definition of a measurement vague here on purpose) - is the main engineering challenge in Quantum computing these days.
3. The coefficients in superposition could be negative or even complex numbers - as long as their sum of squares is exactly 1. This might look weird or useless, but that's the source of Quantum computing's power.
4. We can manipulate the superposition coefficients without measuring them. This is done via something called "Quantum gates" (the analogue of traditional [logic gates](https://en.wikipedia.org/wiki/Logic_gate)), and is mathematically done by multiplying the algebraic representation of the quantum state with a [Unitary matrix](https://en.wikipedia.org/wiki/Unitary_matrix). More on that later.
5. Quantum entanglement - we can have two (or more) particles with a **joint state**. When I say a "joint state" I mean that you cannot think as the state of a single particle anymore - the states are "correlated" in some sense. That also means that measuring one particle *immediately* affects the other, i.e. the joint state of these particles collapses together. This also means manipulating the state (as we discussed) - manipulates the states of both particles at the same time.

As you can see, these are a lot of assumptions - and they also imply many limitations on Quantum computers - unlike popular descriptions of Quantum computing - you cannot simply "evaluate all states in parallel" and return the correct one. Instead, one has to delicately assign "favorable" coefficients to quantum states and eventually perform a measurement that would give a significant probability to the "correct" outcome, and then check if we got the desired solution.

## Qubits
Just like regular [bits](https://en.wikipedia.org/wiki/Bit) we have on a classic computer - we have the Quantum computing equivalent called "Qubits". Classical bits are either 0 or 1, and are implemented using voltage (high voltage above a certain threshold represents a digital 1, low voltage represents a 0).  
Qubits also have 0 and 1 equivalents - for example, a spin of a particle can be used for that: spin down could represent a 0 and spin up could represent a 1 - which we will represent as $\ket{0}$ and $\ket{1}$. Since we have a superposition - a Qubit would have coefficients to those base states: $a\ket{0} + b\ket{1}$. Let us also remember that $a$ and $b$ can be complex numbers, and their sum of squares must add up to 1: $\left|a\right|^2 + \left|b\right|^2 = 1$. Sometimes you might see those coefficients described as **probability amplitudes** - for all means and purposes - it's just a name we give them.  
I will avoid the question of how those states are actually implemented - spin is not the only way to do so - and treat it as a "black-box", just like one doesn't necessarily need to know how classical bits are implemented.  
One important aspect of Qubits is that they cannot be erased or copied - this has many implications on Quantum gates which I will describe shortly.  
Of course, just like one bit is not very interesting - one Qubit might not be as powerful as many. In Quantum computing, we are interested in **multi-Qubit systems**, exploiting the superposition and entanglement properties I mentioned earlier.  
As an example, suppose we have 2 Qubits - they can have a joint state of $\frac{1}{\sqrt{2}} \ket{00} + \frac{1}{\sqrt{2}} \ket{11}$ (note how the coefficients's squares sum up to 1!) - if we perform a measurements - both Qubits collapse to the same base state - $\ket{0}$ or $\ket{1}$! This is how entanglement looks like mathematically, and we will see its use in this blogpost series.  
One more important thing - we will represent the states $\ket{0}$ and $\ket{1}$ as column vectors - this will be useful when we discuss Quantum gates. So:

$$
\ket{0} = \begin{bmatrix}
1 \\
0
\end{bmatrix}
$$

$$
\ket{1} = \begin{bmatrix}
0 \\
1
\end{bmatrix}
$$

In a multi-Qubit system we will extend the rows:

$$
\ket{00} = \begin{bmatrix}
1 \\
0 \\
0 \\
0
\end{bmatrix}
$$

$$
\ket{01} = \begin{bmatrix}
0 \\
1 \\
0 \\
0
\end{bmatrix}
$$

$$
\ket{10} = \begin{bmatrix}
0 \\
0 \\
1 \\
0
\end{bmatrix}
$$

$$
\ket{11} = \begin{bmatrix}
0 \\
0 \\
0 \\
1
\end{bmatrix}
$$

## Quantum gates
A Quantum gate is the equivalent of a classical logic gate, and is a building-block of quantum operations.  
As I mentioned - all Quantum operations that are not measurements are essentially Unitary matrix multiplications with the vector that represents the current quantum state.  
As a reminder, a [Unitary matrix](https://en.wikipedia.org/wiki/Unitary_matrix) means that the [conjugate transposed](https://en.wikipedia.org/wiki/Conjugate_transpose) matrix is the [inverse](https://en.wikipedia.org/wiki/Invertible_matrix), or, in other words: $UU^*=U^*U=I$. In physical notation you might see the dagger ($\dagger$) notation used: $UU^\dagger=U^\dagger U=I$.  
That means that:
1. The number of inputs and outputs in a Quantum gate must be equal (so gates like [OR gate](https://en.wikipedia.org/wiki/OR_gate) cannot be quantum gates). A gate working on $n$ Qubits will be represented by a square $2^n \times 2^n$ matrix.
2. Quantum gates must be reversible.
3. Quantum gates are linear transformations that preserve the vector norm (that makes their matrices Unitary) - which means we can examine how they operate on the base states and use Linear algebra to see how they operate on a superposition state.

Let's examine some common Quantum gates!

### The Pauli-X, Pauli-Y and Pauli-Z gate
The **Pauli-X gate** works on a single Qubit and looks like this:

$$
X = \begin{bmatrix}
0 & 1 \\
1 & 0
\end{bmatrix}
$$

Let us see how it affects the base states:

$$
X \ket{0} = \begin{bmatrix}
0 & 1 \\
1 & 0
\end{bmatrix} \begin{bmatrix}
1 \\
0
\end{bmatrix} = \begin{bmatrix}
0 \\
1
\end{bmatrix}
$$

$$
X \ket{1} = \begin{bmatrix}
0 & 1 \\
1 & 0
\end{bmatrix} \begin{bmatrix}
0 \\
1
\end{bmatrix} = \begin{bmatrix}
1 \\
0
\end{bmatrix}
$$

I am assuming you are familiar with [matrix multiplication](https://en.wikipedia.org/wiki/Matrix_multiplication).  
Note how the Pauli-X gate, when operating on pure states - is essentially a [NOT gate](https://en.wikipedia.org/wiki/Inverter_(logic_gate)), but of course - there is no classical equivalent of working on a superposition: $X\left(a \ket{0} + b \ket{1} \right) = b \ket{0} + a \ket{1}$.  
We will mark this as:  
$\ket{0} \to \ket{1}$  
$\ket{1} \to \ket{0}$

The matrix representation of the **Pauli-Y gate** looks like this:

$$
Y = \begin{bmatrix}
0 & -i \\
i & 0
\end{bmatrix}
$$

And this is how it operates on the base states:  
$\ket{0} \to i \ket{1}$  
$\ket{1} \to -i \ket{0}$

Note how it uses [the imaginary number i](https://en.wikipedia.org/wiki/Imaginary_unit).

Lastly, here is the **Pauli-Z gate**:

$$
Z = \begin{bmatrix}
1 & 0 \\
0 & -1
\end{bmatrix}
$$

$\ket{0} \to \ket{0}$  
$\ket{1} \to - \ket{1}$

Note how the Pauli-Z gate flips the sign of $\ket{1}$ (but leaves $\ket{0}$ unchanged).

### Hadamard gate
The **Hadamard gate** (denoted as H) is a very useful gate - it takes "pure" states and turns them into "mixed" states, and vice versa:

$$
H = \frac{1}{\sqrt{2}}\begin{bmatrix}
1 & 1 \\
1 & -1
\end{bmatrix}
$$

Note how it operates on pure states:

$\ket{0} \to \frac{1}{\sqrt{2}} \left( \ket{0} + \ket{1} \right)$  
$\ket{1} \to \frac{1}{\sqrt{2}} \left( \ket{0} - \ket{1} \right)$

And how it operates on mixed states:

$\left( \frac{1}{\sqrt{2}} \ket{0} + \frac{1}{\sqrt{2}} \ket{1} \right) \to \ket{0}$  
$\left( \frac{1}{\sqrt{2}} \ket{0} - \frac{1}{\sqrt{2}} \ket{1} \right) \to \ket{1}$

Note how Hadamard gates are their own inverses.  
The operation itself is extremely useful since it means we can always start with Qubits that are in a pure state (by performing a measurement, for example) and turn them into mixed states - basically, **creating superposition**. If a pure-state Qubit is operated by a Hadamard gate, measuring it will result in a truly 50%-50% random chance of being either $\ket{0}$ or $\ket{1}$, as dictated by the Born rule:  

$\left( \frac{1}{\sqrt{2}} \right)^2 = \left( - \frac{1}{\sqrt{2}} \right)^2 = \frac{1}{2}$

### Phase gates
**Phase gates** (marked as S and sometimes as P) map the phase of its input Qubit by some angle $\varphi$:

$$
P(\varphi) = \begin{bmatrix}
1 & 0 \\
0 & e^{i \varphi}
\end{bmatrix}
$$

So:

$\ket{0} \to \ket{0}$  
$\ket{1} \to e^{i \varphi} \ket{1}$

Sometimes in literature you might see **T gates** - those refer to Phase gates with $\varphi = \frac{\pi}{4}$.

### CNOT gate
Up until now, we've seen Quantum gates that work on a single Qubit - the **CNOT** (short for "Controlled-NOT") gate operates on two:

$$
CNOT = \begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 1 \\
0 & 0 & 1 & 0
\end{bmatrix}
$$

Let's see how it works on pure states:

$\ket{00} \to \ket{00}$  
$\ket{01} \to \ket{01}$  
$\ket{10} \to \ket{11}$  
$\ket{11} \to \ket{10}$

We can see the CNOT gate works on two Qubits - the "control" Qubit and the "input" Qubit - if the control Qubit is 0 the input Qubit is left unchanged, but if the control Qubit is 1 - the input Qubit is flipped.  
This gate is equivalent to a classical [XOR gate](https://en.wikipedia.org/wiki/XOR_gate) but again - it works on Qubits that might be in a superposition.

### Toffoli gate
The **Toffoli gate** (also known as a **CCNOT gate**) works on 3 Qubits:

$$
CCNOT = \begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 \\
0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 \\
\end{bmatrix}
$$

Let us also see how it operates on pure states:

$\ket{000} \to \ket{000}$  
$\ket{001} \to \ket{001}$  
$\ket{010} \to \ket{010}$  
$\ket{011} \to \ket{011}$  
$\ket{100} \to \ket{100}$  
$\ket{101} \to \ket{101}$  
$\ket{110} \to \ket{111}$  
$\ket{111} \to \ket{110}$

The way to think about Toffoli gates is like a gate with two control Qubits and one input Qubit, with the following rule - the target Qubit is inverted if and only if the two control Qubits are on.  
The amazing thing about Toffoli gates is that they are **classically universal** - in a sense that any reversible classical circuit can be implemented using Toffoli gates.  
This fact alone means that Quantum computers are at least as strong as classical computers, computationally.

## Quantum circuits
Just like classical circuits, Quantum circuits become more interesting when you combine different gates together.  
However, we have several very strong limitations on Quantum circuits:
1. There is no way to take a Qubit and clone its state - this is known as the [No-cloning theorem](https://en.wikipedia.org/wiki/No-cloning_theorem). That means we cannot use the same Qubit as an input to two separate gates, for example.
2. The number of input Qubits has to be equal to the number of output Qubits - this is the result of the fact Quantum gates must be reversible and the No-cloning theorem.
3. The exception to the irreversibility statement comes in two forms:
  1. Measurement - once we measure the system, all previous amplitutes are gone - we "collapse" the Qubit's state to one of its base states, based on the Born rule. At that point, the Qubit behaves like a classical bit.
  2. We can discard Qubits if we so choose - ignoring their state completely.

When we build Quantum circuits, we usually pass Qubits in a pipeline of some sort and eventually perform a measurement.

### Quantum universal function
We can build a reversible Quantum circuit which calculates some function $f:\\{0,1\\}^n \to \\{0,1\\}$ (i.e. a function that receives $n$ bits and returns a single bit).  
Note the function $f$ does *not* have to be reversible!  
The way we do this is by turning the function into a new Quantum function: $F\left( x \right):\\{ 0, 1 \\}^{n+1} \to \\{ 0, 1 \\}^{n+1}$ - note we now have the same amount of Qubits as inputs and outputs.  
What we do is add a new input Qubit $\ket{b}$, and implement $\ket{x} \ket{b} \to \ket{x} \ket{f \left( x \right) \oplus b}$, or in other words:
- We add a new input Qubit $\ket{b}$.
- The entire input $\ket{x}$ is preserved as the output $\ket{x}$.
- We add one more output Qubit which will be $\ket{f \left( x \right) \oplus b}$ - so, the information from $f\left( x \right)$ exists in the last output Qubit, alongside the control Qubit $\ket{b}$.

## The Deutsch algorithm
Here is our first example of a Quantum algorithm - it might look boring or impractical at first, but it's a nice example of the "advantage" you get when using Quantum computing.  
The problem is quite simple - let's assume we have a function $f$ that gets one bit and returns one bit. That function can be "constant" (i.e. $f(0) = f(1)$) or "balanced" (the number of inputs that yield 0 is equal to the number of inputs that yield 1) - we'd like to sample the function $f$ and indicate if the function is constant or not.  
Classically, the solution is simple - call $f(0)$ and compare to $f(1)$ - but that means we have to invoke function $f$ **twice**.  
The algorithm we will present (called [the Deutsch algorithm](https://en.wikipedia.org/wiki/Deutsch–Jozsa_algorithm)) samples the function only once!  
We do that without learning the value of $f$ for a particular input - we manipulate the phase instead.

### The function as a Quantum gate
As a reminder, a Quantum gate must be reversible, so we can't use the function $f$ as-is.  
Instead, we will build a gate $U_f$ that operates on two Qubits and performs the following:  
$U_f \ket{x} \ket{y} = \ket{x} \ket{y \oplus f\left( x \right)}$

So, the first Qubit is preserved, and the second Qubit is XORed with the function value at $\ket{x}$.  
This is **exactly the CNOT gate** we've presented earlier, except that the control Qubit depends on the function $f$.

### Preparing the input
We start with two Qubits: $\ket{0} \ket{1}$ and pass each of them through a Hadamard gate.  
We thus get the first Qubit as $\frac{1}{\sqrt{2}} \left( \ket{0} + \ket{1} \right)$, and the second Qubit as $\frac{1}{\sqrt{2}} \left( \ket{0} - \ket{1} \right)$, so the new state is $\frac{1}{2} \left( \ket{00} - \ket{01} + \ket{10} - \ket{11} \right)$.

### Applying the function
Now we'd like to apply $U_f$ that we described earlier - let's see how it affects the Qubits.  
Let us examine the second Qubit, which is now in the state of $\frac{1}{\sqrt{2}} \left( \ket{0} - \ket{1} \right)$.  
If $f(x) = 0$ then $U_f$ does nothing to the second Qubit.  
However, if $f(x) = 1$ then $U_f$ flips the sign of the second Qubit - so we get a new state: $\frac{1}{\sqrt{2}} \left( \ket{1} - \ket{0} \right)$.  
In other words - if the input was $\ket{x}$, we now get as output: ${\left( -1 \right)}^{f\left( x \right)} \ket{x}$.  
This is quite remarkable - note how the function value becomes **encoded as a phase** instead of a bit!
