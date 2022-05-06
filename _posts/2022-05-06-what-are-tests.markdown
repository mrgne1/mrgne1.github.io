---
layout: post
title: "What the H3%@ are Tests?"
categories: [programming, unit tests, testing, semiconductor, random thoughts]
assets: "/assets/what-are-tests"
mathjax: true
# post_series: "angular-plotly"
# repo_url: "https://github.com/mrgne1/mocking-the-backend"
# repo_name: "Mocking the Back-end"
---

If you've been programming for any length of time you will have heard of unit tests. You extract a piece of your code, something small and cohesive. It should also be comprehensible (i.e. you know what it should do). Then you turn that piece of code into a self-contained unit. This unit has the characteristic that it can be run independent of the rest of your code base.

Once your code has been transformed into this self-contained unit you can throw a bunch of inputs into it and see what happens. Most importantly you can compare what happens to what you thought would happen. If you happened to write code to apply the inputs and then code that would log mismatches between actual behavior and expected behavior you have just created a unit test. Simple as.

My first experience with a unit test was in my image processing class. The class's content mostly boiled down to writing Java to manipulate matrices. For example we would compute the convolution of a vertical edge detection filter (small matrix) with an image (big matrix). The output of this convolution should indicate where vertical edges were in the image. Visually this looks like:

![example of image convolution](\assets\what-are-tests\convolution_example.png)

Since this is a vertical edge detection filter all of the horizontal edges are supressed in the output and the vertical edges are bounded by a bright line and a dark line. Cool. 

In the image processing class our homework assignment would be something like: implement convolution to compute the vertical edge detection. If we were allowed to program in Python the results might look like:

{% highlight Python %}
import numpy as np

def convVEdgeDetection(image):
    filter = np.array([
        [1/3, 0, -1/3],
        [1/3, 0, -1/3],
        [1/3, 0, -1/3],
    ])
    J, I = image.shape
    output = np.zeros(image.shape)
    for j in range(J):
        for i in range(I):
            output[j, i] = image[j - 1, i - 1] * filter[0, 2] \
                         + image[j, i - 1] * filter[1, 2] \
                         + image[(j + 1) % J, i - 1] * filter[2, 2] \
                         + image[j - 1, i] * filter[0, 1] \
                         + image[j, i] * filter[1, 1] \
                         + image[(j + 1) % J, i] * filter[2, 1] \
                         + image[j - 1, (i + 1) % J] * filter[0, 0] \
                         + image[j, (i + 1) % J] * filter[1, 0] \
                         + image[(j + 1) % J, (i + 1) % J] * filter[2, 0]
    return output

output = convVEdgeDetection(image)

{% endhighlight %}

This code is horrible. I mean, what is it doing? If you don't know what convolution is you're lost, if you know convolution, but not image convolution you're really lost and if you know image convolution you're just mad. I'm not going to tell you what it is doing. If you really want to know look up 2d image convolution. What I will focus on is that this code has a lot that can go wrong. There are 9 multiplications with 18 different values taken from two different matrices and these values are indexed in weird ways (`(j + 1) % J`). It is very easy to mess this code up, thankfully we have unit tests.

One plus of the `convVEdgeDetection()` function is that it is mostly self-contained, we just need to provide an input/output pair for each test. In my image processing class all the unit tests were created by the TA. So, I only saw them and used them. This blog is poorly funded so we will have to do with a make-believe TA (AKA me) to generate these input/output pairs for us. So, we crack the whip and tell our TA: "build that test". Our TA runs away and they come back with:

{% highlight Python%}
from collections import namedtuple
Test = namedtuple("Test", "input", "output")
test1 = Test(
    np.array([
        [0, 0, 0, 1, 0, 0, 0],
        [0, 0, 0, 1, 0, 0, 0],
        [0, 0, 0, 1, 0, 0, 0],
        [1, 1, 1, 1, 1, 1, 1],
        [0, 0, 0, 1, 0, 0, 0],
        [0, 0, 0, 1, 0, 0, 0],
        [0, 0, 0, 1, 0, 0, 0],
    ]),
    np.array([
        [0, 0,   1, 0,   -1, 0, 0],
        [0, 0,   1, 0,   -1, 0, 0],
        [0, 0, 2/3, 0, -2/3, 0, 0],
        [0, 0, 2/3, 0, -2/3, 1, 1],
        [0, 0, 2/3, 0, -2/3, 0, 0],
        [0, 0,   1, 0,   -1, 0, 0],
        [0, 0,   1, 0,   -1, 0, 0],
    ])
)
{% endhighlight %}

How did our TA generate this? Likely they just googled something, but they will tell you they did it by hand. Our TA knows what the `convVEdgeDetection()` function is supposed to do so they look at the input and the manually compute the output. Just in case you weren't counting, that little test took 441 multiplications and 392 additions to create. By hand my left foot!

