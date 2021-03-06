# Macho - C++ Machine Objects

The Machine Objects class library (in short Macho) allows the creation of
state machines based on the "State" design pattern in straight C++. It
extends the pattern with the option to create hierarchical state machines,
making it possible to convert the popular UML statechart notation to working
code in a straightforward way. Other features are entry and exit actions,
state histories and state variables.

Copyright (c) 2005 by Eduard Hiti (feedback to macho@ehiti.de)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

You are encouraged to provide any changes, extensions and corrections for
this software to the author at the above-mentioned email address for
inclusion into future versions.


Description:

States are represented as C++ classes. The hierarchy of states follows the
inheritance relation between these state classes. A set of state classes for
a single state machine derives directly or indirectly from a top state class,
making it the composite state holding all other states. Events are processed
by calling virtual methods of the top state class. Substates redefine the
behaviour of these event handler methods.

Special methods "entry", "exit" and "init" are called on state entry, state
exit and state initialization of super- and substates (in the order defined
by statechart semantics and current machine state).

An object of type "Machine" maintains the current state of a state machine
and dispatches events to it. The "Machine" type is a template class
parametrized with the top state class of the state machine to be run.

State data is not kept in state classes (because state class instances are
created just once and then reused, whereas state data should be instantiated
or destroyed each time its state is entered or left). State data is put in
"Box" types specific to each state class instead, which are managed by the
Machine object. Boxes are retrieved by calling the "box" method.
Superstate boxes are accessible by qualifiying the "box" method with the
state class name (e.g. TOP::box()).

A history of entered substates can be kept for superstates. With a special
transition into the superstate the history substate can be reentered. History
can be shallow (only direct substates) or deep (any substate).


Example:

```
#include "Macho.hpp"
#include <iostream>
using namespace std;

namespace Example {
	TOPSTATE(Top) {
		struct Box {
			Box() : data(0) {}
			long data;
		};

		STATE(Top)

		virtual void event1() {}
		virtual void event2() {}

	private:
		void entry();
		void exit();
		void init();
	};

	SUBSTATE(Super, Top) {
		STATE(Super)
		HISTORY()

	private:
		void entry();
		void exit();
	};

	SUBSTATE(StateA, Super) {
		struct Box {
			Box() : data(0) {}
			int data;
		};

		STATE(StateA)

		void event1();

	 private:
		void entry();
		void exit();
		void init(int i);
	};

	SUBSTATE(StateB, Super) {
		STATE(StateB)

		void event2();

	private:
		void entry();
		void exit();
	};

	void Top::entry() { cout << "Top::entry" << endl; }
	void Top::exit() { cout << "Top::exit" << endl; }
	void Top::init() { setState<StateA>(42); }

	void Super::entry() { cout << "Super::entry" << endl; }
	void Super::exit() { cout << "Super::exit" << endl; }

	void StateA::entry() { cout << "StateA::entry" << endl; }
	void StateA::init(int i) { box().data = i; }
	void StateA::exit() { cout << "StateA::exit" << endl; }
	void StateA::event1() { setState<StateB>(); }

	void StateB::entry() { cout << "StateB::entry" << endl; }
	void StateB::exit() { cout << "StateB::exit" << endl; }
	void StateB::event2() { setState<StateA>(); }
}

int main() {
	Macho::Machine<Example::Top> m;
	m->event1();
	m->event2();

	return 0;
}

```

Output is:

Top::entry

Super::entry

StateA::entry

StateA::exit

StateB::entry

StateB::exit

StateA::entry

StateA::exit

Super::exit

Top::exit


Version History:

	  0.9.8 (released 2017-03-26):
		 - Implemented parameter packs to allow for any number of parameters

	  0.9.7 (released 2007-12-1):
		 - Introduction of template states
		 - fixed rare memory leak

	  0.9.6 (released 2007-09-01):
		 - Changes to state transition semantics (see file "changes_0_9_6.txt")
		 - New mechanism for state initialization
		 - Runtime reflection on state relationships now possible

	  0.9.5 (released 2007-05-01):
		 - Introduction of parametrized state transitions

	  0.9.4 (released 2006-06-01):
		 - Snapshot functionality added

	  0.9.3 (released 2006-04-20):
		 - Code reorganization (file Macho.cpp added)

	  0.9.2 (released 2006-04-10):
		 - Memory leak plugged
		 - MSVC6 version updated

	  0.9.1 (released 2006-03-30):
		 - Introduction of persistent boxes
		 - Speed and size optimizations
		 - Machine instance can be accessed in event handlers with method "machine"

	  0.9 (released 2006-01-15):
		 - Introduction of queuable event type

	  0.8.2 (released 2005-12-15):
		 - Code size reduction by minimizing use of template classes

	  0.8.1 (released 2005-12-01):
		 - Added MSVC6 variant (see directory "msvc6")
		 - Added method "clearHistoryDeep"

	  0.8 (released 2005-11-01):
		 - Initial release
