# PODEM-Algorithm-implementation
An ATPG tool using PODEM algorithm in C++ that generates a test to detect any given list of Single-Stuck-at Faults

 #### class Circuit
 * A circuit contains primary inputs and outputs, and a number of interconnected gates.
   
 * The API for interacting with a Circuit includes functions to access the gates in the
  system. Functions you may use in your project include getPIGates() and getPOGates(), which
  return the inputs and outputs of the circuit. Also useful are setPIValues(), which the reference
  code uses to set the values of the inputs of the circuit, and clearGateValues() which it uses
  to clear the logical values after each simulation iteration.

 * An important thing to understand: we will use a special Gate type called a PI to represent
   the circuit's primary inputs. This isn't really a logic gate, it's just a way of 
   making it easy to access the inputs. So, when you call  getPIGates() you will get a vector of
   pointers to the special PI gates.
   
 * We don't use a special structure for the primary outputs (POs); instead we simply maintain a 
  list of which gates drive the output values. So when you call getPOGates() you will get
   a vector of pointers to the normal gates that drive the outputs.
   
 * Lastly, note that there are a number of functions here that are only used when the initial 
   representation of the circuit is constructed. (This is done for you by the yyparse() function 
   (see main.cc). These functions are ones that you will never have to manipulate yourself, and 
   they are marked with a note that indicate this.
   

#### class Gate
 *  A Boolean logic gate, with pointers to its input sources and destinations of its outputs.
   
 * A Gate is a data structure for storing the information about a gate. It contains a number of 
  useful things:
 * A variable that indicates what type of gate it is. Note that this will be stored in a number. In 
   the top of ClassGate.h, you will find a list of #define macros that give names to each gate type.
   For example, we will use the value 0 to represent a NAND gate. However, instead of remembering that, you
   can simply use the macros like: "if (gateType == GATE_NAND)". If you want to get the type of a gate
   object, call the get_gateType() function.
    
 * One important thing to note: if this gate represents a primary input (PI) of the circuit, then
   the gate will be of type GATE_PI.
   
 * Vectors of pointers to this gate's predecessors and successors. The predecessors are the gates 
   whose outputs are the inputs to this gate. (If this gate is a PI, then it has no predecessors.)
   The successors are the gates that take in the input of this gate. (If this gate drives a PO, then
   it has no successors. You can access these vectors by running the  get_gateInputs() and
   get_gateOutputs() functions.
   
 *   A logical value that indicates the output value of the gate in the current simulation. Currently,
    legal values are, 0, 1, D, B ("not D"), X, and "unset." The "unset" value indicates that the 
    value is unknown because it has not yet been computed. The X value indicates a true "unknown" 
    input value. You can get the value of a gate's output by using the getValue() function, and 
    set it using the setValue() function. 
    
 *   These values are encoded as numbers, but (like the gate type) you will use macros defined at the
     top of ClassGate.h (instead of hard-coding the numbers into the system). For example, to set the
     value of a gate to 1, you would call setValue(LOGIC_ONE).
 

 

#### Part 1: 
    - Writing the getObjective() function
    - Writing the updateDFrontier() function
    - Writing the backtrace() function
    - Writing the podemRecursion() function
 Then basic PODEM implementation should be finished.

 turning on a "checkTest" mode that will use your 
 simulator to run the test after you generated it and check that it
 correctly detects the faults.

#### Part 2:
    - Writing the eventDrivenSim() function.
    - Change the simulation calls in your podemRecursion() function
      from using the old simFullCircuit() function to the new
      event-driven simulator.
 Then, PODEM implementation should run considerably faster
 (probably 8 to 10x faster for large circuits).
 Test everything and then evaluate the speedup.
 A quick way to figure out how long it takes to run something in
 Linux is:
   time ./atpg ... type other params...


#### SUMMARY-

Test vector generation for Stuck at 1 and Stuck at 0 faults can be done using the PODEM algorithm. It consists of four basic steps :1) OBJECTIVE, 2)BACKTRACE, 3)IMPLICATION, 4)D-FRONTIER. Initially we parse the input file and set up the circuit, then we setup the output file, after which we open the fault file. We start by setting up the fault and initialize the D-frontier. After the fault is activated we will use the D-frontier to get an objective. After the objective gate and objective value is set, use the backtrace function to figure out which input to set it to, then determine the implications of the input you set by simulating the circuit by calling simFullCircuit function.
Podem recursion is the main function which calls the backtrace, get objective and update d-frontier function, simFullCircuit and setValueCheckFault functions. Heuristics like Scoap Measurability, helped optimize the algorithm by choosing the D-frontier optimally. Another optimization in this circuit is carried out via the function eventDrivensim. This helps us optimize as each time PODEM makes a decision, it changes one input from X to 0 or 1. In this case  by only stimulating things that are changed and by not stimulating the entire circuit we can optimize it 5x to 10x.

