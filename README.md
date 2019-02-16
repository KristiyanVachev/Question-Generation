#  Question Generation
This project was originally intended for an AI course at Sofia University. During it's execution, I was constraint on time and couldn't implement all the ideas I had, but I plan to continue working.

## General idea
The idea is to generate multiple choice answers from text, by splitting this complex problem to simpler steps:

 - **Identify keywords** from the text and use them as answers to the questions.
 - **Replace the answer** from the sentence with *blank space* and use it as the base for the question.
 - **Transform the sentence** with a blank space for answer to a more *question-like sentence*.
 - **Generate words**, that are similar to the answer, as *incorrect answers*.

![Question generation step by step gif](https://media.giphy.com/media/1n4JPydITD3mGvTZBZ/giphy.gif)

## Execution

### Data Exploration
Before I could to anything, I wanted to understand more about how questions are made and what kind of words are it's answers.

I used the [SQuAD 1.0](https://rajpurkar.github.io/SQuAD-explorer/) dataset which has about 100 000 questions generated from Wikipedia articles.

You can read about the insights I've found in the *Data Exploration* jupyter notebook.

### Identifying answers
My assumption was that **words from the text would be great answers for questions**. All I needed to do was to decide which words, or short phrases, are good enough to become answers.

I decided to do a binary classification on each word, or named entity, from the text. [spaCy](https://spacy.io/) really helped me with the word tagging.

#### Feature engineering
I pretty much needed to create the entire dataset for the binary classification. 
I extracted each non-stop word from the Wikipedia articles in the SQuAD dataset added some features on it:

 - **Part of speech**
 - Is it a **Named entity**
 - Are only **alpha characters** used
 - **Shape** - whether it's only alpha characters, digits, has punctuation (xxxx, dddd, Xxx X. Xxxx)
 - **Word count**

I think that a **TF-IDF** score and **cosine similarity** *to the title* could be the most influential when training but I didn't have the time.

I would like to implement them and think of some other features - maybe whether it's in the start, middle or end of a sentence,  information about the words surrounding it and more... 

After that it would be useful to see which features are most important for the algorithm.

#### Training
I found the problem similar to *spam filtering*, where a common approach is to tag each word of an email as coming from a spam or not a spam email.

I used scikit-learn's **Gaussian Naive Bayes** algorithm to classify each word whether it's an answer.

The results were surprisingly good - at a quick glance, the algorithm classified most of the words as answers. The ones it didn't were in fact unfit.

I would like to play a bit more with the data and try to get fewer words classified as answers. I think it would be a great idea to get a score *(0 to 1)* for each word. That way I could start generating questions for the words with the highest score and eventually, for the lower scoring ones I would expect lower quality questions.

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

## Future work
I find this topic quite interesting and with a lot of potential. I would like to try all of the ideas I didn't have time for and explore some more.  

After that, I will merge all of the work I've done, into a script that would directly generate a *json* file with questions and answers ready to be used in an application. 