With our TA-built test we can run our unit test by using the code `False not in np.isclose(test.output, convVEdgeDetection(test.input))`. Quick, easy and we're done right? Well, not so fast. We've only done one test and there are a lot of ways that the convolution function can go wrong. For example, the input is left-right and top-bottom symmetric. It is possible that the `convVEdgeDetection()` is wrong in a way that won't be detectable because of these symmetries. So we crack the whip again. We tell the TA generate a bunch of tests. We want our TA to think about all the ways that a convolutions algorithm can go wrong and then they should generate a test for each of them. 441 multiplications and 392 addtions.

However, unit tests aren't the only tests and software isn't the only industry. I've worked in the semiconductor industry and they also do testing, lots of it. 

As you are probably aware, integrated circuits are built on silicon wafers, thin disks of nearly pure silicon. What you may not be aware of is that an integrated circuit is built up in layers. The process usually starts with building the transistors piece-by-piece and then moves on to creating layers and layers of wires to connect the transistors together. 

In many points along the way of building an integrated circuit layer-by-layer it is possible to stop and take a look at the last layer that was created before we bury it under all the layers to come. We can look at that layer and identify parts of the circuit that weren't created properly or particle contaminates that (somehow) attached themselves on our wafer. We call these defects. When you are building circuits at nanometer scales with a process that can be described as screen printing with light (photo-lithography), it's a good idea to look how many and what kind of defects you have on any given layer. 

However, wafers can be $$ 3.24 \times 10^{16} $$ square nanometers in area and we have feature sizes in the 10's of nanometers. I don't want to look at that much stuff, you don't want to look at that much stuff, we don't have a TA and it's "against the rules" to make the interns look at it (something about job satisfaction). Not only don't we want to look at that much stuff we don't even want to store it. Instead we take a picture of a small section of that circuit and we then run a test to check whether that picture shows a defect. With this method we only have to look at and store the pictures that contain defects.

There are a lot of technical difficulties to overcome in this approach. For example, the production process has a little bit of variation in the output. When you design a wire it will be straight as an arrow. Add a turn into that wire and it makes a sharp right angle. However, in the circuit as manufactured it will be a wobbly mess with a rounded off corner. So what is and isn't a defect can be a fuzzy proposition.

The main problem, however, is knowing what that little section of the circuit is supposed to look like. It's nice to know things like those two little wires are supposed to be one big wire and vice versa. One solution is to create a reference model, one that doesn't have any defects. However, you don't have to create a reference model at all. If you have another circuit you can just do a comparison because it is unlikely that any two circuits will share the same defect. How lucky for us that the circuit we are inspecting happens to be located on a wafer with 200 other circuits that should look pretty similar. Take a picture of one, take a picture of another; if they're similar then no defect, if they don't look similar then yes defect.

A second area where testing is used in the silicon industry is in the design of the circuit itself. Hardware design oddly starts with software and a programming language. This programming language is otherwise known as a hardware description language (HDL). The designers will use HDL to define a model of the circuit. From that HDL model of the circuit a gate-level model of the circuit will be built. This is just a circuit that consists of logic-gates (AND, OR, NOT, etc). As if two models weren't enough the designers will also build a transistor model and a layout model and more.

The point of all these different models of the circuit is to look at the circuit, check characteristics and then *modify* it. This raises the question: did our modifications change the function of the circuit? Does my new transistor model implement the same funcationality as the HDL model and the gate-level model and the other models? Silicon circuit designers can answer this question by testing the different models. One way is to simulate each model and then feed inputs and compare outputs between the different models. 

Silicon designers, however, have found neat trick to check not just one or two inputs or even a hundred, but all possible inputs. First each model is transformed into a boolean formula. The various formulas are then combined to form a boolean function such that the function will be 1 if there exists an input where the models differ in output. Identifying inputs that will make a boolean function evaulate to 1 is the boolean satisfiability problem (SAT). SAT is well studied, not well understood and there are tons of programs that will find the inputs to make a boolean function evaluate to 1. These programs are called SAT Solvers. The problem reduces to transform each model, construct the boolean function and use a SAT solver to see if the function has a 1 output.

If you've followed along this far, you might be wondering what it is I am getting after. Well, it's this: testing is comparing two different instances of the same thing and it's not necessary that either be perfect for the test to work. For the unit test we were comparing the `convVEdgeDetection()` function to the TA convolution function, for the silicon wafer we compared two different circuits of the same design and for the silicon design we compared two (three or four or ...) different models of the same circuit.

Thinking in this way changes the way I see code testing. In unit testing we often test based on input/output pairs and only a few of them. This limitation is because one of the two instances (the human) is very slow. This slowness means that very few tests can be built. These few tests then have to be chosen carefully evaluate all the corner cases. Except, we always forget a corner case. So, we need to carefully choose our tests, because we only have a few tests, because one of the things we're testing is very slow. Overcoming slowness is what computers were created for. If we could make sure that both of the things we were testing were run on silicon and not meat, speed would not be an issue and tests would be abundant.

