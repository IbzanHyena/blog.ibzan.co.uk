// title: Writing a Neural Network from Scratch in J for Fun
// author: Ibzan
// date: 2023-05-30

# Writing a Neural Network from Scratch in J for Fun

I have been thinking a lot about neural networks recently, so I decided why not make my own in J?
The vast majority of neural network &mdash; and machine learning in general &mdash; code that I interact with is written in Python (by which I mean C or C++ with a Python frontend).
This hides a lot of the complexity of the implementation to allow the end user to focus on the higher-level aspects of training one.

However, theoretically, a simple neural network shouldn&rsquo;t be too hard to implement manually.
The majority of the operations are effectively array and matrix operations &mdash; a task J should be well-suited for.

So why not do it?

## Primer: what is a neural network?

It&rsquo;s a directed acyclic graph (DAG) made of layers of nodes.
Each layer is fully-connected with adjacent layers.
The first layer is called the **input layer** and the last layer is the **output layer**.
Layers in between are called **hidden layers**.

The edges each have a weight.
Data (numbers) flows through the network from the input layer to the output layer.
Each node takes a weighted average of the values it receives based on the weight of the edges, and then that value becomes its output.
Often, the output is first run through a function called the **activation function** before being sent down the network further.

This diagram shows a basic example of a neural network:

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3org/  1999/xlink" class="centred" display="block">
  <circle cx="20%" cy="33%" r="5%" />
  <circle cx="20%" cy="67%" r="5%" />
  <circle cx="50%" cy="25%" r="5%" />
  <circle cx="50%" cy="50%" r="5%" />
  <circle cx="50%" cy="75%" r="5%" />
  <circle cx="80%" cy="50%" r="5%" />
  <line x1="25%" x2="45%" y1="32%" y2="25%"/>
  <line x1="25%" x2="45%" y1="33%" y2="49.5%"/>
  <line x1="25%" x2="45%" y1="34%" y2="75%"/>
  <line x1="25%" x2="45%" y1="66%" y2="26%"/>
  <line x1="25%" x2="45%" y1="67%" y2="50.5%"/>
  <line x1="25%" x2="45%" y1="68%" y2="76%"/>
  <line x1="55%" x2="75%" y1="25%" y2="49%"/>
  <line x1="55%" x2="75%" y1="50%" y2="50%"/>
  <line x1="55%" x2="75%" y1="75%" y2="51%"/>
  <text x="20%" y="95%" text-anchor="middle" font-size="small">Input   layer</text>
  <text x="50%" y="95%" text-anchor="middle" font-size="small">Hidden   layer</text>
  <text x="80%" y="95%" text-anchor="middle" font-size="small">Output   layer</text>
</svg>

This kind of neural network is called a [feedforward][feedforward] neural network.
  There are also more complex kinds, such as [convolutional][convolutional] and [recurrent][recurrent], and many others, but I won&rsquo;t go into those here.

### Training

Now, how does the network actually learn?
To do this, first we must be able to evaluate how good it is currently.
There are lots of different methods for evaluating neural networks, and machine learning models generally, depending on what task they are trying to solve.

For this example, I decided to try a classification problem, since they are fairly simple.
The objective of a classification problem is to sort the inputs into discrete groupings, present in the training data.
In this case, there is one input node for each feature being considered, and one output node for each class.
The value of each output node then corresponds to the weighting of that class being selected.

>>> Neural networks used for classification are called [perceptrons][perceptron].

A lot of the time, these outputs are run through something like the [logistic function][logistic] to normalise them into the 0&ndash;1 range (and, as we will see later, I chose not to bother, for simplicity).

So, to evaluate the performance of our model &mdash; there are a few typically-used metrics.
I will be using accuracy.
While it has its flaws, it is easy to understand for a basic project like this.

**Accuracy** is just the number of correct classifications divided by the total number made.
Two out of four correct?
0.5 accuracy.
Three out of five?
0.6.

To make the network learn, now we just need to tweak its weights and see if our evaluation metric improves or not.
By doing this cleverly, networks can learn very quickly!
A widely-used method for doing this is called [backpropagation][backpropagation].

