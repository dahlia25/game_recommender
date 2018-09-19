# Building a Game Recommender

## Goal

There are several recommenders that operate based on numerical rating scores. So I thought it would be interesting to build a video game recommender based on user review text.

## Data

Used Selenium and BeautifulSoup to collect data on over 3400 games with a total of approximately 280,000 user comments from Metacritic.com.

The following features were gathered for each game:

* Game Title
* Platform
* Genre(s)
* Developer/Publisher
* Number of Players (allowed in the game)

For each game comment, the following features were collected:

* Username
* User Score
* Review Text
* Number of Helpfulness

Due to limited time, I was unable to collect all ~15,000 games listed on Metacritic. To collect data more efficiently, I set up multiple scrapers on an EC2 instance in AWS.

### Tools & Packages Required

* Python
* Pandas
* Matplotlib
* Pickle
* Scikit-Learn
* Selenium
* BeautifulSoup
* AWS (Amazon Web Services)
* NLP (Natural Language Processing)
* LangDetect (a package used to detect languages; can be pip installed)
* VaderSentiment (a pre-built sentiment analyzer for social media text; can be pip installed)
* NLTK (for text processing)

# Project Design

## General Approach

I used two approaches to build the game recommender.

1. Build two item-item collabortive recommenders based on: Sentiment Score and User Score.
2. Build a recommender using NLP to find themes and features within user review text. 

By doing this, I can recommend games that share similar features/themes and sentiment ratings to the those that each user likes.

### First Approach: Build Item-Item Recommenders based on User Sentiment Score and User Score

Every user comment on Metacritic is accompanied by a User Score (which is given by the user); this is also another measure for user sentiment.

Although the user score is given, I also performed sentiment analysis (using VaderSentiment) on each user comment by using Vader's *compound score* to assign sentiment scores to each comment. Click [here](https://github.com/cjhutto/vaderSentiment) for more information on VaderSentiment.

Prior to building the recommenders, I did a comparison between vader scores (from VaderSentiment) and user scores. From this, I saw two main concerns:

1. User scores do not always accurately reflect what the user critiques; a user could give high number ratings, but give harsh critiques or vice versa.
2. Words have different positive, neutral or negative weights in VaderSentiment. So a review could be negative, but if the positive words outweigh the negative words, the sentiment score will result as a positive review. This also indicates that VaderSentiment is unable to detect sarcasm.

Using item-item collaborative filtering, I built two recommenders; one based on Vader sentiment score, and the other based on actual user score. Each recommender provides a list of video game recommendations, so I find and recommend the overlapping games from the two lists. 

### Second Approach: Build Recommender based on Features in User Review Text using NLP

A second recommender is built because sentiment scores are not able to capture any themes or features in the text. When I say features, I am talking about capturing themes/features such as, 'horror survival', 'RPG', 'puzzle games' and etc.

Below outlines the general steps I took to tackle this approach:

1. Text preprocessing; removed all non-English reviews, punctuations, common and rare words, followed by lemmatization. 
1. Aggregate all comments into one big text/article for **each** game. 
2. Used TF-IDF to vectorize all text.
3. Used LSA (Latent Semantic Analysis) to reduce dimensionality.
4. Topic modeling; to check if all topics make sense.
5. Build recommender using NearestNeighbors (with cosine similarity). 

### Final Product

With recommenders based on sentiment scores and review text, I was able to build a final recommender by combining the two types of recommenders. Both types of recommenders generate a list of recommendations. The final recommender will recommend games that overlapped between the two recommenders.

1. Have the text-based recommender generate a list of recommendations.
2. Use the sentiment-based recommender to filter out the 'top' or 'highly' rated games.
3. Generate a list of recommendations agreed by recommenders from steps 1 and 2. 

By doing this, not only are the features/themes captured, I am also able to recommend such games that are also 'highly' rated by other users. 

## Future Improvements & Problems Encountered

* Due to limited data, I was not able to evaluate the performance of my recommender. I would be able to evaluate the performance if I could retrieve information on video game recommendations that users liked/disliked.
* Find features that users liked/disliked of games could help us understand game consumers better in the future. 

Some problems that I encountered in this project:

* When I was using NLP to analyze user review text, after topic modeling, I also performed clustering. LDA (Latent Dirichlet Allocation) with CountVectorizer resulted in the best clustering, but the topics did not make sense. In the end, I chose to use LSA (Latent Semantic Analysis) with the TF-IDF vectorizer, since the topics made sense and the clusters were decent.
