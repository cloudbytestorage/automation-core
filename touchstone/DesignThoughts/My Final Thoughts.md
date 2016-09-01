## The demo day

This was the day, when I showcased this scripting DSL that I had been building.
There was one brilliant question from one of my colleagues.

> "We created a sort of DSL which looks similar to Python".

What's great about it ? Why reinvent the wheel.

### My explanation

Well, there are many languages which have less learning curve or a syntax which
can be learned in hours. Python stands out in this category & hence it has usage 
from academia to industry to mobile to IOT.

Now lets try to think from Real World perspective:

- We write hundreds of thousands of code in Python in our controller
- There are huge no of if else statement
- Lots of exception handling
- Dangerous no of debug statements
- We do the dirty parsing logic in our Py code with good no of do-while, etc. loops.
- We need to learn multiples of design patterns to make the code hardened.

I believe, these are the signs of imperative programming.

Here comes the question:
- Can we remove this boilerplate & verboseness & yet write hardened logic in Python ?
- We don't want our support, qa, customer to learn this tricks to write telecom grade Py code ?
- Can we code the use-cases in minutes ?

The answer lies in :
- Creating wrappers in whatever language.
- However these wrappers should not consist of monstrous amount of LOC
  - leading to unit test cases & release management, & this & that...
- Groovy helped me to create a scoped syntaxed language by 
  - taking care of non-functional requirements.
  - with some 200 odd LOC.
- Second & best solution will be to:
  - Move away from imperative language to a functional language.
  - This is a bigger topic & you can get more on it in Internet.
