## Project 6: Classification


<div class="project">

![](https://github.com/HEATlab/cs151-classification/blob/master/digits.gif)  
Which Digit?

![](https://github.com/HEATlab/cs151-classification/blob/master/pacman_multi_agent.png)  
Which action?


### <a name="Introduction"></a>Introduction

In this project, you will design three classifiers: a perceptron classifier, a large-margin (MIRA) classifier, and a slightly modified perceptron classifier for behavioral cloning. You will test the first two classifiers on a set of scanned handwritten digit images, and the last on sets of recorded pacman games from various agents. Even with simple features, your classifiers will be able to do quite well on these tasks when given enough training data.

Optical character recognition ([OCR](http://en.wikipedia.org/wiki/Optical_character_recognition)) is the task of extracting text from image sources. The data set on which you will run your classifiers is a collection of handwritten numerical digits (0-9). This is a very commercially useful technology, similar to the technique used by the US post office to route mail by zip codes. There are systems that can perform with over 99% classification accuracy (see [LeNet-5](http://yann.lecun.com/exdb/lenet/index.html) for an example system in action).

Behavioral cloning is the task of learning to copy a behavior simply by observing examples of that behavior. In this project, you will be using this idea to mimic various pacman agents by using recorded games as training examples. Your agent will then run the classifier at each action in order to try and determine which action would be taken by the observed agent.

The code for this project includes the following files and data, available as a [zip file](https://github.com/HEATlab/cs151-classification/archive/master.zip).

<table class="intro" border="0" cellpadding="10">

<tbody>

<tr>

<td colspan="2">**Data file**</td>

</tr>

<tr>

<td>data.zip</td>

<td>Data file, including the digit and face data.</td>

</tr>

<tr>

<td colspan="2">**Files you will edit**</td>

</tr>

<tr>

<td>perceptron_pacman.py</td>

<td>The location where you will write your behavioral cloning perceptron classifier.</td>

</tr>

<tr>

<td>perceptron.py</td>

<td>The location where you will write your perceptron classifier.</td>

</tr>

<tr>

<td>mira.py</td>

<td>The location where you will write your MIRA classifier.</td>

</tr>

<tr>

<td>dataClassifier.py</td>

<td>The wrapper code that will call your classifiers. You will also write your enhanced feature extractor here. You will also use this code to analyze the behavior of your classifier.</td>

</tr>

<tr>

<td>answers.py</td>

<td>Answer to Question 2 goes here.</td>

</tr>

<tr>

<td colspan="2">**Files you should read but NOT edit**</td>

</tr>

<tr>

<td>classificationMethod.py</td>

<td>Abstract super class for the classifiers you will write.  
(You **should** read this file carefully to see how the infrastructure is set up.)</td>

</tr>

<tr>

<td>samples.py</td>

<td>I/O code to read in the classification data.</td>

</tr>

<tr>

<td>util.py</td>

<td>Code defining some useful tools. You may be familiar with some of these by now, and they will save you a lot of time.</td>

</tr>

<tr>

<td>mostFrequent.py</td>

<td>A simple baseline classifier that just labels every instance as the most frequent class.</td>

</tr>

</tbody>

</table>

**Files to Edit and Submit:** You will fill in portions of [perceptron.py](https://github.com/HEATlab/cs151-classification/blob/master/perceptron.py), [mira.py](https://github.com/HEATlab/cs151-classification/blob/master/mira.py), [answers.py](https://github.com/HEATlab/cs151-classification/blob/master/answers.py),[perceptron_pacman.py](https://github.com/HEATlab/cs151-classification/blob/master/perceptron_pacman.py), and [dataClassifier.py](https://github.com/HEATlab/cs151-classification/blob/master/dataClassifier.py) (only) during the assignment, and submit them. You should submit these files with your code and comments. Please _do not_ change the other files in this distribution or submit any of our original files other than this file.

**Evaluation:** Your code will be autograded for technical correctness. Please _do not_ change the names of any provided functions or classes within the code, or you will wreak havoc on the autograder. However, the correctness of your implementation -- not the autograder's judgements -- will be the final judge of your score. If necessary, we will review and grade assignments individually to ensure that you receive due credit for your work.

**Academic Dishonesty:** We will be checking your code against other submissions in the class for logical redundancy. If you copy someone else's code and submit it with minor changes, we will know. These cheat detectors are quite hard to fool, so please don't try. We trust you all to submit your own work only; _please_ don't let us down. If you do, we will pursue the strongest consequences available to us.

**Getting Help:** You are not alone! If you find yourself stuck on something, contact the course staff for help. Office hours, section, and the discussion forum are there for your support; please use them. If you can't make our office hours, let us know and we will schedule more. We want these projects to be rewarding and instructional, not frustrating and demoralizing. But, we don't know when or how to help unless you ask.

**Discussion:** Please be careful not to post spoilers.

</div>

<div class="project">

### <a name="Q1"></a>Question 1 (4 points): Perceptron

A skeleton implementation of a perceptron classifier is provided for you in `perceptron.py`. In this part, you will fill in the `train` function.

Unlike the naive Bayes classifier, a perceptron does not use probabilities to make its decisions. Instead, it keeps a weight vector *w<sup>y</sup>* of each class *y* (*y* is an identifier, not an exponent). Given a feature list *f*, the perceptron compute the class *y* whose weight vector is most similar to the input vector *f*. Formally, given a feature vector *f* (in our case, a map from pixel locations to indicators of whether they are on), we score each class with:

<div>
  
![\( \mbox{score}(f,y) = \sum\limits_{i} f_i w^y_i \)](http://www.sciweavers.org/tex2img.php?eq=%5C%28%20%5Cmbox%7Bscore%7D%28f%2Cy%29%20%3D%20%5Csum%5Climits_%7Bi%7D%20f_i%20w%5Ey_i%20%5C%29&bc=White&fc=Black&im=jpg&fs=12&ff=arev&edit=0)

</div>

Then we choose the class with highest score as the predicted label for that data instance. In the code, we will represent *w<sup>y</sup>* as a `Counter`.

#### Learning weights

In the basic multi-class perceptron, we scan over the data, one instance at a time. When we come to an instance *(f, y)*, we find the label with highest score:

<div>
  
![y' = arg \max\limits_{y''} score(f,y'')](http://www.sciweavers.org/tex2img.php?eq=%5C%28%20y%27%20%3D%20arg%20%5Cmax%5Climits_%7By%27%27%7D%20score%28f%2Cy%27%27%29%20%5C%29&bc=White&fc=Black&im=png&fs=12&ff=arev&edit=0)

</div>

We compare *y'* to the true label \(y\). If *y' = y*, we've gotten the instance correct, and we do nothing. Otherwise, we guessed *y'* but we should have guessed *y*. That means that *w<sup>y</sup>* should have scored *f* higher, and *w<sup>y'</sup>* should have scored *f* lower, in order to prevent this error in the future. We update these two weight vectors accordingly:

<div>

![w^y = w^y + f](http://www.sciweavers.org/tex2img.php?eq=%5C%28%20w%5Ey%20%3D%20w%5Ey%20%2B%20f%20%5C%29&bc=White&fc=Black&im=png&fs=12&ff=arev&edit=0)

</div>

<div>

![w^{y'} = w^{y'} - f](http://www.sciweavers.org/tex2img.php?eq=%5C%28%20w%5E%7By%27%7D%20%3D%20w%5E%7By%27%7D%20-%20f%20%5C%29&bc=White&fc=Black&im=png&fs=12&ff=arev&edit=0)

</div>

Using the addition, subtraction, and multiplication functionality of the `Counter` class in `util.py`, the perceptron updates should be relatively easy to code. Certain implementation issues have been taken care of for you in `perceptron.py`, such as handling iterations over the training data and ordering the update trials. Furthermore, the code sets up the `weights` data structure for you. Each legal label needs its own `Counter` full of weights.

#### Question

Fill in the `train` method in `perceptron.py`. Run your code with:

<pre>python dataClassifier.py -c perceptron </pre>

**Hints and observations:**

*   The command above should yield validation accuracies in the range between 40% to 70% and test accuracy between 40% and 70% (with the default 3 iterations). These ranges are wide because the perceptron is a lot more sensitive to the specific choice of tie-breaking than naive Bayes.
*   One of the problems with the perceptron is that its performance is sensitive to several practical details, such as how many iterations you train it for, and the order you use for the training examples (in practice, using a randomized order works better than a fixed order). The current code uses a default value of 3 training iterations. You can change the number of iterations for the perceptron with the `-i iterations` option. Try different numbers of iterations and see how it influences the performance. In practice, you would use the performance on the validation set to figure out when to stop training, but you don't need to implement this stopping criterion for this assignment.

</div>