>>> Normally, for training, something called a **loss function** is used instead.
In neural network training, this typically takes the form of a function that provides the difference between the _actual_ network outputs and the _expected_ outputs.
By reducing the output of the loss function, the performance of the model improves.  
Typical loss functions include [Euclidean distance][distance], [cross-entropy][cross-entropy], and [mean squared error][mse].

I am not clever enough to want to do that quickly, however.
Instead, I opted for a different approach: taking the best-performing model and varying its weights _randomly_ and seeing if it gets any better or not.
Although inefficient, this method still works reasonably well.

## Writing some code in J

It helps to divide the problem into a few steps:

- Read in input data
- Prepare input data for training
- Construct a network, and run data through it
- Evaluate a network
- Train the network in some way
- Repeat this process until the result is satisfactory

### Read in input data

Our input data is the widely-used [iris dataset][iris].
The dataset is made of five columns: petal and sepal length and width, and iris species.
There are three species and one hundred and fifty total entries, fifty per species.

The data is stored in CSV format, so we can make use of J&rsquo;s [CSV addon][j-csv] to read it for us.

```j
require 'csv'
X =: makenum }:"1 }: readcsv 'iris.data'
y =: &gt; {:"1 }: readcsv 'iris.data'
```

In machine learning contexts, `X` is used to refer to the input features, and `y` to the known outputs.
The `X` is upper case to represent that it is a matrix; the `y` is lower case as it is a vector.

The last column is the species of iris &mdash; our class to predict &mdash; so we extract it separately.
By using [`}:`&nbsp;curtail][nuvoc-curtail] we remove the last row of the CSV (blank, because the file has a trailing newline) and then [`{:`&nbsp;tail][nuvoc-tail] with [`"`&nbsp;rank][nuvoc-rank] retrieves the last column.
[`&gt;`&nbsp;open][nuvoc-open] finally unboxes the array.

To get the input features we follow a similar approach.
Once again, the file is curtailed, and then each row is curtailed to produce an array of numbers only.
`makenum` is part of the CSV addon and converts the boxed array into a numeric array.

### Prepare input data for training

At the moment, our `X` columns are all quite different.
Let&rsquo;s calculate the means and standard deviations to show this:

```j
   ] means =: (+/%#) X
5.84333 3.054 3.75867 1.19867
   ] stds =: (+/%#)&amp;.:*: X -"1 means
0.825301 0.432147 1.75853 0.760613
```

Don&rsquo;t worry too much about the J.  
Mean on top, standard deviation below.

!!! Open this if you _are_ worried about the J
Calculating the mean is a common adage in J &mdash; &ldquo;sum over count,&rdquo; or just `+/ % #`, as above.  
The standard deviation is then found by subtracting the mean from the initial values and calculating the square root of the mean of the squared values.
Square and square root are inverses &mdash; so we can use [`&amp;.:`&nbsp;under][nuvoc-under] to perform those operations.
The verb we execute under squaring is the mean, and it&rsquo;s exactly the same as before.  
There is also an appearance from [`"`&nbsp;rank][nuvoc-rank] here, because we are calculating the mean for all of the columns at once.
It is necessary to use rank so that the shapes of the arrays being subtracted work.  
More specifically, `$ X` is `150 4`, and `$ means` is `4`.
Without using rank, J would try to broadcast over the first axis &mdash; obviously that doesn&rsquo;t work here, as 150 is not equal to 4.

In a neural network, uneven data like this is not desirable, because it means that variations in one input value can have much larger impacts than variations in others.
To solve this, data is usually normalised.
There are three typical normalisation strategies: min-max, median, and z-score.

- In **min-max**, the data is scaled such that the minimum value is 0 and the maximum value is 1.
- In **median**, the data is scaled such that the median value is 1.
- In **z-score**, the data is treated as coming from a [normal distribution][normal], and then each value is replaced with the number of standard deviations away from the mean that it is.

These different approaches have different advantages.
We want to make all of the different features have the same weighting if there are variations, so z-score seems like the best fit.
Reusing the earlier code, we can rescale `X` quite easily:

```j
   Xscaled =: (X -"1 means) %"1 stds
   (+/%#) Xscaled
_4.32247e_16 _6.57252e_16 2.5091e_16 _1.76155e_16
   (+/%#)&.:*: (-"1 +/%#) Xscaled
1 1 1 1
```

