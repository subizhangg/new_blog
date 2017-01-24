---
layout: post
title: "What Does Rolling Stone Greatest 500 Albums List Look Like?"
date: 2017-01-22
output: html_document
share: true
categories: blog
tags: [visualization]
---

## Part 0. Motivation and Outline

**Motivation**

Listening to album alone has always been my biggest pleasure. As one of the most influential music brand and magazine, Rolling Stone once published a list in 2012 of the Greatest 500 Albums of all time based on opinions from music industrial figures. I hope to discover more insights about music development trend as well as characteristics by visualizing this dataset. In addition, I also want to briefly explore my personal taste within the 500 albums.

[Rolling Stone's 500 Greatest Albums of All Time - wikipedia](https://en.wikipedia.org/wiki/Rolling_Stone%27s_500_Greatest_Albums_of_All_Time)


**Outline**

- Part 0: Motivation and Outline

- Part 1: Data source and Collection

- Part 2: Data Preparation

- Part 3: Visualization of Rolling Stone's Greatest 500 Albums List

    - Based on Year
    - Based on Genre
    - Based on Artist


- Part 4: Visualization of My Own Music Taste

- Part 5: Limitation and Further Polish Need to Be Done Later



## Part 1. Where Does My Data Come From ?


My dataset is a csv file , which comes from public Kaggle online database [kaggle 500 albums dataset](https://www.kaggle.com/notgibs/500-greatest-albums-of-all-time-rolling-stone). However, I want to point out that the dataset is a second-hand dataset after cooking. It was created by Gibs, a graduate student at University of Texas at Austin.

- The observations within variables "Album", "Year" and "Artist" were collected from [MusicBrainz](https://musicbrainz.org/series/8668518f-4a1e-4802-8b0d-81703ced6418).

- The observations within variables "Genre" and "Subgenre" were collected from [Discogs API](https://www.discogs.com/developers/#).

- The observations within variable "personal_preference" was collected based on my subjective rating.(1 = really enjoy it, 0 = haven't listened to it before or don't enjoy it at all).

## Part 2. Preparation: Set Up Dependencies and Import the data

{% highlight r %}
# set up dependencies
library(tidyverse)
library(viridis)
library(reshape)

# import the data
album_data <- read_csv("../data/albumlist.csv")

# preview the dataset's structure
str(album_data)
{% endhighlight %}

{% highlight text %}
## Classes 'data.frame'  :	500 obs. of  7 variables:
## $ X1                 : chr  "Number" "1" "2" "3" ...
## $ Year               : int  1967 1966 1966 1965 1965 1971 1972 1979 1966 1968 ...
## $ Album              : chr  "Sgt. Pepper's Lonely Hearts Club Band" "Pet Sounds" "Revolver" "Highway 61 Revisited" ...
## $ Artist             : chr  "The Beatles" "The Beach Boys" "The Beatles" "Bob Dylan" ...
## $ personal_preferance: int  0 1 0 0 0 0 0 0 0 0 ...
## $ Genre              : chr  "Rock" "Rock" "Rock" "Rock" ...
## $ Subgenre           : chr  "Rock & Roll, Psychedelic Rock" "Pop Rock, Psychedelic Rock" "Psychedelic Rock, Pop Rock" "Folk Rock, Blues Rock" ...
{% endhighlight %}


## Part3. Visualization of Rolling Stone's Greatest 500 Albums List


### 1. Year

{% highlight r %}
album_data %>%
  ggplot(aes(x=Year)) +
  geom_histogram(aes(y=..density..),color="darkblue", fill="lightblue") +
  geom_density(alpha=.4, fill="white",color = "darkblue",linetype="longdash",size = 2) +
  ggtitle("Which Year Witnessed the Most Greatest Albums? ")

{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-3-1.png)

The golden era of the most greatest albums were generally between 1965 to 1980, when a large amount of wonderful artists started to jump onto the stage. The most renowned 1969 Woodstock Music Festival was also held in this period.


### 2. Genre

#### (1) What were the most awesome genres in the past 60 years ?

{% highlight r %}
album_data %>%
  ggplot(aes(x=reorder(Genre, -table(Genre)[Genre]), fill = Genre)) +
  geom_bar() +
  xlab("Genre") +
  scale_fill_viridis(option="viridis",discrete = TRUE) +
  ggtitle("The Frequency of Various Genres Appeared in Rolling Stone List")

{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-4-1.png)


"Rock" nearly took 60 percent seats and 300 rock albums appeared in Rolling Stone list. But the results are not astonishing to me for two reasons: the list was voted by many rock musicians; Rolling Stone itself is a rock music brand . "Soul" as one of the root genre in contemporary music was placed in the 2nd ranking with 60 albums being included. Although both "Techno" and "Hip-Hop" were new fresh waves born in 70s and 80s,they were third and fourth respectively. An interesting point is that "Pop" music's performance was much lower than I expected and only 10 albums were chosen on the list. In fact, tons of great pop stars have been very active on music circle in the past few years. One possible explanation might be evaluation standard of rock musicians and industry figures is very different from the public mainstream preference.


#### (2) Was there a popular genre dominated in the industry in each decade?

{% highlight r %}
# covert each specific year into decade scale
album_data$decade <- (album_data$Year - 1950) %/% 10
album_data$decade <- (album_data$decade * 10) + 1950
album_data$decade <- factor(album_data$decade)

# summarize the number of album in each decade and each genre
decade_genre <- album_data %>%
  group_by(decade,Genre) %>%
  summarise(num_album = n())

# visualize it
ggplot(data = decade_genre , aes(x = decade, y = Genre)) +
  geom_tile(aes(fill = num_album,colour = "orange")) +
  scale_fill_gradient(low = "snow", high = "royalblue4") +
  ggtitle("Albums Number in Each Decade and Genre ")
{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-5-1.png)


#### **60's: A prosperous decade of diverse genres co-existence**

During 50's, only three genres (rock, jazz and blue) were lively in the list. Then a bunch of genres suddenly sprouted in 70s, such as techno, soul, folk and Country. The popularity of soul became a main feature in that age.

#### **70's: Rock Music Heyday**

The most conspicuous purple cell within the heat map is 70's rock music. Over a hundred (20 percent) 70's rock albums were selected in this list. Countless rock stars swept over among Europe and North America in seventies, such as Led Zeppelin, David Bowie, Pink Floyd , Queen and The Sex Pistols.  

#### **80's: Jazz began to fade, Hip-Hop began to grow**

Eighties was a divide period witnessed the growth of Hip-Hop and the fading of Jazz. It's a very interesting phenomenon deserves further and deeper study.

#### **21st-century: New dominating pattern of Rock and Hip-Hop**

After entering 21st century, the taste of industry figures becomes more single and steady. Rock and Hip-Hop have been the most popular genres in music industry.

### 3. Artist

#### (1) Who were the most influential artists in the past 60 years?

{% highlight r %}

artist_top30 <- album_data %>%
  mutate(Artist = factor(Artist)) %>%
  group_by(Artist) %>%
  summarize(num_album = n()) %>%
  arrange(desc(num_album)) %>%
  head(30)

artist_top30 %>%
  ggplot(aes(x = num_album,y=reorder(Artist, num_album))) +
  geom_point(size=3,color = "red",alpha = 0.5) +
  ylab("Artist") +
  theme_bw() +
  theme(panel.grid.major.x = element_blank(),
        panel.grid.minor.x = element_blank(),
        panel.grid.major.y = element_line(color='grey60', linetype='dashed')) +
  ggtitle("Top 30 Influential Artists")

{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-6-1.png)


#### (2) Who were the most remarkable artists in specific genre?

##### - **Jazz**

{% highlight r %}
artist_jazz <- album_data %>%
  filter(Genre == "Jazz") %>%
  group_by(Artist) %>%
  summarise(count = n()) %>%
  arrange(desc(count))  


ggplot(artist_jazz, aes(x=reorder(Artist,-count), y=count)) +
  geom_point(size=5,pch = 17, col = "violetred3") +
  theme_bw() +
  theme(axis.text.x = element_text(angle=60, hjust = 1),
        panel.grid.major.y = element_blank(),
        panel.grid.minor.y = element_blank(),
        panel.grid.major.x = element_line(color="pink", linetype='dashed',size = 1)) +
  xlab("Jazz Artist") +
  ggtitle("The 8 Most Influential Jazz Artists")
{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-7-1.png)


##### - **Soul**

{% highlight r %}
artist_soul <- album_data %>%
  filter(Genre == "Soul") %>%
  group_by(Artist) %>%
  summarise(count = n()) %>%
  arrange(desc(count)) %>%
  head(10)

ggplot(artist_soul, aes(x=reorder(Artist,-count), y=count)) +
  geom_point(size=5,pch = 19, col = "royalblue3") +
  theme_bw() +
  theme(axis.text.x = element_text(angle=60, hjust = 1),
        panel.grid.major.y = element_blank(),
        panel.grid.minor.y = element_blank(),
        panel.grid.major.x = element_line(color="powderblue", linetype='longdash',size = 1)) +
  xlab("Soul Artist") +
  ggtitle("The 8 Most Influential Soul Artists")
{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-8-1.png)


##### - **Rock**

{% highlight r %}
artist_rock <- album_data %>%
  filter(Genre == "Rock") %>%
  group_by(Artist) %>%
  summarise(count = n()) %>%
  arrange(desc(count)) %>%
  head(10)

ggplot(artist_rock, aes(x=reorder(Artist,-count), y=count)) +
  geom_point(size=5,pch = 17, col = "green4") +
  theme_bw() +
  theme(axis.text.x = element_text(angle=60, hjust = 1),
        panel.grid.major.y = element_blank(),
        panel.grid.minor.y = element_blank(),
        panel.grid.major.x = element_line(color="palegreen3", linetype='dashed',size = 1)) +
  xlab("Rock Artist") +
  ggtitle("The 8 Most Influential Rock Artists")
{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-9-1.png)


## Part4. Visualization of My Own Music Taste

{% highlight r %}
genre_preference <- album_data %>%
  filter(personal_preferance == 1) %>%
  group_by(Genre) %>%
  summarise(count = n()) %>%
  arrange(desc(count))

genre_preference %>%
  ggplot(aes(x=reorder(Genre,-count), y=count, fill=Genre)) +
  geom_bar(width = 1, stat = "identity", color = "grey50", lwd = 1) +
  scale_fill_brewer(palette='Pastel1') +
  xlab("Genre") +
  ggtitle("Which was Subi's favourite Music Genre?")

{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-10-1.png)


{% highlight r %}
decade_preference <- album_data %>%
  filter(personal_preferance == 1) %>%
  group_by(decade) %>%
  summarise(count = n()) %>%
  arrange(desc(count))

decade_preference %>%
  ggplot(aes(x=reorder(decade,-count), y=count, fill=decade)) +
  geom_bar(width = 1, stat = "identity", color = "grey50", lwd = 1) +
  scale_fill_brewer(palette='Pastel1') +
  xlab("Age") +
  ggtitle("Which was Subi's Favourite Music Decade?")
{% endhighlight %}

![center](/images/rollingstone_500_albums/unnamed-chunk-11-1.png)

In terms of music taste, I am indeed a nostalgic person. Even now I still often enjoy listening to 60's and 70's albums. Most my favorite artists came from sixties and seventies, such as The kinks, Miles Davis, Carpenters, Billy Joel, The Beach Boys, Pink Floyd and Marvin Gaye. Sometimes I wish I could cross back to 50 years ago, and spend one night in a dark and wonderful underground livehouse, listening to my favorite idol's songs.

{% highlight r %}
# list part of my favorite albums
album_data %>%
  filter(personal_preferance==1) %>%
  head(10)
{% endhighlight %}

{% highlight text %}
## Year  Album                          Artist
## 1966	 Pet Sounds	                    The Beach Boys
## 1959	 Kind of Blue	                  Miles Davis		
## 1973	 Inner visions	                Stevie Wonder
## 1973	 The Dark Side of the Moon	    Pink Floyd
## 1965	 A Love Supreme	                John Coltrane
## 1978	 Here, My Dear	                Marvin Gaye
## 1977	 The Stranger	                  Billy Joel
## 1970	 Close to You	                  Carpenters
## 1967	 Something Else by The Kinks	  The Kinks
## 1979	 The Wall	                      Pink Floyd
{% endhighlight %}


## Part 5: Limitation and Further Polishment Need to Be Done

#### Limitation

(1) **Genre and subgenre classification might not be accurate.** Firstly, it's very common that one album is a fusion mixed various genres. For instance, it's hard to classify whether Acid Jazz belongs to Jazz or Techno. Secondly, the classification mechanics of Discog is unknown. Discog is a user-built database, so the genre labels are probably created by different users. The consistence of genre and subgenre naming could be an issue here.

(2) Only little information about each album was collected.

(3) The visualizations are mostly at general introductive level.


#### Further Polishment

(1) Collect more dimensional data of each album to go deeper about music development trend, such as artist nationality, circulation volumes, award history, rating, republication times, music brand. It could be a tough work to collect these trivial snippets from different places, but it's worthwhile to do that.

(2) Introduce statistical regression analysis and hypothesis testing between variables to quantize this topic.

(3) Introduce potential influence factors analysis of abrupt changes within specific music courses. Related contemporary music history materials should be included to enrich the report's depth.

(4) Add case study targeting one specific genre & subgenre, or decade. For instance, A Rock'n'roll big events time-line summarization would be nice to supplement much music industrial background.   
