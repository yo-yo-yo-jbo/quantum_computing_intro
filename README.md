# Introduction to Quantum computing
With the recent advancements in Quantum computing, I think it's about time that I write something about the subject.  
I am quite hesitant - this is not my area of expertise, so if I write something incorrect here - please message me and I will correct it.  
My goal in this blogpost series is to get to the major Quantum computing algorithms - [Grover's algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm) and [Shor's algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm), while completely skipping the physical parts - which I know even less about.  
I will warn that there is some mathematical background necessary - particularly [https://en.wikipedia.org/wiki/Linear_algebra](Linear algebra) and [complex numbers](https://en.wikipedia.org/wiki/Complex_number) - but I will try to explain what I can.

## Background
Apparently, on the smallest scales, the world behaves very different than what we're used to. This part is extremely hard to accept as it's substantially different from our day-to-day experiences, and while (in my opinion) we still do not know what *truly* happens at those scales - we have good mathematical models for those behaviors.  
Interestingly, those behaviors and\or the mathematical model could be used for actual computation that would have certain advantages to classical computing.  
Instead of rambling on - let me write out those behaviors:
1. Quantum particles (e.g. [Photons](https://en.wikipedia.org/wiki/Photon), [Electrons](https://en.wikipedia.org/wiki/Electron) and so on) have certain properties that do not have definite values before measurements. Some of those properties could be the particle's position, [momentum](https://en.wikipedia.org/wiki/Momentum), [spin](https://en.wikipedia.org/wiki/Spin_(physics)) (intrinsic [angular momentum](https://en.wikipedia.org/wiki/Angular_momentum)) and so on. We say that those properties are in a **[superposition](https://en.wikipedia.org/wiki/Superposition_principle)** - not because of lack of information about the property, but because that's how nature works. For example, a particle's [spin](https://en.wikipedia.org/wiki/Spin_(physics)) can be measured as "up" or "down", but is normally in a superposition of "up" and "down" - kind of a "mix" of them. We mark that as $\ket{\varphi} = a \ket{\uparrow} + b \ket{\downarrow}$ - and we say that the spin of the particle has a coefficient of "a" in the "up" state, and a coefficient of "b" to be in the "down" state.
2. Measuring a particle "collapses" the property into a base state - randomly (but not uniformly). For example, in the spin example, we asserted $\ket{\varphi} = a\ket{\uparrow} + b\ket{\downarrow}$. If we measure the spin - it would collapse into the state $\ket{\uparrow}$ with a probability of $\left|a\right|^2$, and collapse into the state $\ket{\downarrow}$ with a probability of $\left|b\right|^2$. This rule is known as [the Born rule](https://en.wikipedia.org/wiki/Born_rule), and dictates a strong assumption on the coefficients - **their values squared must sum up to 1**, since they are probabiliies. In mathematical notation: $a^2 + b^2 = 1$, and generally for $n$ coefficients that I will be marking as $c_i$: $\sum_{i=1}^{n}{\left|c_i\right|}^2 = 1$. Note that **we do not control the result** - nature decides on its own based on the coefficients and the Born rule. Note that after a measurement, follow-up measurements result in the same value always - that's why we call this phenomenon a "collapse". I do not want to get into the engineering or physics aspects, but maintaining quantum particles in superpositions without making the collapse due to unintentional measurements (I will keep the definition of a measurement vague here on purpose) - is the main engineering challenge in Quantum computing these days.
3. The coefficients in superposition could be negative or even complex numbers - as long as their sum of squares is exactly 1. This might look weird or useless, but that's the source of Quantum computing's power.
4. We can manipulate the superposition coefficients without measuring them. This is done via something called "Quantum gates" (the analogue of traditional [logic gates](https://en.wikipedia.org/wiki/Logic_gate)), and is mathematically done by multiplying the algebric representation of the quantum state with a [Unitary matrix](https://en.wikipedia.org/wiki/Unitary_matrix). More on that later. A measurement, by the way - is mathematically a multiplication of the state by a **non-*Unitary-matrix**. We'll see that later too.
5. Quantum entanglement - we can have two (or more) particles with a **joint state**. When I say a "joint state" I mean that you cannot think as the state of a single particle anymore - the states are "correlated" in some sense. That also means that measuring one particle *immediately* affects the other, i.e. the joint state of these particles collapses together. This also means manipiulating the state (as we discussed) - manipulates the states of both particles at the same time.

As you can see, these are a lot of assumptions - and they also imply many limitations on Quantum computers - unlike popular descriptions of Quantum computing - you cannot simply "evaluate all states in parallel" and return the correct one. Instead, one has to delicately assign "favorable" coefficients to quantum states and eventually perform a measurement that would give a significant probability to the "correct" outcome, and then check if we got the desired solution.

## Qubits
Just like regular [bits](https://en.wikipedia.org/wiki/Bit) we have on a classic computer - we have the Quantum computing equivalent called "Qubits". Clasical bits are either 0 or 1, and are implemented using voltage (high voltage above a certain threshold represents a digital 1, low voltage represents a 0).  
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
\ket{01} = \begin{bmatrix}
0 \\
0 \\
0 \\
1
\end{bmatrix}
$$

## Quantum gates
A Quantum gate is the equivalent of a classical logic gate, and is a building-block of quantum operations.  
As I mentioned - all Quantum operations that are not measurements are essentially Unitary matrix multiplications with the vector that represents the current quantum state.  
As a reminder, a [Unitary matrix](https://en.wikipedia.org/wiki/Unitary_matrix) means that the [conjugate trasposed](https://en.wikipedia.org/wiki/Conjugate_transpose) matrix is the [inverse](https://en.wikipedia.org/wiki/Invertible_matrix), or, in other words: $UU^*=U^*U=I$. In physical notation you might see the dagger ($\dagger$) notation used: $UU^\dagger=U^\dagger U=I$.  
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

The matrix representation of the **Pauli-Y gate** looks like this:

$$
Y = \begin{bmatrix}
0 & -i \\
i & 0
\end{bmatrix}
$$

And this is how it operates on the base states:  
$Y \ket{0} = i \ket{1}$  
$Y \ket{1} = -i \ket{0}$

Note how it uses [the imaginary number i](https://en.wikipedia.org/wiki/Imaginary_unit).

Lastly, here is the **Pauli-Z gate**:

$$
Z = \begin{bmatrix}
1 & 0 \\
0 & -1
\end{bmatrix}
$$

$Z \ket{0} = \ket{1}$  
$Z \ket{1} = - \ket{1}$

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

$H \ket{0} = \frac{1}{\sqrt{2}} \left( \ket{0} + \ket{1} \right)$  
$H \ket{1} = \frac{1}{\sqrt{2}} \left( \ket{0} - \ket{1} \right)$

And how it operates on mixed states:

$H \left( \frac{1}{\sqrt{2}} \ket{0} + \frac{1}{\sqrt{2}} \ket{1} \right) = \ket{0}$  
$H \left( \frac{1}{\sqrt{2}} \ket{0} - \frac{1}{\sqrt{2}} \ket{1} \right) = \ket{1}$

Note how Hadamard gates are their own inverses.  
The operation itself is extremely useful since it means we can always start with Qubits that are in a pure state (by performing a measurement, for example) and turn them into mixed states - basically, **creating superposition**. If a pure-state Qubit is operated by an Hadamard gate, measuring it will result in a truly 50%-50% random chance of being either $\ket{0}$ or $\ket{1}$, as dictated by the Born rule:  

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

$P(\varphi)\ket{0} = \ket{0}$  
$P(\varphi)\ket{1} = e^{i \varphi} \ket{1}$

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

$CNOT \ket{00} = \ket{00}$  
$CNOT \ket{01} = \ket{01}$  
$CNOT \ket{10} = \ket{11}$  
$CNOT \ket{11} = \ket{10}$

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

$CCNOT \ket{000} = \ket{000}$  
$CCNOT \ket{001} = \ket{001}$  
$CCNOT \ket{010} = \ket{010}$  
$CCNOT \ket{011} = \ket{011}$  
$CCNOT \ket{100} = \ket{100}$  
$CCNOT \ket{101} = \ket{101}$  
$CCNOT \ket{110} = \ket{111}$  
$CCNOT \ket{111} = \ket{110}$

The way to think about Toffoli gates is like a gate with two control Qubits and one input Qubit, with the following rule - the target Qubit is inverted if and only if the two control Qubits are on.  
The amazing thing about Toffoli gates is that they are **clasically universal** - in a sense that any reversible classical circuit can be implemented using Toffoli gates.  
This fact alone means that Quantum computers are at least as strong as classical computers, computationally.
