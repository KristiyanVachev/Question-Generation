
#  Question Generation

This project was originally intended for an AI course at Sofia University. During it's execution, I was constraint on time and couldn't implement all the ideas I had, but I plan to continue working on it... and I did pick up the topic for my Master's thesis, using **T5 Transformers to generate question-answer pairs along with distractors**. Check it out in the [Question-Generation-Transformers](https://github.com/KristiyanVachev/Question-Generation-Transformers) repository. 

The approach for identifyng keywords used as target answers has been accepted in the RANLP2021 conference - [Generating Answer Candidates for Quizzes and Answer-Aware Question Generators](https://arxiv.org/abs/2108.12898v1).


## General idea
The idea is to generate multiple choice answers from text, by splitting this complex problem to simpler steps:

 - **Identify keywords** from the text and use them as answers to the questions.
 - **Replace the answer** from the sentence with *blank space* and use it as the base for the question.
 - **Transform the sentence** with a blank space for answer to a more *question-like sentence*.
 - **Generate distractors**, words that are similar to the answer, as *incorrect answers*.

![Question generation step by step gif](https://media.giphy.com/media/1n4JPydITD3mGvTZBZ/giphy.gif)

## Installation

### Creating a virtual environment *(optional)*
To avoid any conflicts with python packages from other projects, it is a good practice to create a [virtual environment](https://docs.python.org/3/library/venv.html) in which the packages will be installed. If you do not want to this you can skip the next commands and directly install the the requirements.txt file. 

Create a virtual environment :

    python -m venv venv

Enter the virtual environment:

*Windows:*

    . .\venv\Scripts\activate

*Linux or MacOS*

    source .\venv\Scripts\activate

Install ipython inside the venv:

    ipython kernel install --user --name=.venv

Install jupyter lab inside the venv:

    pip install jupyterlab

### Installing packages

    pip install -r .\requirements.txt 
    
### Run jupyter

    jupyter lab

## Execution

### Data Exploration
Before I could to anything, I wanted to understand more about how questions are made and what kind of words are it's answers.

I used the [SQuAD 1.0](https://rajpurkar.github.io/SQuAD-explorer/) dataset which has about 100 000 questions generated from Wikipedia articles.

You can read about the insights I've found in the *Data Exploration* jupyter notebook.

### Identifying answers
My assumption was that **words from the text would be great answers for questions**. All I needed to do was to decide which words, or short phrases, are good enough to become answers.

I decided to do a binary classification on each word from the text. [spaCy](https://spacy.io/) really helped me with the word tagging.

#### Feature engineering
I pretty much needed to create the entire dataset for the binary classification. 
I extracted each non-stop word from the paragraphs of each question in the SQuAD dataset and added some features on it like:

 - **Part of speech**
 - Is it a **Named entity**
 - Are only **alpha characters** used
 - **Shape** - whether it's only alpha characters, digits, has punctuation (xxxx, dddd, Xxx X. Xxxx)
 - **Word count**

And the label **isAnswer** - whether the word extracted from the paragraph is the same and in the same place as the answer of the SQuAD question. 

Some other features like **TF-IDF** score and **cosine similarity** *to the title* would be the great, but I didn't have the time to add them.

Other than those, it's up to our imagination to create new features - maybe whether it's in the start, middle or end of a sentence,  information about the words surrounding it and more... Though before adding more feature it would be nice to have a metric to assess whether the feature is going to be useful or not.

#### Model training
I found the problem similar to *spam filtering*, where a common approach is to tag each word of an email as coming from a spam or not a spam email.

I used scikit-learn's **Gaussian Naive Bayes** algorithm to classify each word whether it's an answer.

The results were surprisingly good - at a quick glance, the algorithm classified most of the words as answers. The ones it didn't were in fact unfit.

The cool thing about *Naive Bayes* is that you get the **probability** for each word. In the demo I've used that to order the words from the most likely answer to the least likely.

### Creating questions
Another assumption I had was that **the sentence of an answer could easily be turned to a question**. Just by placing a *blank space* in the position of the answer in the text I get a **"cloze" question** *(sentence with a blank space for the missing word)*

**Answer:** 
Oxygen

**Question:**
 \_____ is a chemical element with symbol O and atomic number 8.

I decided it wasn't worth it to transform the cloze question to a more question-looking sentence, but I imagine it could be done with a **seq2seq neural network**, similarly to the way text is translated from one language to another.

### Generating incorrect answers
The part turned out really well. 

For each answer I generate it's most similar words using **word embeddings** and **cosine similarity**.

![Most similar words to oxygen](https://i.gyazo.com/175b9f86b3defc0798800cb06169cc3f.png)

Most of the words are just fine and could easily be mistaken for the correct answer. But there are some which are obviously not appropriate.

Since I didn't have a dataset with incorrect answers I fell back on a more classical approach.

I removed the words that **weren't the same part of speech** or **the same named entity** as the answer, and added some more context from the question.

I would like to find a dataset with multiple choice answers and see if I can create a *ML model* for generating better incorrect answers.

## Results
After adding a Demo project, the generated questions aren't really fit to go into a classroom instantly, but they are't bad either. 

The cool thing is the **simplicity** and **modularity** of the approach, where you could find where it's doing bad (*say it's classifying verbs*) and plug a fix into it. 

Having a complex Neural Network (*like all the papers on the topics do*) will probably do better, especially in the age we're living. But the great thing I found out about this approach, is that it's like a *gateway for a software engineer*, with his software engineering mindset, to get into the field of AI and see meaningful results. 

## Future work (*updated*)
I find this topic quite interesting and with a lot of potential. I would probably continue working in this field.

 I even enrolled in a *Masters of Data Mining* and will probably do some similar projects. I will link anything useful here.

I've already put some more time on finishing the project, but I would like to transform it more to a tutorial about getting into the field of AI while having the ability to easily extend it with new custom features. 

## Updates

**Update - 29.12.19:** 
The repository has become pretty popular, so I added a new notebook (*Demo.ipynb*) that combines all the modules and generates questions for any text. I reordered the other notebooks and documented the code (a bit better). 

**Update - 09.03.21:** 
Added a requirements.txt file with instructions to run a virtual environment and fixed the bug a with *ValueError: operands could not be broadcast together with shapes (230, 121) (83, )*

I have also started working on my Master's thesis with a similar topic of Question Generation. 

**Update - 27.10.21:** 
I have uploaded the code for my Master's thesis in the [Question-Generation-Transformers](https://github.com/KristiyanVachev/Question-Generation-Transformers) repository. I highly encourage you to check it out. 

Additionally the approach using a classfier to pick the answer candidates has been accepted as a students paper in the RANLP2021 conference. [Paper here](https://arxiv.org/abs/2108.12898v1).


