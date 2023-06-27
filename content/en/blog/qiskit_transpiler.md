---
title: "Understanding Qiskit Transpiler"
date: 2023-06-27T11:09:01+09:00
author: "Pawan Poudel"
description: "What is a Qiskit Transpiler? Making things easier..."
tags: ["qiskit","transpiler"]
---

Transpiler is a tool to convert a programming language at the input to the other programming language at the output.

So, is compiler a transpiler?

No, not necessarily!

A Compiler converts high-level code into lower-level instructions at different abstraction levels. On the other hand, a transpiler converts a high-level code into another high-level code at the same level of abstraction. 

(Note: It is not necessary that the languages be high-level; these can be low-level to low-level as well.)

To differentiate a transpiler from a compiler, we may take an example,

![Trasnspiler-Compiler differentiation](/static/ibm_qc.png)


So, why do we need a transpiler if the level of abstraction is not meant to change?


Let’s get things simpler,

Suppose you are a JavaScript developer. 

Like other programming languages, JS also has its standardized ways of writing codes; ES5(ECMA Script5) and ES6(ECMA Script6).

Though both are standards for JS, they differ from each other in terms of how variables are defined, objects are defined, merged, and so on. So, it is obvious that you encounter a problem when you want to deploy code written in ES6 standard on a platform with ES5 support.

Then what next?

This is when a transpiler comes in handy.

You use a transpiler to convert ES6 standard code to ES5 and let it deploy.


Transpiler also makes it way easier for a developer to migrate a code written earlier to more latest standards and make it compatible with the updated versions.


A short capsule before entering the next segment,

*Existing IBM QX (IBM Quantum Composer) Architecture lacks abstraction layers.*

Then what does a Qiskit Transpiler do?


### Qiskit Transpiler

Qiskit transpiler documentation defines the process of transpilation as,

*“The process of rewriting a given input circuit to match the topology of a specific quantum device, and/or to optimize the circuit for execution on present-day noisy quantum systems.”*

Transpiler in Qiskit is an SDK that takes in an abstract quantum circuit set to run and converts it into a circuit that runs on a physical backend.

To be specific, it is a central component of Qiskit terra; designed for modularity and extensibility.

The process of transpilation is a stochastic process. i.e., we can get different results for the same input at different times. To guarantee the same output, qiskit transpiler has *seed_transpiler*, which can be set at different values.

![Stages in a qiskit transpiler](image.png)
(Source: https://qiskit.org/documentation/locale/ja_JP/apidoc/transpiler.html#overview )

The figure above shows the steps that the qiskit transpiler takes to convert the abstract input circuit into a physically realizable resulting circuit.

These stages are high-level stages in the trasnspilation pipeline that we can manipulate. Proper manipulation of each of these stages can create an optimal circuit.

    Before diving into the stages of preset pass manager, 
    lets have a short look at what the pass and pass managers refer to.

    Pass:
    Pass is a transformation function of a circuit that performs a small, well-defined action and combines with other passes to undergo an aggressive optimization of quantum circuits.
    All transpiler passes in Qiskit are available at qiskit.transpiler.passes

    Pass Manager:
    The way which pass is combined with which pass and by what way is determined by a pass manager. 
    It schedules the passes and allows passes to communicate with each other providing a shared space. 

More information on these topics can be obtained [here](https://qiskit.org/documentation/stable/0.24/tutorials/circuits_advanced/04_transpiler_passes_and_passmanager.html)

As shown above, the preset pass manager in transpiler has the following stages:

#### 1.	Init

This stage involves the process of unrolling the instructions and converting the circuit such that only one and two qubit-gates are present.
Basically, the conversion of originally present gates into simpler ones is the prerequisite to mapping the logical gates to the physical ones. This is described in the next step.


#### 2.	Layout

For a brief and easier understanding, this stage is responsible for mapping the virtual qubits one-to-one to the physical qubits present at the backend of the circuit.

The mapping process depends upon which optimization level you are choosing and the target backend you are using. 

At this stage, it should be taken in mind that the efficiency of the circuit is highly dependent on the initial layout. So, it should be considered with more importance. The best layout gives the least number of swap operations and minimizes the loss that gets introduced due to the non-uniform noise properties across the device.

To find the best layout among the many possible, preset pass managers undergo two steps;

i. Find a layout that doesn’t involve any swaps; which can be the perfect layout.

ii. If the perfect layout is not found previously, use a heuristic method.

More about the insides of these steps is described in [Qiskit Transpiler documentation](https://qiskit.org/documentation/locale/ja_JP/apidoc/transpiler.html#layout-stage).  
![Circuit layout](image.png)
(Source: https://qiskit.org/documentation/locale/ja_JP/apidoc/transpiler.html#layout-stage)


#### 3.	Routing

It is important that once the layout has been decided, for those qubits not connected with each other in the layout, a technique is required to apply gates that require input from those unconnected qubits. This is done by using multiple swap gates in such a way that the two required qubits finally come alongside adjacent to each other and the gates be applied. 

The introduction of swap gates involves the introduction of more noise to the circuit. So, the deal is to make the number of swap gates as small as possible.

This optimal swap mapping is a difficult problem in computation and is regarded as one in the class of NP-hard problems. For this solution, Qiskit by default uses SabreSwap; a stochastic heuristic algorithm.
The number of trials in SabreSwap can be adjusted in Preset Pass Managers.

More information on how the optimal number of swaps is determined can be found at [Qiskit Transpiler documentation](https://qiskit.org/documentation/locale/ja_JP/apidoc/transpiler.html#routing-stage).


#### 4.	Translation
Not all gates are supported by quantum devices in the backend. However, we do not restrict ourselves to gates while writing a quantum program. So, it is for sure that the gates that we introduce in the circuit have to be converted into their equivalent simpler gates that are contained in the quantum devices. 

This process is executed in the translation stage.

In qiskit, we can query about what basic gates are present in the quantum device in the backend by querying *Target*.


#### 5.	Optimization

This loop runs multiple times until a certain required depth is achieved. Optimization is also involved in finding the best initial layout for the circuit. 

Since the swap gates are converted into simpler gates in the earlier stage, the optimization stage plays a role in reducing the depth of the circuit which might have increased due to the introduction of new basic gates. In some cases, the depth may get reduced, but there are those cases as well when there is no significant reduction in the depth of the circuit.


#### 6.	Scheduling

This stage introduces delay into the thus obtained circuit to account for all the idle time in the circuit. To get a very basic insight into what scheduling does, it introduces a delay before and after a gate is applied to a qubit and expects no further manipulation of that qubit.

When a gate is applied to a qubit in a circuit and no other gates can be applied to other remaining qubits (for cases when the input to other gates depend on the result at the output of that previous gate), the delay is introduced to the remaining qubits till no gate has been introduced to them.

More about the scheduling stage is described in [Qiskit Transpiler documentation](https://qiskit.org/documentation/locale/ja_JP/apidoc/transpiler.html#scheduling-stage).