This is much better &mdash; exactly what z-score is meant to produce.
All the different quantities have been rescaled such that their mean is (effectively) zero, and their standard deviations are one.
Hence, the same relative variation in one column is represented by the same absolute variation now.

Now, we can prepare the `y` values, too.
We have categorical data currently:

```j
   ~. y
Iris-setosa
Iris-versicolor
Iris-virginica 
```

and we must turn it into numerical data.
There are two approaches for this: one-hot encoding, and ordinal encoding.
I chose to use ordinal encoding because of how I planned to evaluate the model later.
In ordinal encoding, each class is given a distinct integer instead of a textual value.
Thus, `Iris-setosa` becomes 0, `Iris-versicolor` becomes 1, and `Iris-virginica` becomes 2.

```j
   labels   =: ~. y
   yClasses =: labels i. y
```

For simplicity later, I will now overwrite our original variables.

```j
   X =: Xscaled
   y =: yClasses
```

There is one thing left to do for data preparation: splitting our data into training and test data.

**Training** data is the set of data we will use to judge the model while it learns.  
**Test** data is a separate set that is used to independently analyse the model&rsquo;s performance.

By separating these data sets, we prevent the model from simply learning every possible case in the input and regurgitating the answer back at us &mdash; this is called _overfitting_ in machine learning.
It is not desirable because an overfitted model often performs poorly on novel data, i.e., data not present in what it learnt from, as instabilities in the hyper-specific behaviour it developed are exposed.

In more advanced machine learning approaches, there are [methods to try and prevent overfitting][early stopping] by utilising the test and training data sets together.
I did not bother to implement these, because I am lazy.

To split the test and training data, I simply picked the first ten entries of each class to be for the test set, and the rest for training.

```j
   testMask =: 9 >: 50 | i. 150
   Xtest    =: testMask # X
   ytest    =: testMask # y
   Xtrain   =: (-. testMask) # X
   ytrain   =: (-. testMask) # y
```

Our input data is now prepared for training!

### Construct a network, and run data through it

Before we can actually get to training though, we have to draw the rest of the owl.

**Question**: how do we go from the input layer to the output layer?  
**Answer**: one layer at a time.

**Question**: how do we go from one layer to the next?  
**Answer**: matrix multiplication and the activation function.

Recall that a layer is made up of some number of nodes, each of which pass on a value to the next layer.
This is a vector!
To transform a vector of size m to a vector of size n, you can multiply by an m&times;n matrix.

So, we can represent the neural network as a collection of matrices &mdash; each matrix provides the transformation from one layer to the next.
Let&rsquo;s write some code to generate that!

```j
   randMatrix =: 1 -~ 2 * (?@:$) &amp; 0
   randLayers =: 2 &lt;@randMatrix\ ]
```

Here, `randMatrix` generates an array of the specified size full of random numbers from &minus;1 to 1.
`randLayers` then takes a list of layer sizes and generates appropriately-sized matrices to transform between them.

