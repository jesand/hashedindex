===============================
hashedindex
===============================

.. image:: https://img.shields.io/travis/MichaelAquilina/hashedindex.svg
        :target: https://travis-ci.org/MichaelAquilina/hashedindex

.. image:: https://img.shields.io/pypi/v/hashedindex.svg
        :target: https://pypi.python.org/pypi/hashedindex


Fast and simple InvertedIndex implementation using hash lists (python dictionaries).

* Free software: BSD license
* Documentation: https://hashedindex.readthedocs.org.

Features
--------

hashedindex provides a simple to use inverted index structure that is flexible enough to work with all kinds of use cases.

Basic Usage:

.. code-block:: python

    import hashedindex
    index = hashedindex.HashedIndex()

    index.add_term_occurrence('hello', 'document1.txt')
    index.add_term_occurrence('world', 'document1.txt')

    index.get_documents('hello')
    Counter({'document1.txt': 1})

    index.items()
    {'hello': Counter({'document1.txt': 1}),
    'world': Counter({'document1.txt': 1})}

    example = 'The Quick Brown Fox Jumps Over The Lazy Dog'

    for term in example.split():
        index.add_term_occurrence(term, 'document2.txt')

The hashedindex is not limited to strings, any hashable object can be indexed.

.. code-block:: python

   index.add_term_occurrence('foo', 10)
   index.add_term_occurrence(('fire', 'fox'), 90.2)

   index.items()
   {'foo': Counter({10: 1}), ('fire', 'fox'): Counter({90.2: 1})}

Text Parsing
------------

The hashedindex module comes included with a powerful textparser module with methods to split text into
tokens.

.. code-block:: python

   from hashedindex import textparser
   list(textparser.word_tokenize("hello cruel world"))
   [('hello',), ('cruel',), ('world',)]

Tokens are wrapped within tuples due to the ability to specify any number of n-grams required:

.. code-block:: python

   list(textparser.word_tokenize("Life is about making an impact, not making an income.", ngrams=2))
   [(u'life', u'is'), (u'is', u'about'), (u'about', u'making'), (u'making', u'an'), (u'an', u'impact'),
    (u'impact', u'not'), (u'not', u'making'), (u'making', u'an'), (u'an', u'income')]

Take a look at the function's docstring for information on how to use `stopwords`, specify a `min_length` or
`ignore_numeric` terms.

Integration with Numpy and Pandas
---------------------------------

The initial idea behind hashedindex is to provide a really quick and easy way of generating matrices for machine
learning with the additional use of numpy, pandas and scikit-learn. For example:

.. code-block:: python

   from hashedindex import textparser
   import hashedindex
   import numpy as np

   index = hashedindex.HashedIndex()

   documents = ['spam1.txt', 'ham1.txt', 'spam2.txt']
   for doc in documents:
       with open(doc, 'r') as fp:
            for term in textparser.word_tokenize(fp.read()):
                index.add_term_occurrence(term, doc)

   # You *probably* want to use scipy.sparse.csr_matrix for better performance
   X = np.as_matrix(index.generate_feature_matrix(mode='tfidf'))

   y = []
   for doc in index.documents():
       y.append(1 if 'spam' in doc else 0)
   y = np.asarray(doc)

   from sklearn.svm import SVC
   classifier = SVC(kernel='linear')
   classifier.fit(X, y)

You can also extend your feature matrix to a more verbose pandas DataFrame:

.. code-block:: python

   import pandas as pd
   X  = index.generate_feature_matrix(mode='tfidf')
   df = pd.DataFrame(X, columns=index.terms(), index=index.documents())

All methods within the code have high test coverage so you can be sure everything works as expected.

Found a bug? Nice, a bug found is a bug fixed. Open an Issue or better yet, open a pull request.
