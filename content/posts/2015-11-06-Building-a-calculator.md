---
title: "How hard can building a Calculator be right?"
description: "How hard can building a Calculator be right?"
tags: [parser, python, gui]
comments: true
share: true
cover_image: '/content/images/2015/4/scaffolding.jpg'
---

<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.5.0/css/font-awesome.min.css">

## BackDrop: 

So I was reading about this wonderful module called `tkinter` some days back. And man was I not sucked into it!

Sure, we do have some very good GUI modules like `PyQT`, `wxPython`, `PySide`. And I don't deny the fact that some are way better than `tkinter`. But one advantage which makes `tkinter` stand out from the crowd is that, it comes pre-packaged with the python package. 

What I mean by that is you don't have to install anything to run and write GUI's with it and run my Calculator program for instance.

## Dabbling with it:

I got naturally interested in it so after reading the docs for some time. I thought why not apply it by making something small. It got me thinking about what to make with it!

After some rambling aroung, I settled with the thought of making a Calculator using `Tkinter`. Aiming for a file search program at my second go.

## Taking a piece of paper:

So the first task was to decide whether to use `OOP`'s for this project or not. I figured out that adding `class`'es would only complicate it in the first go. 

Second was to decide which functions/operators should it have initially, upon which initial design should be made.

The problem which took me the hardest to figure out was where we had to parse the input entered by the user. 
Take this input as an example. 

![calcBlog_1](https://raw.githubusercontent.com/tasdikrahman/www.tasdikrahman.com/master/images/calcBlog_1.jpg)

Now to get `80` as the first operand, one has to press `8` and then `8` again. Now you and I know that it is `80`. But how do you make the program understand that?

I figured out that, it was best that I dumped the whole input into a function called `calculate` where I got the whole of the entered input through `display.get()`, `display` being an `Entry()` object.

### How was I dumping what I clicked into `Entry()` widget? 

I used two functions for that, `get_variables(num)` for `operand` and `get_operation(operator)` for `operator`. Inside each, I have a `global` variable, whose value gets incremented each time control transfers to any of these functions. I use this `global` variable to keep track of the position of the next data item(be it a operand or operator) to be inserted into the `Entry` widget.

Here is what I did for `get_variables()`

{% highlight python %}
def get_variables(num):
    """Gets the user input for operands and puts it inside the entry widget"""
    global i
    display.insert(i, num)
    i += 1
{% endhighlight %}

So for a typical `Button`, lets take `7` here. I have 

{% highlight python %}
seven = Button(root, text = "7", command = lambda : get_variables(7), font=FONT_LARGE)
seven.grid(row = 4, column = 0)
{% endhighlight %}


Well one problem solved!

### Adding and `<-` (undo) button

Now what if you pressed something wrong, you don't want to press the `AC` button to clear the whole of the entered text as you would have to again waster your time into typing it again. How do we achieve that?

I figured out that the `whole_string` stores the whole of the input and what I wanted with the `<-` button was an undo of what I did last. 

So I just had to remove the last index of the string `whole_string`
For that.

{% highlight python %}
new_string = whole_string[:-1]
print(new_string)
clear_all()
display.insert(0, new_string)
{% endhighlight %}

That did it what `<-` was supposed to do

### How do I seperate the operands from the operators?

Now at first thoughts, I thought I should hardcode it for each and every operator. Like you have `+` inside the `whole_string` which stores the value of `display.get()`. And then you do a 

{% highlight python %}
if '+' in whole_string:
    # Now to split the contents into `operands`, I did a `var1, var2 = whole_string.split(operator)`. 
    var1, var2 = whole_string.split("+")
    result = int(var1) + int(var2)
{% endhighlight %}

But you notice that this fails at the slightest of sneezes. Enter more than 2 variables and then you are left with an extra unused operand. You will left making `if-else` clauses for the rest of this project. 

So I dropped this approach for good!

After some frustrating 3 hours and messing around and deleting with 3 `branches`. I finally came to this.

>Why not user `parser` to parse the expression and evaluate it.

So this is what I did

{% highlight python %}
def calculate():
    whole_string = display.get()
    try:
        formulae = parser.expr(whole_string).compile()      ## returns a `parser` object
        result = eval(formulae)                             ## evaluates the parsed expression
        clear_all()
        display.insert(0, result)
    except Exception:
        clear_all()
        display.insert(0, "Error!")
{% endhighlight %}

This solved my problem and it works for most of the test cases I have checked with, but by far the best approach would be write my own `parser`. 

Will refactor to include it in my next release.

## Download it!

<a href="https://github.com/tasdikrahman/pyCalc/releases/download/v1.0/pyCalc_v1" target="_blank"><i class="fa fa-linux fa-2x"></i></a>

## Fork this project

Feel free to fork this project and make changes to it. 

<div markdown="0"><a href="https://github.com/tasdikrahman/pyCalc" class="btn btn-info">Github repo</a></div>

## Filing bugs

If you find some bugs, 
please file an issue on the github page 

<div markdown="0"><a href="https://github.com/tasdikrahman/pyCalc/issues/new" class="btn btn-danger">Report a bug</a></div>

Till then. Goodbye!