# Chapter 1 - Verification Guidelines

> The most important principle you can learn as a verification engineer
> is: "Bugs are good." ... Each bug that can be found before tape-out is
> one fewer that ends up in the customer's hands.
> (pg. 1)

Typical features that are unique to hardware verification languages 
(pg. 2):

- Constrained-random stimulus generation
- Functional coverage
- Parallel computing
- HDL types 
- Tight integration with event-simulator
- Higher-level structures (maybe not synthesizable?)

The goal of verification is ensure that a "design is an accuracte
representation of the specification." (pg. 2)

Hierarchy of test levels (pg. 3):

1. Block level tests: looking for bugs in a module
2. Integration phase: looking for "discrepancies at boundaries between 
   blocks."
3. Design Under Test (DUT): testing the entire system.

All tests should perform "interesting activities", meaning that edge cases
and real-world use cases should be simulated in order to try and detect 
bugs in the design. (pg. 3)

Also, after checking the the DUT performs correctly, a verification engineer
will inject errors to see how the DUT responds to errors when they do occur.
(pg. 3)

VMM
: Verilog Methodology Manual for SystemVerilog (index)

OVM / UVM 
: Methodologies for verification of hardware designs (pg. 4)

CRT 
: Constrained-Random Tests (pg. 4)

Basic Testbench Functionality (pg. 5):

- Generate stimulus
- Apply stimulus to DUT 
- Capture reponse 
- Check for correctness 
- Measure progress against verication goals

Directed Testing 
: A testing approach where the verification plan includes a list of tests 
  where each test is concentrated on a specific feature. You then tackle 
  each test sequentially, gradually covering 100% of the desired tests over 
  time. (pg. 5-6)

Brute force typically will not work for hardware designs. If you are 
designing a 32-bit adder for example, there are $$ 2^64 $$ possible inputs 
to the adder, and running a testbench for that many inputs would take 
wayyyyy too much time. (pg. 6)

Random testing with functional coverage tools and a modular testing framework 
takes longer to set up than a directed test, but will produce quicker results
once set up and completed. (pg. 7)

Random testing _also_ can find bugs that you may not have expected, while 
directed testing is likely to only find the bugs that you did expect.
(pg. 7)

**A random testbench can be turned into a directed testbench, but a directed
testbench cannot be turned into a random testbench.** (pg. 7)

Constrained-random stimulus allows us to _constrain_ the random input to a 
subset that we actually want to consider. (pg. 8)

```systemverilog 
$random() // generates a random value (non-synthesizable)
```

Most missed bugs come because not enough configurations are tested. "Most 
tests just use the design as it comes out of reset," which is like testing 
an operating system without any applications installed or files created.
(pg. 9)

ATM
: Asynchronous Transfer Mode (pg. 119)

In parallel random testing you need to consider how to generate unique seeds
for each test. The book gives the example using time of day along with CPUID
and the process ID for generate unique seeds for each test running on 
multiple machines with multicore processors. (pg. 11)

Random tests using functional coverage can utilize a feedback loop where 
everytime a test is run coverage is determined, and the random inputs are 
improved to cover more test cases. This improves simulation time. (pg. 12)

Dynamic feedback in your testbench (where the testbench automatically 
modifies inputs to obtain better coverage) are challenging to implement on 
real designs, so often the random test / functional coverage feedback loop 
is done manually by a verification engineer. (pg. 13)

Some formal verification is done by tools to do dynamic feedback, such as 
Synopsis' Magellan which generates a state space, then checks how many 
states your test reaches, then changes inputs to reach more states. How this 
works in a method that is more effecient than just searching the statespace 
for correctness? I'm not actually sure. 

BFM 
: Bus Functional Models 

For pure simulation testbenches, non-synthesizable testbench components
(BFMs) are acceptable, while if you are using an FPGA to test, you need 
synthesizable BFMs.

Layered Testbenches 
: Layered testbenches compose together separate testbench components in 
  order to provide code reusablility, as well as the ability to better 
  reason about the testbench itself. Basically, this is an implementation 
  of abstraction for testbenches. (pg. 14-17)

A basic layered testbench (pg. 17-19):

1. Signal layer: signals are connected into the inputs and outputs of 
   the DUT, and assertions are rigged up to check correct function of 
   inner components of the DUT.
2. Command layer: 
   - Driver: provides basic functions for the DUT, abstracting away the 
     actual signal assignments for an operation to take place. 

     For example, the driver may provide `read` and `write` tasks that 
     abstract away the specific signals that need to be raised to accomplish
     those operations.
   - Assertions: check correctness of signals in the DUT.
   - Monitor: Takes signal transistions and groups them together into command 
     outputs? Not exactly clear on what this does.
3. Functional Layer:
   - Agent: receives high level transaction requests and translates them into
     sequences of commands. The commands are also send to the _scoreboard_.

     AKA: the _transactor_
   - Scoreboard: Receieves commands from the agent and predicts output. I 
     think that this is like the "golden" or "reference" model.
   - Checker: compares the output from the scoreboard and the monitor 
     (in the command layer) to see if the operation is correct.
4. Scenario Layer: This layer simulates the operations of scenarios that we
   expect our device to be able to handle.
5. Test layer: the top level of the testbench that evaluates coverage.

Testbench Environment
: The layers and blocks of the testbench environment are written once, at 
  the beginning of the project and shouldn't change too much. They are 
  dynamically used by different DUTs by injecting different behavior into 
  each block. (pg. 18)

> Do you need all these layers in your testbench? ... **A complicated 
> design _requires_ a sophisticated testbench.**
> (pg. 19)

```systemverilog
// basic transactor code 
task run();
    done = 0;
    while (!done) begin 
        // Get the next transaction
        // Make transformations
        // Send out transactions
    end 
endtask
```