Here's an idea, write one function, then write a second one and see if they're the same. This initially seems absured. I just wrote one function that does the job, why would I just repeat that work. The key insight here is that these functions can't just be the same they must be different and they must be different in a specific way. We want to insure that errors made in the process of writing the first function will not be the same errors made in the process of writing the second function. For example, think of the silicon wafer testing. In that case we are testing two different circuits on the same wafer. The reason this works is that the errors (defects) are random and it is incredibly unlikely that two circuits will exhibit the same defect. 

If we consider the number and type of errors to be a random variable associated with the process of writing a function. We can declare one random variable to be $$A$$ for the first process and $$B$$ for the second process. Our desired state would be that they share no errors between them or at least very few. We can describe this mathmatically as $$\sum_{e \in \mathcal{E}} Pr[A = e] \times Pr[B = e] \approx 0$$. The visual analog of this would look like:

![non-overlap of error distributions](\assets\what-are-tests\ideal_error_distribution_overlap.png)

One approach could be to implement the two functions in different languages. Since we to minimize the overlap of error distributions these languages should be *very* different languages. If our language is C then we should probably avoid pairing it with any of the other curly-brace languages. They're practically the same. The thought process for writing code is very similar and the way that you structure code is also pretty similar. Something like Python might work to eliminate some errors. Consider the following two functions:

{% highlight C %}
// C code
void convert_matrix(float *mat, int w, int h) { // UhOh!
  for (int i = 0; i <= w; i++) // UhOh!
    for (int j = 0; j < h; j++); // UhOh!
        mat[i][j] = 255.0 * mat[i][j] // UhOh!
}
{% endhighlight %}

{% highlight Python %}
# Python
def convertMatrix(mat):
    for i in range(len(mat)):
        row = mat[i]
        for j in range(len(row)):
            row[j] = 255 * row[j]
{% endhighlight %}

The structure, the syntax and the built-in functionality of Python helps to eliminate many of the troubling issues with writing C. I count 5 errors in that C code. However I don't think Python is a good language for this use-case. I think Python will have a subset of errors from C. This means that Python errors will have a strong likelihood of being C errors as well. 

For many of the other languages that I have encountered I suspect that there will still be a core of error overlap that will be difficult to remove. It is simply because we construct our languages on top of similar data objects (int, arrays, maps) and similar data-flow constructs (if-else, functions). Then again, maybe I just don't know enough interesting languages.

A language that I find intriguing for this use-case us UML. It's Turing complete (for whatever that's worth) and there are compilers for it, though not very good ones. I find UML interesting because it was and is used for planning and documentation. Think about it, use UML to plan and/or document your design. Code it up in C or Python or Pascal. Then you can re-use the UML to build a test-suite. Now, these two functions will likely still share errors, because both the UML and the C code share part of the creation process. At least you'll be able to say that the code is what's in the documentation.

A potentially good use-case for two programming languages is when one of them is obtuse. If your code-base tends to be a bunch of illegible Perl and you can't afford to migrate to a more sensible language. You could build test cases in that sensible language. In a similar vein, if a piece of code needs to be optimized for performance and it starts to look like arcane code from a grand-wizzard of yore. It might be nice to have a less-performant, more-legible piece of code to test against it. Is it really necessary that your test code run in $$O((n/3)^{0.9998}log(n/1.02321))$$ instead of $$O(n log(n))$$? I didn't think so.

A better approach than picking different languages and the same programmer might be to use the same language and different programmers. Instead of pair-programming, divorced-programming! Tell programmer #1 what to do and lock them a room. Then tell programmer #2 what to do and lock them in a different room, in a different building, on a different campus, in a different country, ideally without cellphones or internet (programmers don't *need* StackOverflow). Only let programmer #1 and #2 out when the code is complete! Now you have two different versions of the same code. Throw a bunch of inputs at the two versions and compare to see if either has a bug in it. 

We've elimnated the need for output in the input/output testing pair. That's cool, but we can do better. Instead of writing our test inputs manually, we can manually write a test input generator function to automatically generate inputs. Sprinkle in some contrained-randomization (borrowed from SystemVerilog) and you have a powerful tool for thouroughly testing your code. 

If you think this is a waste. It well might be. Two programmers to do one task instead of two. Consider though, that we already do pair programming where two programmers work on a single task. The hope is that as one becomes fatigued or blocked the other can step in. Two programmers sharing their energy, skill and experience with the purpose of creating better code. Pair programming is one way to deploy programmers, but you can also deploy them through divorced programming. In divorced programming the goal isn't speed or communication but correct code for critical applications.

When you have a piece of code that is critical and absolutely has to be correct. Send two programmers off to create it twice. Build a test-input generator and then create all the corner case inputs you can think of. It'll be the most tested and validated piece of code in your code-base.

