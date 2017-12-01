---
title: "Predicting Transformer Affiliation from Motto"
layout: post
---

# The Story
To give an example of working with data while still having fun, I've decided to
tackle [Transformers](https://en.wikipedia.org/wiki/Transformers).
Specifically, after finding [this
dataset](http://infochimps.com/datasets/transformers-character) someone scraped
together from Wikipedia, I thought it'd be interesting to see if we can
predict the affiliation (Autobot vs. Decepticon) of a character using one of
[Scikit-Learn](http://scikit-learn.org/)'s
[naive Bayes
classifiers](https://en.wikipedia.org/wiki/Naive_Bayes_classifier).

Enough chit-chat; let's roll up our sleeves and dive in!

# Preliminaries
Checking out the data, it looks like the entries are indexed by name (on the
far left of the line), which approximates the Transformer's name.
Let's see how many entries we're dealing with:

{% highlight bash %}
$ grep -P '^\S' infobox_transformers_character.yaml | wc -l
718
{% endhighlight %}

There are 718 entries, but it looks like there may be repeats (i.e., either the
data isn't as clean as we'd hoped, or the names are not a unique enough
identifier).

{% highlight bash %}
$ grep -P '^\S' infobox_transformers_character.yaml |\
      uniq -c |\
      awk '{print $1}' |\
      sort -n |\
      uniq -c
250  1
 85  2
 45  3
 19  4
 12  5
  2  6
  1  7
  1  8
{% endhighlight %}

So it looks like the majority of names aren't repeated, but there are a few;
one is even repeated 8 times!

{% highlight bash %}
$ grep -P '^\S' infobox_transformers_character.yaml |\
      uniq -c |\
      sort -nr |\
      head
8 Prowl_(Transformers):
7 Bonecrusher_(Transformers):
6 Ultra_Magnus:
6 Ironhide:
5 Thrust_(Transformers):
5 Swindle_(Transformers):
5 Snarl_(Transformers):
5 Skydive_(Transformers):
5 Scavenger_(Transformers):
5 Razorclaw:
{% endhighlight %}

Prowl is the most popular, eh? Let's look at the different entries with that
name.

{% highlight bash %}
$ grep -P -A5 '^Prowl' infobox_transformers_character.yaml
# output omitted
{% endhighlight %}

So these certainly seem to be unique entries; we should uniquify these names,
because `PyYAML` turns YAML into a dictionary (and therefore squashes things
under a single heading).

{% highlight bash %}
$ cat infobox_transformers_character.yaml |\
    perl -ple 'BEGIN{$n = 0;} if (m/(^\S+)[:]/) { $_ = $1 . ($n++) . ":"; }'
# output omitted
{% endhighlight %}

Looks good; let's save it to a new YAML file.

{% highlight bash %}
$ perl -ple 'BEGIN{$n = 0;} if (m/(^\S+)[:]/) { $_ = $1 . ($n++) . ":"; }' \
    infobox_transformers_character.yaml > unique_names.yaml
{% endhighlight %}

# Bring out the Python
Let's start up IPython and start digging.

{% highlight python %}
>>> import yaml
>>> with open('unique_names.yaml') as f:
        raw_data = yaml.load(f)
{% endhighlight %}

Loading that raw data into `Pandas`:


{% highlight python %}
>>> import numpy as np
>>> import pandas as pd
>>> messy_df = pd.DataFrame(raw_data).transpose()
>>> has_motto = messy_df[messy_df.motto.notnull()]
>>> len(has_motto)
485
>> any(has_motto.affiliation.isnull())
False
{% endhighlight %}

There are 485 entries that have a motto, and every entry with a motto also has
an affiliation.
As you might expect, though, our data isn't clean enough yet; there are some
affiliations we don't expect (i.e. not just "Autobot" or "Decepticon"), and the
mottos aren't in the cleanest form.

{% highlight python %}
>>> has_motto.affiliation.unique()
array(['Decepticon', 'Autobot', 'Maximal', 'Predacon',
       'Autobot, later Maximal', 'Mini-Con',
       'Predacon, later Maximal then Decepticon',
       'Maximal, later Autobot', 'Vehicon',
       'Maximal, \\n Former Predacon', 'Mutant',
       'Decepticon, later Autobot', 'None/Decepticon',
       'Decepticon, later Predacon', 'botconcron'],
       dtype=object)
>>> has_motto.affiliation.value_counts()
Autobot                                    213
Decepticon                                 143
Mini-Con                                    40
Maximal                                     28
Predacon                                    23
Vehicon                                      9
Maximal, later Autobot                       7
Decepticon, later Autobot                    6
Mutant                                       4
Autobot, later Maximal                       4
Maximal, \n Former Predacon                  3
None/Decepticon                              2
Decepticon, later Predacon                   1
botconcron                                   1
Predacon, later Maximal then Decepticon      1
{% endhighlight %}


Consulting some [expert knowledge](http://en.wikipedia.org/), it looks like

- Maximals can count as Autobots since they're descended from them
- Predacons can count as Decepticons, since they're the enemies of the Maximals
- Mini-Cons can be either, so we'll have to ignore them
- Vehicons are Decepticons
- I was unable to find botconcron; this looks like a transcription error, but
  there's only one so we'll drop it

Cleaning up and creating a new variable `numeric_affil` with 0 = Decepticon and
1 = Autobot:

{% highlight python %}
>>> def normalize_aff(aff):
        if any(bad in aff for bad in 'Decepticon Predacon Vehicon'.split()):
            return 0
        elif any(good in aff for good in 'Autobot Maximal'.split()):
            return 1
        else:
            return np.nan

>>> has_motto['numeric_affil'] = has_motto.affiliation.apply(normalize_aff)
{% endhighlight %}

Cleaning the mottos:

{% highlight python %}
>>> def clean_motto(m):
        return m.lower()
                .replace('\u2019 ', "'")
                .replace('\u2019', "'")
                .replace('\\"', '')
                .replace('<br>', '')
                .replace('</br>', '')
>>> has_motto['clean_motto'] = has_motto.motto.apply(clean_motto)
{% endhighlight %}

This still isn't perfect, but it's much better than we started with.

Now, we've got our cleaned data:

{% highlight python %}
>>> data = has_motto.dropna()[['numeric_affil', 'clean_motto']]
>>> data.columns = 'affiliation motto'.split()
{% endhighlight %}

# Scikit-Learn
Let's import what we'll need: two vectorizers for transforming text into
matrices, and the two naive Bayes classifiers we'll compare.
We'll also grab the `word_tokenize` function from `NLTK`.

{% highlight python %}
>>> from nltk import word_tokenize
>>> from sklearn.feature_extraction.text import CountVectorizer
>>> from sklearn.feature_extraction.text import TfidfVectorizer
>>> from sklearn.naive_bayes import BernoulliNB
>>> from sklearn.naive_bayes import  GaussianNB
>>> binary_vectorizer = CountVectorizer(
        binary=True, min_df=1, tokenizer=word_tokenize)
>>> tfidf_vectorizer = TfidfVectorizer(
        min_df=1, tokenizer=word_tokenize)
>>> binary_nb, gaussian_nb = BernoulliNB(), GaussianNB()
{% endhighlight %}

To compare the Bernoulli and the Gaussian naive Bayes classifiers, we will need
to use our vectorizers on them both.
To play fairly, we'll break out data into train and test groups.

{% highlight python %}
>>> from sklearn.cross_validation import train_test_split
>>> binary_motto = binary_vectorizer.fit_transform(data.motto)
>>> tfidf_motto = tfidf_vectorizer.fit_transform(data.motto)
>>> (binary_motto_train,
     binary_motto_test,
     affil_train,
     affil_test) = train_test_split(
        binary_motto, data.affiliation, test_size=0.2, random_state=1729)
>>> (tfidf_motto_train,
     tfidf_motto_test,
     affil_train,
     affil_test) = train_test_split(
     tfidf_motto, data.affiliation, test_size=0.2, random_state=1729)
{% endhighlight %}

Now to build the two models we wish to compare.

{% highlight python %}
>>> binary_nb.fit(binary_motto_train, affil_train)
>>> gaussian_nb.fit(tfidf_motto_train.toarray(), affil_train)
{% endhighlight %}

# Comparing Models
If this were a real life task, we may not have test data.
To compare our models, then, we won't resort to the test data just yet;
instead, we'll use [leave one out
cross-validation](http://en.wikipedia.org/wiki/Cross-validation_%28statistics%29).

{% highlight python %}
>>> from sklearn.cross_validation import cross_val_score, LeaveOneOut
>>> binary_cv_score = cross_val_score(binary_nb,
                                      binary_motto_train,
                                      affil_train,
                                      cv=LeaveOneOut(len(affil_train)))
>>> gaussian_cv_score = cross_val_score(gaussian_nb,
                                        tfidf_motto_train.toarray(),
                                        affil_train,
                                        cv=LeaveOneOut(len(affil_train)))
>>> print("Binary CV Score: {0:.5f}".format(binary_cv_score.mean()))
>>> print("Gaussian CV Score: {0:.5f}".format(gaussian_cv_score.mean()))
Binary CV Score:        0.65341
Gaussian CV Score:      0.58523
{% endhighlight %}

Though neither of these two classifiers look great, it looks like the binary
one wins on our training data.
Let's pretend we've made the decision to field that classifier in production
and check out its performance on the test data.

# Binary Classifier Results
{% highlight python %}
>>> from sklearn import metrics
>>> binary_prediction = binary_nb.predict(binary_motto_test)
{% endhighlight %}

Here's the simplest way of looking at our Binary NB's performance: a confusion
matrix.
As we can see from this, it looks like our classifier was pretty good at
getting Autobots right, but Decepticons were harder to accurately predict---I
guess this makes sense, since Decepticons are rather sneaky.

{% highlight python %}
>>> print(metrics.confusion_matrix(affil_test, binary_prediction))
                  Predicted Decepticon    Predicted Autobot
 Was Decepticon                      8                   28
 Was Autobot                         6                   46
{% endhighlight %}

For a more in-depth measure of the classifier's performance, we can turn to
`sklearn`'s `classification_report`:

{% highlight python %}
>>> print(metrics.classification_report(
            affil_test,
            binary_prediction,
            target_names='Decepticon Autobot'.split()))

             precision    recall  f1-score   support
 Decepticon       0.57      0.22      0.32        36
    Autobot       0.62      0.88      0.73        52
avg / total       0.60      0.61      0.56        88
{% endhighlight %}

According to this, we can see that our classifier is best at identifying actual
Autobots (see the Autobot recall score), but isn't that great at much else.
For more on how to interpret this table, see [Scikit-Learn's
documentation](http://scikit-learn.org/stable/modules/model_evaluation.html#binary-classification).

# Conclusion
It looks like we didn't do that well at classifying Transformer affiliation
from motto; our best classifier only had an F1 score slightly above 50%.
One possibility is that we didn't have enough data; perhaps a classifier
trained on thousands of samples would do better than ours that had less than
400.
Another possibility is that mottos aren't that indicative of affiliation to
begin with, and we should look for our predictive power elsewhere.
In any case, it's kind of fitting to use Machine Learning on Transformers.
