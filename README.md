# Qiskit Implementations of Important Quantum Algorithms
Here I implemented some important Quantum Algorithms using Qiskit. I implemented them using Jupyter Notebook.

The algorithms I implemented are:
- Grover Search Algorithm for $2\times 2$ Sudoku
- Iterative Phase Estimation
- Kushlevitz and Mansour Algorithm: [LEARNING DECISION TREES USING THE FOURIER SPECTRUM](https://dl.acm.org/doi/pdf/10.1145/103418.103466)

## Problem 1: Grover Search Algorithm for $2\times 2$ Sudoku

### Problem Statement 

Consider a 2×2 binary sudoku, with two simple rules:
* No column may contain the same value twice
* No row may contain the same value twice

Assigning each square in the sudoku to a variable: Construct a quantum circuit to output the solution for this sudoku.
<div align="center">

| __v0__ | __v1__ |
| :---: | :---: | 
| __v2__ | __v3__ |

</div>

## Problem 2: Iterative Phase Estimation
### Problem Statement
Given a unitary matrix $U$ and an eigenstate $|\Psi\rangle$ of $U$ with an unknown eigenvalue $e^{i 2\pi \varphi}$, estimate the value of $\varphi$.

Let's first assume for simplicity that $\varphi$ can have an exact binary expansion, that is, it can be written as $$\varphi = \varphi_1/2 + \varphi_2/4 + \cdots + \varphi_m/2^m = 0.\varphi_1 \varphi_2 \cdots \varphi_m$$ where we are using "decimal" point notation in base 2. Unlike the usual phase estimation algorithm where we would need $m$ auxiliary qubits, IPE requires only a single auxiliary qubit, let's call this qubit, $q$ for notational convenience.

Suppose that we initialize $q$ in the state $|+\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}$. What happens if we apply the *controlled*-$U^{2^t}$ gate, with $q$ being the control, given that the target is an eigenstate of $U$?

Let $|\Psi \rangle$ be the eigenstate of $U$ with eigenvalue $e^{i 2\pi \varphi}$, we have
$$|+\rangle |\Psi \rangle = \left(\frac{|0\rangle + |1\rangle}{\sqrt{2}}\right) |\Psi \rangle = \frac{|0\rangle |\Psi \rangle + |1\rangle |\Psi \rangle}{\sqrt{2}} \overset{C\left(U^{2^t}\right)}{\longrightarrow} frac{|0\rangle |\Psi \rangle + e^{i 2 \pi 2^{t} \varphi} |1\rangle |\Psi \rangle}{\sqrt{2}} = \left(\frac{|0\rangle  + e^{i 2 \pi 2^{t} \varphi} |1\rangle}{\sqrt{2}}\right) |\Psi \rangle.$$

The qubits on the eigenstate remain unchanged, while a phase of $e^{i 2 \pi 2^{t} \varphi}$ has been "kicked back" (similar to phase kickback in Grover's algorithm) into the state of the auxiliary qubit $q$.

Let's look at what happens for different values of $t$
- for $t=0$, the phase would be $e^{i 2 \pi 2^{0} \varphi} = e^{i 2 \pi \varphi} = e^{i 2 \pi 0.\varphi_1 \varphi_2 ... \varphi_m}$
- for $t=1$, the phase would be $e^{i 2 \pi 2^{1} \varphi}= e^{i 2 \pi \varphi_1} e^{i 2 \pi 0.\varphi_2 \varphi_3 ... \varphi_m} = e^{i 2 \pi 0.\varphi_2 \varphi_3 ... \varphi_m}$
- for $t=2$, the phase would be $e^{i 2 \pi 2^{2} \varphi} = e^{i 2 \pi 0.\varphi_3 \varphi_4 ... \varphi_m}$
- for $t=m-1$, the phase would be $e^{i 2 \pi 2^{m-1} \varphi} = e^{i 2 \pi 0.\varphi_m}$.

In the last case where $t = m - 1$, the phase is $e^{i 2 \pi 0.\varphi_m}$, which is equal to $1$ if $\varphi_m = 0$ and $-1$ if $\varphi_m = 1$.
In the first case, the auxiliary qubit $q$ would be in the state $|+\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}$, and in the second case it would be
in the state $|-\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}$. 

Therefore, measuring the qubit in the Pauli $X$ basis would distinguish these cases with a 100\% success rate.
This is done by performing a Hadamard gate on the qubit before measuring it. 

In the first case, we would measure 0 and in the second case we would measure 1; in other words, the measured bit would be equal to $\varphi_m$.

### Algorithm

