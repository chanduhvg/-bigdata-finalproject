# Big Data Final Project
This repo is for processing text using Databricks Community Edition and PySpark.

# Data Source
* The Project Gutenberg EBook of Oliver Twist, by Charles Dickens
* https://www.gutenberg.org/files/730/730-0.txt

# Tools & Languages
* Python Programming Language
* Word Cloud
* PySpark 
* Seaborn
* Spark Processing Engine
* Databricks Community Edition

# Link to the published Databricks notebook
https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/1459184000507674/3872742223684322/573857438918659/latest.html

# Commands
## Data Injection
* We just need to import urllib.requests to read the data from a url. Using the urllib.request library, we can request or pull data from a url and store it in a temporary file.
```
import urllib.request 
stringInURL = "https://www.gutenberg.org/files/730/730-0.txt"
urllib.request.urlretrieve(stringInURL,"/tmp/gopi.txt")
```

* Using dbutils.fs.mv method we will transfer the temp file to the data folder of databricks storage.
```
dbutils.fs.mv("file:/tmp/gopi.txt", "dbfs:/data/OliverTwist.txt")
```

* Now we will transfer the data file into Spark, using sc.textfile(sparkContext) into Spark's RDD(Resilient Distributed Datasets).
```
rawRDD= sc.textFile("dbfs:/data/OliverTwist.txt")
```
## Data Cleaning
* For cleaning the data, we will break down the data using flatMap, covert the text to lower case, remove empty spaces and split the text into terms.
```
wordsRDD = rawRDD.flatMap(lambda eachLine: eachLine.lower().strip().split(" "))
```

* To remove the punctuation from the text, we will be importing re(Regular Expression) library. 
* Stopwords are words that do not add much meaning to a sentence (For example: the, have, etc.). In order to remove these we will be using pyspark.ml.feature by importing stopwordsRemover
```
import re
tokensRDD = wordsRDD.map(lambda w1: re.sub(r'[^A-Za-z]', '', w1))
from pyspark.ml.feature import StopWordsRemover
remover = StopWordsRemover()
stopwords = remover.getStopWords()
cleanWordsRDD = tokensRDD.filter(lambda word: word not in stopwords)

```


* To remove empty spaces from the data use the following command.
```
 removeSpacesRDD = cleanWordsRDD.filter(lambda x: x != "")
```

## Data Processing
* In data processing, we will map the words to key value pairs by using the following command.
*  We will be using reduceByKey() to get the word count by(word,count).
```
keyValueRDD= removeSpacesRDD.map(lambda word: (word,1))
wordCountRDD = keyValueRDD.reduceByKey(lambda acc, value: acc+value)
```

* We will be using sortbykey() which lists the words in the descending order and then prints the top fifteen most used words in 'The Jungle Book'.
```
OliverTwistResults = wordCountRDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(10)
print(OliverTwistResults)
```
![result](https://github.com/alekhyajaddu/bigdata-finalproject/blob/main/top15.JPG?raw=true)

## Charting
* We will be using Pandas, MatPlotLib, and Seaborn to visualize.
```
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
  
source = 'The Project Gutenberg EBook of Oliver Twist, by Charles Dickens'
title = 'Top Ten Words in ' + source
xlabel = 'Words'
ylabel = 'Count'

df = pd.DataFrame.from_records(OliverTwistResults, columns =[xlabel, ylabel]) 
plt.figure(figsize=(10,5))
sns.barplot(xlabel, ylabel, data=df, palette="husl").set_title(title)
```
![wordCount](https://github.com/alekhyajaddu/bigdata-finalproject/blob/main/wordcountChart.JPG?raw=true)

# WordCloud
* To create a wordcloud, we will be needing nltk and wordcloud libraries.
* Before using these libraries we need to install and download nltk, wordcloud and popular to over come name not defined and stopwords errors.
```
pip install wordcloud
```

```
pip install nltk
```

```
nltk.download('popular')
```

```
import wordcloud
import nltk
import matplotlib.pyplot as plt

from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from wordcloud import WordCloud

class WordCloudGeneration:
    def preprocessing(self, data):
        data = [item.lower() for item in data]
        stop_words = set(stopwords.words('english'))
        paragraph = ' '.join(data)
        word_tokens = word_tokenize(paragraph) 
        preprocessed_data = ' '.join([word for word in word_tokens if not word in stop_words])
        print("\n Preprocessed Data: " ,preprocessed_data)
        return preprocessed_data

    def create_word_cloud(self, final_data):
        wordcloud = WordCloud(width=1600, height=800, max_words=10, max_font_size=200, background_color="white").generate(final_data)
        plt.figure(figsize=(12,10))
        plt.imshow(wordcloud)
        plt.axis("off")
        plt.show()

wordcloud_generator = WordCloudGeneration()
import urllib.request
url = "https://www.gutenberg.org/files/730/730-0.txt"
request = urllib.request.Request(url)
response = urllib.request.urlopen(request)
input_text = response.read().decode('utf-8')

input_text = input_text.split('.')
clean_data = wordcloud_generator.preprocessing(input_text)
wordcloud_generator.create_word_cloud(clean_data)

```

![wordCloud](https://github.com/alekhyajaddu/bigdata-finalproject/blob/main/wordCloud.JPG?raw=true)

# References
* https://github.com/sudheera96/pyspark-textprocessing
* https://www.section.io/engineering-education/word-cloud/
* https://seaborn.pydata.org/tutorial/color_palettes.html