#### Other functions used are->

#### Helper functions
vector<char> constructInputLine(string line)- Used to parse in the values from the input file.

void printUsage();

vector<char> constructInputLine(string line)- Used to parse in the values from the input file.

bool checkTest(Circuit* myCircuit); This function gets called after your PODEM algorithm finishes.
 If you enable this, it will clear the circuit's internal values,
  and re-simulate the vector PODEM found to test your result.
 
string printPIValue(char v);a helper function used when storing the final test you computed.

#### Functions for logic simulation

void simFullCircuit(Circuit* myCircuit); set all non-input gates to LOGIC_UNSET
 and call the recursive simulate function on all output gates.

void simGateRecursive(Gate* g); Recursive function to find and set the value on Gate* g.
This function calls simGate and setValueCheckFault.  This function prepares Gate* g to to be simulated by recursing
 on its inputs (if needed). Then it calls  simGate(g) to calculate the new value.  Lastly, it will set the Gate's output value based on the calculated value.


void eventDrivenSim(Circuit* myCircuit, queue<Gate*> q); This function takes as input the Circuit* and a queue<Gate*> indicating the remaining gates that need to be evaluated.  The basic idea behind this is to keep a queue of gates whose output value may potentially change while there are still gates in the queue. 

char simGate(Gate* g); This is a gate simulation function , it will simulate the gate g
 with its current input values and return the output value.

char evalGate(vector<char> in, int c, int i); Evaluate a NAND, NOR, AND, or OR gate.
 in-The logic value of this gate's inputs.
 c- The controlling value of this gate type (e.g. c==0 for an AND or NAND gate)
 i- The inverting value for this gate (e.g. i==0 for AND and i==1 for NAND)
 * \returns The logical value produced by this gate (not including a possible fault on this gate).

char EvalXORGate(vector<char> in, int inv);  Evaluate an XOR or XNOR gate.
 in- The logic value of this gate's inputs.
 inv- The inverting value for this gate (e.g. i==0 for XOR and i==1 for XNOR)
 * \returns The logical value produced by this gate (not including a possible fault on this gate).

int LogicNot(int logicVal); Perform a logical NOT operation on a logical value using the LOGIC_* macros

void setValueCheckFault(Gate* g, char gateValue);Set the value of Gate* g to value gateValue, accounting for any fault on g.

#### Functions for PODEM:

bool podemRecursion(Circuit* myCircuit);
the main recursive  algorithm.  This  function  calls  the  backtrace(),  getObjective(),  setValueCheckFault(), and simFullCircuit() functions. 

bool getObjective(Gate* &g, char &v, Circuit* myCircuit); We use the D-frontier to find an objective, therefore first we update the D-frontier, if the D-frontier is empty we return false.
PARAMETERS-
 g- Use this pointer to store the objective Gate your function picks.
 v- Use this char to store the objective value your function picks.
 *  \returns True if the function is able to determine an objective, and false if it fails.

void updateDFrontier(Circuit* myCircuit); to execute this, clear the D-frontier vector,  stored as a global variable, loop over all the gates in the circuit and check if it can be a D-frontier if so, add it to the vector.

void backtrace(Gate* &pi, char &piVal, Gate* objGate, char objVal, Circuit* myCircuit); given objective objGate and objVal, then figure out which input to set and which value to set it to.
PARAMETERS-
pi Output: A Gate pointer to the primary input your backtrace function found.
piVal Output: The value you want to set that primary input to
objGate Input: The objective Gate (computed by getObjective)
objVal Input: the objective value (computed by getObjective)



#### Global Variables used are

vector<Gate*> dFrontier;: a vector of Gate pointers for storing the D-Frontier. 

Gate* faultLocation;    : holds a pointer to the gate with stuck-at fault on its output location. 
 
char faultActivationVal;: holds the logic value you will need to activate the stuck-at fault. 