1. Initialize the circuit with the eigenstate $|\Psi\rangle$.
2. Initialize the auxiliary qubit in state $|+\rangle$.
3. Perform a $controlled-U^{2^{m-1}}$ operation, and measuring $q$ in the Pauli $X$ basis. This gives us the value of the least significant bit $\varphi_{m}$.
4. Reset the auxiliary qubit using the reset gate available in qiskit.
5. For the next step apply a $controlled-U^{2^{m-2}}$ operation. The relative phase in $q$ after these operations is now $e^{i 2 \pi 0.\varphi_{m-1}\varphi_{m}}= e^{i 2 \pi 0.\varphi_{m-1}} e^{i 2 \pi \varphi_m/4}$. \
To extract the phase bit $\varphi_{m-1}$, first perform a phase correction by rotating around the $Z$-axis by an angle $-2 \pi \varphi_m/4=-\pi \varphi_m/2$, which results in the state of  $q$ to be $|0\rangle + e^{i 2 \pi 0.\varphi_{m-1}} | 1 \rangle$. Measure $q$ in the Pauli $X$ basis to obtain the phase bit $\varphi_{m-1}$. \
This is where the main advantage lies, rotating the auxiliary qubit at step $k$ based on the known information till step $k-1$.

6. Repeat the same procedure for $m$ steps to retrieve the entire phase $\varphi_1,\varphi_2,...,\varphi_m$

## Problem 3: Kushlevitz and Mansour Algorithm
### Randomized Algorithms
Randomized algorithms and probabilistic methods play a key role in modern computer science. Randomized algorithms make random choices during their execution and achieve results with certain accuracy within a reasonable time. In today's exercise, we will see how randomized algorithms can be efficiently compared to their brute-force counterparts using a well-known algorithm to learn boolean functions called the "Kushlevitz-Mansour" (KM) algorithm.

This is a primer for the classes starting from Tuesday i.e., Nov 7 where we will be looking at exploiting boolean functions, and Fourier analysis in both classical and quantum to develop new ideas for Quantum algorithmic design.

### Background
* Consider a boolean function of three inputs $x_1,x_2,x_3 \in \{-1,+1\}$ whose output is determined by the expression $f = 0.5 + 0.5x_2 + 0.5x_3 - 0.5x_2x_3$.
* In general, for a 3-variable boolean function, there are 8 (2^3) possible terms $(\phi, x_1, x_2, x_3, x_1x_2, x_1x_3, x_2x_3, x_1x_2x_3)$. But we can see that only 4 monomial terms are present for the above case. 
* These coefficients are called Fourier coefficients and the set of these coefficients is called a Fourier spectrum.
* Now, consider learning the expression when the number of variables is large, say n=200. It will require a truth table of entries 2^200 which is impossible to store in the memory let alone computing the spectrum in a brute-force approach.
* This is where randomized algorithms like Kushlevitz-Mansour come to the rescue.
* KM algorithm outputs an expression that is close to the original function, in polynomial time, given the set of Fourier coefficients is sparse. Such functions are usually referred to as *concentrated* boolean functions.


### References
  1. [LEARNING DECISION TREES USING THE FOURIER SPECTRUM](https://dl.acm.org/doi/pdf/10.1145/103418.103466)
  2. [Implementation Issues in the Fourier Transform Algorithm](https://link.springer.com/article/10.1023/A:1011034100370)
  
### Assumptions 
  1. Query access to an oracle to the target function must be available to query it for a given input.
  2. Fourier spectra of the function is concentrated.

**Principle:** The approach works by iteratively growing the strings corresponding to the monomial terms (prefix), estimating the corresponding Fourier coefficient, and thereafter pruning the strings whose Fourier coefficient is not larger than a given *threshold*. The pruning allows us to significantly reduce the number of monomial terms to be explored without compromising on the final accuracy.

>**Note: This algorithm is equivalently applicable for both Boolean Valued Boolean Functions as well as Real-Valued Boolean Functions**

### Algorithm 

The algorithm can be explained using two subroutines Coeff($\alpha$) and Approx($\alpha$) as follows\
SUBROUTINE Coeff($\alpha$)
* $\beta_\alpha$ = Approx($\alpha$)  // $\beta_\alpha$ estimates the total value of Fourier coefficients that share the prefix with $\alpha$ 
    * IF $\beta_\alpha \geq \theta^2/2$ THEN
        * IF $|\alpha| == n$ THEN Output $\alpha$
    * ELSE Coeff($\alpha 0$); Coeff($\alpha 1$) // Explore

Consider for a $l$-bit binary string $\alpha \in \{0,1\}^l$, the corresponding monomial $\chi_{\alpha} = \prod_{i}x_i^{\alpha[i]}$ and the value $\chi_{\alpha}(z) = (-1)^{\alpha.z}$ for any $z \in \{0,1\}^l$.\
SUBROUTINE Approx($\alpha$):
* Choose at random $x_i \in \{0,1\}^{n-k}$ for $1\leq i\leq m_1$
* For each $x_i$
    * Choose at random $y_{i,j} \in \{0,1\}^{k}$ for $1\leq j \leq m_2$
        * Let $A_i = \frac{1}{m_2}\sum\limits_{j=1}^{m_2}f(y_{i,j}x_i)\chi_{\alpha}(y_{i,j})$
* Compute $\beta_\alpha = \frac{1}{m_1}\sum\limits_{i=1}^{m_1}A_i^2$
* Output $\beta_\alpha$
