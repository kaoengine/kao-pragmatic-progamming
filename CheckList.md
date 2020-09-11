## CheckList

### Languages To Learn
Tired of C, C++, and Java? Try the following languages. Each of these languages has different capabilities and a different "flavor." Try a small project at home using one or more of them.
- CLOS
- Dylan
- Eiffel
- Objective C
- Prolog
- Smalltalk
- TOM

### The WISDOM Acrostic
- **W**hat do you want them to learn?
- What **i**s their interest in what you've got to say?
- How **s**ophisticated are they?
- How much **d**etail do they want?
- Whom do you want to **o**wn the information?
- How can you **m**otivate them to listen to you?
 
### How to Maintain Orthogonality
- Design independent, well-defined components.
- Keep your code decoupled.
- Avoid global data.
- Refactor similar functions.

### Things to prototype
- Architecture
- New functionality in an existing system
- Structure or contents of external data
- Third-party tools or components
- Performance issues
- User interface design

### Architectural Questions
- Are responsibilities well defined?
- Are the collaborations well defined?
- Is coupling minimized?
- Can you identify potential duplication?
- Are interface definitions and constraints acceptable?
- Can modules access needed data – when needed?

### Debugging Checklist
- Is the problem being reported a direct result of the underlying bug, or merely a symptom?
- Is the bug really in the compiler? Is it in the OS? Or is it in your code?
- If you explained this problem in detail to a coworker, what would you say?
- If the suspect code passes its unit tests, are the tests complete enough? What happens if you run the unit test with this data?
- Do the conditions that caused this bug exist anywhere else in the system?

### Law of Demeter for Functions
An object's method should call only methods belonging to:

- Itself
- Any parameters passed in
- Objects it creates
- Component objects

### How to Program Deliberately
- Stay aware of what you're doing.
- Don't code blindfolded.
- Proceed from a plan.
- Rely only on reliable things.
- Document your assumptions.
- Test assumptions as well as code.
- Prioritize your effort.
- Don't be a slave to history.

### When to Refactor
- You discover a violation of the DRY principle.
- You find things that could be more orthogonal.
- Your knowledge improves.
- The requirements evolve.
- You need to improve performance.

### Cutting the Gordian Knot
When solving _impossible_ problems, ask yourself:

- Is there an easier way?
- Am I solving the right problem?
- Why is this a problem?
- What makes it hard?
- Do I have to do it this way?
- Does it have to be done at all?

### Aspects of Testing
- Unit testing
- Integration testing
- Validation and verification
- Resource exhaustion, errors, and recovery
- Performance testing
- Usability testing
- Testing the tests themselves