!!! More detailed explanation
[`$`&nbsp;shape][nuvoc-shape] creates an array of the specified shape with values from another array (or an atom).
Here, the value is zero everywhere.
Then, [`?`&nbsp;roll][nuvoc-roll] turns these zeroes into random numbers between zero and one.
The rest of `randMatrix` is responsible for the rescaling.
It doubles the array and then subtracts one from it.  
In `randLayers`, [`\`&nbsp;infix][nuvoc-infix] generates an appropriate matrix for each adjacent pair of layers.
Additionally, it has to box the matrices via [`&lt;`&nbsp;box][nuvoc-box] to avoid adding padding to them.

To run data through this network, we can simply prepend our `X` values to the array and then apply matrix multiplication and the activation function until we have finished the array.
A slight quirk of J here is that we have to reverse the array of matrices after prepending `X`, since J evaluates things right-to-left.

```j
   combineLayers =: {{ u@:(+/%#) .* }}
   predict       =: {{ (u combineLayers)~&amp;:&gt;/ }} @: |.
   prepare       =: ;"_ 1
```

In `combineLayers` and `predict`, `u` is the activation function.
`combineLayers` is just matrix multiplication followed by the activation function.
`predict` does the aforementioned reversal and some unboxing, since our matrices have to be boxed for flatness reasons.

`prepare` is a very little helper that will add the initial `X` values to a network, so it can be given to `predict`.

!!! More details about the above J
[`.`&nbsp;matrix&nbsp;product][nuvoc-matrix-product] is a conjunction that takes a pair of combining functions to make matrix products.  
As a motivating example, we can consider the normal matrix product, written in J as `+/ .*`
The right-hand verb is how the individual elements from rows and columns are combined.
In matrix multiplication this is multiplication &mdash; so `*`.
The left-hand verb is then how these results are combined into the elements of the new matrix; `+/` is sum, as typical.  
In our matrix product, we want to take the weighted mean followed by the activation function for our left-hand verb.
Hopefully, `u@:(+/%#)` makes sense for this.  
`predict` then simply unboxes its operands and uses [`/`&nbsp;insert][nuvoc-insert] to do this across all of the layers.

What should we use as the activation function?

There are lots of choices &mdash; too many to go into here &mdash; but I chose to use [hyperbolic tangent][hyperbolic] because I tried a few and it seemed decent.
Specifically, I also tried ReLU, the identity function, and inverse tangent.  
In J:

```j
   tanh =: 7&amp;o.
   relu =: 0&amp;&gt;.
   id   =: ]
   atan =: _3&amp;o.
```

We are almost ready to run our network for the first time!
There is only one thing left to decide on: our layer sizes.

The first layer has to be the number of input features, and the last layer has to be the number of classes.
We can pick the sizes of the hidden layers.
For this example, I am picking a single hidden layer of 512 neurons.
It doesn&rsquo;t _have_ to be a power of two, but that just feels nice, y&rsquo;know?
Like some kind of a superstition accrued from using computers for too long.

```j
   inNodes  =: _1 { $ X
   outNodes =: # labels
   layers   =: inNodes , 512 , outNodes
   network  =: randLayers layers
   $ network
2
```

As expected, our network is made of two matrices &mdash; input&ndash;hidden, and hidden&ndash;output.

Let&rsquo;s predict some stuff now!

```j
   $ tanh predict X prepare network
150 3
```

This has produced three output values for each input.
How do we pick which one is the class the model has predicted?
An easy approach: take the largest one.

```j
   argmax =: (i. >./)
   argmax"1 tanh predict Xtrain prepare network
1 1 1 1 0 0 0 1 0 1 1 1 1 1 1 1 1 1 1 1 1 1 0 0 1 1 1 1 1 1 1 1 1 1 0 1 1 1 1 1 1 2 1 2 1 0 2 1 2 1 0 2 2 2 2 2 2 0 2 1 1 1 1 2 2 0 0 2 2 1 1 2 1 1 1 2 2 2 1 2 0 2 0 2 2 0 0 0 2 2 0 2 2 2 0 0 2 0 2 0 2 0 2 2 2 0 0 0 0 0 0 0 2 0 0 0 2 0 0 2
```

So, how good is this?

### Evaluate a network

We can compare this to our known `y` values:

```j
   accuracy =: (+/%#)@:=
   ytrain accuracy argmax"1 tanh predict Xtrain prepare network
0.325
   ytest accuracy argmax"1 tanh predict Xtest prepare network
0.266667
```

>>> Our evaluation metric &mdash; accuracy &mdash; is calculated separately for both the training and testing data sets.
The training accuracy is what will be used to guide the model&rsquo;s evolution.
By contrast, the test accuracy at this point is just for us to see how the network performs on unseen data.

In this case, the neural network wasn&rsquo;t particularly good at accurately predicting which species of iris we had.
That said, it wasn&rsquo;t particularly bad, either!
There are three species of iris in the test set, so randomly guessing we would expect to be right one third of the time.
Comparing a model (of any kind) to a random baseline is a useful technique to make sure that it&rsquo;s actually any good.

This takes us on to the next step, however!

### Train the network in some way

We can take our network and replicate it, and then introduce _mutations_ to the replicas.
The idea borrowed from evolution: random variations in the network&rsquo;s weights may lead to better performance.
For simplicity, only the best model from any given set will be chosen to &ldquo;reproduce&rdquo; like this, and it will also survive to the next generation, to act as a comparison and avoid performance regressing.

To train the network, we can now follow a simple loop:

- Start with a set of models
- See how they perform on the training data
- Pick the best one
- Randomise it!

Here&rsquo;s a bunch of code I wrote to do just that.

```j
   pred      =: tanh predict
   replicate =: {{ ($~ x , #) y }}
   
   fittest =: {{
     accuracies =. y (accuracy argmax"1@:pred)"_ 1 x
     }. x {~ argmax accuracies
   }}
   
   mutate =: {{ (+ randMatrix@:$)&amp;.&gt; y }}
```

`pred` serves the role as a shorthand for our chosen prediction method with an activation function.  
`replicate` takes a network and a number of times to copy it and does just that.  
`fittest` evaluates a number of networks on the same dataset and uses our `accuracy` verb from earlier to pick the best one and return it.
There is also an intricacy here in that it has to remove the added `X` values, which is what the `}.` does on the second line of `fittest`.  
Finally, `mutate` takes a network and adds a random amount of variation to every weight in it.

>>> Varying all the weights at once is not the best way of training, as mentioned earlier, but I am still lazy.
Oh well.

Combining all of this together into a single verb, we get:

```j
   generationSize =: 1000
   
   acc =: {{ n accuracy argmax"1 pred m prepare y }}
   
   evolveStep =: {{
    'Xtr Xte' =. m
    'ytr yte' =. n
    best =. ytr fittest~ Xtr prepare y
    echo Xtr acc ytr best
    echo Xte acc yte best
    echo '-------'
    (, [: mutate generationSize replicate ]) best
   }}
```

`acc` is just a shorthand for prepare-and-evaluate.  
`evolveStep` is more complicated &mdash; it is a conjunction that takes a total of three inputs.
`m` is the `X` data, both training and testing.
Similarly, `n` is the `y` data.
`y` (sorry for the name clash, but it&rsquo;s unavoidable) here represents our networks.

Firstly, `evolveStep` deconstructs `m` and `n` to get the individual data sets.
Then, it finds the best neural network out of the ones it has been given, which will be used as the basis for the next generation.
For diagnostic purposes, it prints out the accuracy on both the testing and training data sets.
Then, it copies the best network it had several times and mutates it, before prepending the unmodified version as well.

In short, `evolveStep` is effectively a function that takes one set of networks and produces a new one.

I have also defined `generationSize` as a constant to control how many networks are made at each step.

We can test this by first generating a number of different networks, and then running `evolveStep` on them.

```j
   firstGeneration =: (randLayers"1) generationSize replicate layers
   $ (Xtrain ; Xtest) evolveStep (ytrain ; ytest) firstGeneration
0.875
0.766667
-------
1001 2
```

Note that out starting train and test accuracy is much better here.
That is because we generated 1000 neural networks rather than just one, and so by pure luck one of them is pretty decent already.

The output of this is our next generation &mdash; ready for `evolveStep` to be run on it again.

As a simple diagram, `evolveStep` performs the following operation:

<svg>
  <!-- first column -->
  <!-- top -->
  <circle cx="15%" cy="11%" r="2%" />
  <circle cx="15%" cy="20%" r="2%" />
  <circle cx="20%" cy="7%" r="2%" />
  <circle cx="20%" cy="15%" r="2%" />
  <circle cx="20%" cy="23%" r="2%" />
  <circle cx="25%" cy="15%" r="2%" />
  <rect x="12%" y="3%" width="16%" height="25%" fill="none"/>
  <!-- middle -->
  <circle cx="15%" cy="46%" r="2%" />
  <circle cx="15%" cy="55%" r="2%" />
  <circle cx="20%" cy="42%" r="2%" />
  <circle cx="20%" cy="50%" r="2%" />
  <circle cx="20%" cy="58%" r="2%" />
  <circle cx="25%" cy="50%" r="2%" />
  <rect x="12%" y="38%" width="16%" height="25%" fill="none"/>
  <!-- bottom -->
  <circle cx="15%" cy="81%" r="2%" />
  <circle cx="15%" cy="90%" r="2%" />
  <circle cx="20%" cy="77%" r="2%" />
  <circle cx="20%" cy="85%" r="2%" />
  <circle cx="20%" cy="93%" r="2%" />
  <circle cx="25%" cy="85%" r="2%" />
  <rect x="12%" y="73%" width="16%" height="25%" fill="none"/>
  <!-- middle columns -->
  <line x1="29%" x2="32%" y1="15%" y2="49%"/>
  <line x1="29%" x2="32%" y1="50%" y2="50%"/>
  <line x1="29%" x2="32%" y1="85%" y2="51%"/>
  <text y="50%" text-anchor="middle" font-size="small">
    <tspan x="40%" dy="-0.5em">Select</tspan>
    <tspan x="40%" dy="1em">best</tspan>
  </text>
  <line x1="47%" x2="52%" y1="50%" y2="50%"/>
  <text x="60%" y="50%" text-anchor="middle" font-size="small" dominant-baseline="central">
    Mutate
  </text>
  <line x1="68%" x2="71%" y1="49%" y2="15%" />
  <line x1="68%" x2="71%" y1="50%" y2="50%" />
  <line x1="68%" x2="71%" y1="51%" y2="85%" />
  <!-- right column -->
  <!-- top -->
  <circle cx="75%" cy="11%" r="2%" />
  <circle cx="75%" cy="20%" r="2%" />
  <circle cx="80%" cy="7%" r="2%" />
  <circle cx="80%" cy="15%" r="2%" />
  <circle cx="80%" cy="23%" r="2%" />
  <circle cx="85%" cy="15%" r="2%" />
  <rect x="72%" y="3%" width="16%" height="25%" fill="none"/>
  <!-- middle -->
  <circle cx="75%" cy="46%" r="2%" />
  <circle cx="75%" cy="55%" r="2%" />
  <circle cx="80%" cy="42%" r="2%" />
  <circle cx="80%" cy="50%" r="2%" />
  <circle cx="80%" cy="58%" r="2%" />
  <circle cx="85%" cy="50%" r="2%" />
  <rect x="72%" y="38%" width="16%" height="25%" fill="none"/>
  <!-- bottom -->
  <circle cx="75%" cy="81%" r="2%" />
  <circle cx="75%" cy="90%" r="2%" />
  <circle cx="80%" cy="77%" r="2%" />
  <circle cx="80%" cy="85%" r="2%" />
  <circle cx="80%" cy="93%" r="2%" />
  <circle cx="85%" cy="85%" r="2%" />
  <rect x="72%" y="73%" width="16%" height="25%" fill="none"/>
</svg>

### Repeat this process until the result is satisfactory

We just need to repeat the above process many times!
J aficionados may know of the [`^:`&nbsp;power&nbsp;of&nbp;verb][nuvoc-power-of-verb] conjunction, which does exactly that.

For the sake of getting a single model returned, we can also wrap it in a final evaluation step to pick the best network from the last generation.

```j
   evolve =: {{
     'Xtr Xte' =. m
     'ytr yte' =. n
     yte fittest~ Xte prepare (m evolveStep n)^:x y
   }}
```

`evolve` will run `evolveStep` over and over again `x` times, on some initial input networks, and then finally pick the best one according to the _test_ data, not the training data.

Let&rsquo;s run it!

```j
   bestNetwork =: 5 (Xtrain ; Xtest) evolve (ytrain ; ytest) firstGeneration
0.875
0.766667
-------
0.891667
0.7
-------
0.891667
0.7
-------
0.891667
0.7
-------
0.891667
0.7
-------
```

Well, it worked!
Kind of.

The network quickly found an improvement, increasing its training accuracy from 87.5% to 89.2%.
However, this actually _decreased_ its accuracy on the test data set.

This is a common occurrence when training machine learning models, especially when there is relatively little data to work with.
Despite this, we can try running it for longer, and hope that by sheer chance &mdash; the only way this can learn &mdash; that we will get an overall better model.

```j
   bestNetwork =: 5 (Xtrain ; Xtest) evolve (ytrain ; ytest) firstGeneration
0.875
0.766667
-------
Many, many omitted numbers here
-------
0.9
0.8
-------
```

Success!

The model has learnt, and this time it improved both on the testing and training data sets.

### Training better?

How could we improve out training process?
Perhaps the obvious place to start with this framework is `mutate` &mdash; at the moment, it adds the same amount of randomness to the network regardless of how good we think the model is currently.

This could be refined by adding _less_ randomness to the model if it is already good, according to our accuracy measurements.
Adjusting the amount of randomness is easy:

```j
   mutate =: {{ (+ x * [: randMatrix $)&.> y }}
```

Now we have an `x` parameter to control the magnitude of the random matrix generated.
Plugging this into `evolveStep`:

```j
   evolveStep =: {{
     'Xtr Xte' =. m
     'ytr yte' =. n
     best =. ytr fittest~ Xtr prepare y
     echo a =. Xtr acc ytr best
     echo Xte acc yte best
     mag =. -. a
     echo '-------'
     (, mag mutate generationSize replicate ]) best
   }}
```

I have picked one minus the train accuracy for the magnitude.

Why?

I dunno, felt right &mdash; the closer we get, the smaller the variations should be, like some crude form of [gradient descent][gradient descent].

Does it help?

```j
   bestNetwork =: 5 (Xtrain ; Xtest) evolve (ytrain ; ytest) firstGeneration
0.841667
0.8
-------
Many, many omitted numbers here
-------
0.875
0.833333
-------
```

In this specific case, yes!
Our _test_ accuracy is better.

In general, it&rsquo;s harder to tell, owing to how random this method is.

## Conclusion

We (I) made a very basic feedforward neural network.
Although it has a lot of shortcomings, it still manages to successfully learn how to classify species of iris from the iris data set better than randomly guessing.

We trained the network by evaluating large numbers of randomly-generated networks, finding which one performs the best, and then creating descendants of it, and repeating the process.
This did work!

It did not work very quickly or reliably.

Was it fun?  
Yes.  
Mission complete.

For further improvements, there are some considerations:

- Adding a weight term to each layer.
  In a neural network, a constant factor can be added at each layer, as well as the value from the calculation.
- Implementing backpropagation.
- Using a proper loss function.

Maybe I&rsquo;ll do those some other time.
For now, this quick project has served its purpose very well.

All in all, a fun bit of code to write in one evening!
I highly recommend you trying it yourself, whether using this kind of approach or not.

[backpropagation]: https://en.wikipedia.org/wiki/Backpropagation
[convolutional]: https://en.wikipedia.org/wiki/Convolutional_neural_network
[cross-entropy]: https://en.wikipedia.org/wiki/Cross_entropy
[distance]: https://en.wikipedia.org/wiki/Euclidean_distance
[early stopping]: https://en.wikipedia.org/wiki/Early_stopping
[feedforward]: https://en.wikipedia.org/wiki/Feedforward_neural_network
[gradient descent]: https://en.wikipedia.org/wiki/Gradient_descent
[hyperbolic]: https://en.wikipedia.org/wiki/Hyperbolic_functions
[iris]: https://archive.ics.uci.edu/ml/datasets/iris
[j-csv]: https://code2.jsoftware.com/wiki/Addons/tables/csv
[logistic]: https://en.wikipedia.org/wiki/Logistic_function
[mse]: https://en.wikipedia.org/wiki/Mean_squared_error
[normal]: https://en.wikipedia.org/wiki/Normal_distribution
[nuvoc-box]: https://code2.jsoftware.com/wiki/Vocabulary/lt
[nuvoc-curtail]: https://code2.jsoftware.com/wiki/Vocabulary/curlyrtco
[nuvoc-infix]: https://code2.jsoftware.com/wiki/Vocabulary/bslash#dyadic
[nuvoc-insert]: https://code2.jsoftware.com/wiki/Vocabulary/slash
[nuvoc-open]: https://code2.jsoftware.com/wiki/Vocabulary/gt
[nuvoc-matrix-product]: https://code2.jsoftware.com/wiki/Vocabulary/dot#dyadic
[nuvoc-power-of-verb]: https://code2.jsoftware.com/wiki/Vocabulary/hatco
[nuvoc-shape]: https://code2.jsoftware.com/wiki/Vocabulary/dollar#dyadic
[nuvoc-rank]: https://code2.jsoftware.com/wiki/Vocabulary/quote
[nuvoc-roll]: https://code2.jsoftware.com/wiki/Vocabulary/query
[nuvoc-tail]: https://code2.jsoftware.com/wiki/Vocabulary/curlylfco
[nuvoc-under]: https://code2.jsoftware.com/wiki/Vocabulary/ampdotco
[perceptron]: https://en.wikipedia.org/wiki/Perceptron
[recurrent]: https://en.wikipedia.org/wiki/Recurrent_neural_network
