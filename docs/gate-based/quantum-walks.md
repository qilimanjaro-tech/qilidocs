# Quantum Walks

## Summary classical random walks

A random walk is a random process involving a "walker" that is placed in some n-dimensional medium, like a grid or a graph. We then repeatedly query some random variable, and based on the outcome of our measurement, the walker's position vector (position on the graph or grid) is updated. A basic example of a random walk is the one-dimensional graphical case, where we consider a marker placed on the origin of a number line with markings at each of the integers. Let the initial position vector of our marker be $\ket{0}$. For  $N$ steps of our random walk, take a set of  $N$ random variables ${X_1, …, X_N}$, which can take on either a value of 1 or -1 with equal probability. To find the updated position vector of our walker, we compute the value:

$$j = \sum_{k=1}^{N}X_{k}$$

Where we know:

$$\ket{Final} = \ket{Initial + j}$$


So for our case, the final position vector is $\ket{j}$.  For a discrete, 1-dimensional random walk on a number-line-like graph, the probability of the random walker being at a specific location follows a binomial distribution. This binomial distribution arises from the behavior of the walker through the 1-dimensional graph which is modeled by the following distribution:

$$P_{N}(X) \ = \ \begin{pmatrix} N \\ R \end{pmatrix} \ p_{r}^R (1 \ - \ p_{r})^{N \ - \ R} \Rightarrow \ X \ = \ R \ - \ L \ \Rightarrow \ P_{N}(X) \ = \ \begin{pmatrix} N \\ \frac{N \ + \ X}{2} \end{pmatrix} \ p_{r}^{\frac{N \ + \ X}{2} } (1 \ - \ p_{r})^{\frac{N \ - \ X}{2} }$$

Where $P(X)$ is the probability of the walker to end up in position $X$, $N$ is the number of total steps taken, $p_r$ is the probability of taking a step to the right and $R$ is the total number of steps taken to the right. If we start from the zero position after many steps, the probability distribution of finding ourselves in a given new position is represented by Figure 1.

![binomial distribution](img/binomial%20distribution.png#center)   
***Figure 1.** Binomial distribution for a classical random walk on a 1-dimensional graph.*

## Quantum random walks

The process of the quantum walk isn't that much different from its classical counterpart, although the observed results of the two processes have many differences. First, let us motivate the creation of a QW. The idea is that when one performs analysis on a classical random walk, you can find that  $\sigma^2 \ \sim \ T$, where $\sigma$ is the standard deviation of the random walk's probability distribution, and $T$ is the number of time-steps of the random walk. For the quantum walk, we can see that $\sigma^2 \ \sim \ T^2$. In other words, the standard deviation grows at a quadratically faster rate. At a high level, this signifies that the quantum walker "spreads out" quadratically faster than the classical one, showing that the process of a QW is quadratically faster than its classical counterpart.

## Quantum Walks for 1-dimensional ring graph

The objective is to translate the elements present in the classical random walk into the quantum formalism. Given a ring graph for a dimension we want to simulate the classical walk but with the elements of the quantum formalism.

![ring graph](img/ring%20graph.png#center)  

***Figure 2.** 1-dimensional ring graph.*

Firstly we need to encode the position or the node of the network we are in, but this is quite easy to do, for a $k$-node network we can form a Hilbert space $H_W$ given by:

$$H_W \ = \ \{\lvert j\rangle \ : \ j \ = \ 0, \ ..., \ K \ - \ 1 \}$$

We also require another vector in order to create a random walk. We need a "coin vector", which will encode the direction in which the random walk will progress at the $T$-th step of the process. This Hilbert space is spanned by the two basis states, representing forward and backward progression on our number-line-like graph (actually, our graph looks more like a ring, so the two basis states will represent clockwise and counter-clockwise motion, but it's the same idea). We will call this Hilbert space $H_C$, and we can again define our spanning set:

$$H_C \ = \ \{\lvert i\rangle \ : \ i \ = \ \downarrow, \ \uparrow\rangle\}$$

Where the upward-arrow symbol represent counter-clockwise motion, and the downward arrow represents clock-wise motion. Now that we have defined all the vectors we need to encode the information about our random walk, we must understand how we can realize these vectors in our quantum algorithm. Well, this is again fairly simple. For a graph of $K = 2^n$  nodes, we require $n$ qubits to encode binary representations of numbers ranging from $0$ to $K-1$, therefore each of the vectors spanning $H_W$ will be given by the binary representation of $j$ corresponding to the basis vector $\ket{j}$. For the coin vector, since we have only two states, we only need one qubit to encode the two possible states:

$$\lvert 0\rangle \ = \ \lvert \uparrow\rangle \ \ \text{and} \ \ \lvert 1\rangle \ = \ \lvert \downarrow\rangle$$

In order to represent the total space of all possible states of our system, we take the tensor product of the two spanning sets, which will then span the new Hilbert space $H_C \otimes H_W$ . We will write a general element of this Hilbert space as $\ket{i} \otimes \ket{j}$. We define a random walk evolution operator as follows:

$$U \ = \ \lvert \uparrow\rangle\langle\uparrow\lvert  \ \otimes \ \displaystyle\sum_{j} \ \lvert j \ + \ 1\rangle\langle j\lvert  \ + \ \lvert \downarrow\rangle\langle\downarrow\lvert  \ \otimes \ \displaystyle\sum_{j} \ \lvert j \ - \ 1\rangle\langle j\lvert$$

We also need to define a coin operator that gives the probabilities of moving in one direction or the opposite direction. For the coin operator we define: 

$$U_C = H$$

The choice of the Hadamard gate is a natural choice for implementing a fair coin operator. One step within the network is given by the circuit shown below:

![circuit quantum walks](img/circuit_quantum_walks.png#center)
***Figure 3.** Quantum walks circuit.*

The circuit shown is equivalent to the application of operator:

$$S \ = \ U \ (H \ \otimes \ I)$$

## Python code experiment

Required libaries

``` py
from qibo import gates, models
from matplotlib import pyplot as plt
```

We define the operator that implements a quantum walk step which is a combination of the addition operator and the subtraction operator:

``` py
def walk_step_qibo(circuit, number_qubits):
    # "Flip" the coin vector
    circuit.add(gates.H(number_qubits))

    # Implement the Addition Operator
    circuit.add(gates.X(number_qubits))

    for i in range(number_qubits, 0, -1):
        controls = list(range(number_qubits, i-1, -1))
        circuit.add(gates.X(i-1).controlled_by(*controls))
        if i > 1:
            circuit.add(gates.X(i-1))

    circuit.add(gates.X(number_qubits))

    # Implement the Subtraction Operator
    for i in range(1, number_qubits + 1):
        controls = list(range(number_qubits, i-1, -1))
        circuit.add(gates.X(i-1).controlled_by(*controls))
        if i < number_qubits:
            circuit.add(gates.X(i))

    return circuit
```

We set the initial state associated with the node of the output graph 

``` py
def initial_state_qibo(circuit, number_qubits):
    # Initial node
    circuit.add(gates.X(1))
    
	  # Initial state for coin operator
    circuit.add(gates.H(number_qubits))
    circuit.add(gates.S(number_qubits))
```

We implement the quantum walk algorithm as the n-times application of the step operator on the initial state

``` py
def generate_walk_qibo(number_qubits, iterator, sample_number):
    circuit = models.Circuit(number_qubits + 1)
    initial_state_qibo(circuit, number_qubits)

    for _ in range(iterator):
        walk_step_qibo(circuit, number_qubits)  

    for q in range(number_qubits):
        circuit.add(gates.M(q))

    result = circuit(nshots= sample_number)
    final = result.frequencies(binary=True)
    return final
```

The final measured distribution represents the final probability of being at each node after n steps

``` py
number_nodes = 7
steps = 30
sample_number = 5000


final = generate_walk_qibo(number_nodes, steps, sample_number)
```   

![generate walks](img/generate-walks.png#center)