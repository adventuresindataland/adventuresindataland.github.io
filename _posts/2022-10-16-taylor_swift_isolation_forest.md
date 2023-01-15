---
title: "What are Taylor Swift's most unusual songs?"
classes: wide
tags:
    - Python
    - anomaly detection
    - music
---

I recently [wrote a blog post analysing the musical evolution of Taylor Swift]({{ site.url }}{{ site.baseurl }}/analysing_taylor_swift/). Writing this post gave me a newfound appreciation for an artist that is constantly evolving. Some of my favourite findings involved finding outliers - songs that were in some way different to the majority. These outliers give us examples of where Taylor Swift has branched out the most. In this post, I’ll be using the Spotify data on Taylor Swift songs again. However, this time the goal is to identify Taylor Swift’s most unusual songs. Along the way, we’ll learn about an anomaly detection technique known as isolation forest.

# Getting the data

I will reuse the dataset of 136 Taylor Swift songs from my previous post. For simplicity, I’ll again focus on the seven Spotify audio features of acousticness, danceability, energy, instrumentalness, liveness, speechiness, and valence.

```python
df = pd.read_csv('taylor_swift_tracks.csv')
df = df[['album_name', 'track_name', 'acousticness', 'danceability', 'energy', 'instrumentalness', 'liveness', 'speechiness', 'valence']]
data_numeric_df = df.loc[:, 'acousticness': 'valence']
data_numeric = data_numeric_df.to_numpy()
```

# Isolation forests

To find the outliers, I’ll use an unsupervised anomaly detection technique known as an **isolation forest**. Most anomaly detection algorithms work by modelling the “normal” data points and then labelling anything outside of that distribution as anomalous. The isolation forest is different - it doesn't involve modelling what a "normal" data points look like, and is therefore said to be a "model-free" approach. The basic idea behind the isolation forest is to look for the most isolated points. The reasoning behind this is that it should be easier to “isolate” an outlier from other points than it is to isolate a normal point from other points.

Let's consider this in more detail. Suppose you were to build a binary decision tree with random splits. At each split, you select one feature and set an arbitrary threshold for that feature. This process creates partitions of the data. Intuitively, if a point is an outlier we would expect that it would be easier to separate from the other points. This is captured by the “isolation depth”, the number of partitions it takes to isolate a point. The lower the isolation depth, the more likely the point is to be an outlier. If you were just calculating one such “isolation tree”, the results could vary wildly between runs, so we ensemble isolation trees to get an “isolation forest”. From this, we calculate the **anomaly score** of each point.

It turns out that this original formulation of the isolation forest suffers from a bias which is remedied in a generalisation of the isolation forest known as the **extended isolation forest**. In the original formulation, branch cuts are always parallel to the coordinate axes, leading to a potential for bias and unwanted artefacts in the anomaly score heatmaps. In the extended isolation forest, branch cuts can occur in random directions and can even intersect with all axes (in the fully extended case). This leads to an algorithm with less bias. To learn more, I highly recommend reading [the paper on the extended isolation forest](https://ieeexplore.ieee.org/document/8888179). 

The extended isolation forest is available as the [Python package `eif`](https://github.com/ergodiclife/eif) which was actually written by the authors of the extended isolation forest paper.

In the code snippet below , I am running a fully extended extended isolation forest with 2,000 trees on the dataset of Taylor Swift songs. Note that due to the stochastic nature of the extended isolation forest, the results will vary somewhat every time you run the isolation forest. 

```python
forest = iso.iForest(data_numeric, 
                     ntrees=2_000, 
                     sample_size=len(data_numeric), 
                     ExtensionLevel=data_numeric.shape[1]-1
                    )
```

# Taylor Swift’s most unusual songs

Now for the moment we’ve been waiting for! Let’s get the top 10
outliers. The `compute_paths` method gives us the anomaly scores, so all we
have to do is sort the songs in descending order of the scores.

```python
anomaly_scores = forest.compute_paths(X_in=data_numeric)
df.loc[:,'anomaly_score'] = anomaly_scores
df.sort_values(by='anomaly_score', ascending=False).head(10)
```

![Taylor Swift
GIF]({{ site.url }}{{ site.baseurl }}/images/taylor-swift-isolation-forest/giphy.gif)

And *voilá*, here are Taylor Swift’s top 10 most unusual songs!

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: center;
    }
</style>
<table border="1" class="dataframe">
<colgroup>
<col style="width: 7%" />
<col style="width: 26%" />
<col style="width: 9%" />
<col style="width: 9%" />
<col style="width: 4%" />
<col style="width: 11%" />
<col style="width: 6%" />
<col style="width: 8%" />
<col style="width: 5%" />
<col style="width: 9%" />
</colgroup>
  <thead>
    <tr style="text-align: center;">
      <th>album_name</th>
      <th>track_name</th>
      <th>acousticness</th>
      <th>danceability</th>
      <th>energy</th>
      <th>instrumentalness</th>
      <th>liveness</th>
      <th>speechiness</th>
      <th>valence</th>
      <th>anomaly_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>evermore</td>
      <td>closure</td>
      <td>0.83</td>
      <td>0.69</td>
      <td>0.70</td>
      <td>0.00</td>
      <td>0.13</td>
      <td>0.25</td>
      <td>0.92</td>
      <td>0.62</td>
    </tr>
    <tr>
      <td>Lover</td>
      <td>I Forgot That You Existed</td>
      <td>0.30</td>
      <td>0.66</td>
      <td>0.32</td>
      <td>0.00</td>
      <td>0.08</td>
      <td>0.52</td>
      <td>0.54</td>
      <td>0.61</td>
    </tr>
    <tr>
      <td>Lover</td>
      <td>It’s Nice To Have A Friend</td>
      <td>0.97</td>
      <td>0.74</td>
      <td>0.18</td>
      <td>0.00</td>
      <td>0.17</td>
      <td>0.04</td>
      <td>0.55</td>
      <td>0.54</td>
    </tr>
    <tr>
      <td>1989</td>
      <td>Shake It Off</td>
      <td>0.06</td>
      <td>0.65</td>
      <td>0.78</td>
      <td>0.00</td>
      <td>0.15</td>
      <td>0.16</td>
      <td>0.94</td>
      <td>0.53</td>
    </tr>
    <tr>
      <td>evermore</td>
      <td>long story short</td>
      <td>0.67</td>
      <td>0.55</td>
      <td>0.73</td>
      <td>0.18</td>
      <td>0.10</td>
      <td>0.04</td>
      <td>0.57</td>
      <td>0.53</td>
    </tr>
    <tr>
      <td>Lover</td>
      <td>Cornelia Street</td>
      <td>0.78</td>
      <td>0.82</td>
      <td>0.62</td>
      <td>0.00</td>
      <td>0.10</td>
      <td>0.08</td>
      <td>0.25</td>
      <td>0.53</td>
    </tr>
    <tr>
      <td>Red</td>
      <td>Stay Stay Stay</td>
      <td>0.31</td>
      <td>0.73</td>
      <td>0.75</td>
      <td>0.00</td>
      <td>0.09</td>
      <td>0.02</td>
      <td>0.93</td>
      <td>0.52</td>
    </tr>
    <tr>
      <td>evermore</td>
      <td>willow</td>
      <td>0.84</td>
      <td>0.39</td>
      <td>0.58</td>
      <td>0.00</td>
      <td>0.14</td>
      <td>0.16</td>
      <td>0.55</td>
      <td>0.52</td>
    </tr>
    <tr>
      <td>Speak Now</td>
      <td>Mean</td>
      <td>0.45</td>
      <td>0.57</td>
      <td>0.76</td>
      <td>0.00</td>
      <td>0.22</td>
      <td>0.05</td>
      <td>0.79</td>
      <td>0.52</td>
    </tr>
    <tr>
      <td>Lover</td>
      <td>False God</td>
      <td>0.74</td>
      <td>0.74</td>
      <td>0.32</td>
      <td>0.00</td>
      <td>0.11</td>
      <td>0.24</td>
      <td>0.35</td>
      <td>0.52</td>
    </tr>
  </tbody>
</table>
</div>

We see that the single most unusual song for Taylor Swift is *“closure”* from *evermore*.

Let’s look at each of these songs in more detail. I’ll give the most likely reasons why the song was identified as an outlier - this is based on the exploratory data analysis from the previous post as well as an analysis of the Shapley values, a technique from interpretable Machine Learning (not included in this post for the sake of readability). Where relevant, I’ll also add quotes from critics and my own comments.

#### 1. *“closure”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=AIFnKqIeEdY))
*“closure”* is Taylor Swift’s most unusual song according to the model, with relatively high values for `speechiness` and `valence`. It has been described as *[“a wild industrial-folk number with Nine Inch Nails-style drums”](https://genius.com/Taylor-swift-closure-lyrics)*.

#### 2. *“I Forgot That You Existed”* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=p1cEvNn88jM))
*“I Forgot That You Existed”* is Taylor’s most “speechy” song with a `speechiness` value of 0.52. For a lot of the song, Taylor isn’t so much singing as she is talking.

#### 3. *"It's Nice To Have a Friend"* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=eaP1VswBF28))
*"It's Nice To Have a Friend"* is one of Taylor's least energetic songs, with an `energy` score of 0.18. It's got a lullaby quality to it and has *["choral backing vocals, gently gleaming steel drums and tenderly plucked harps"](https://www.billboard.com/music/pop/taylor-swift-its-nice-to-have-a-friend-lover-best-song-8528213/)*.

#### 4. *"Shake It Off"* from *1989* ([YouTube link](https://www.youtube.com/watch?v=nfWlot6h_JM))
The hit song *"Shake It Off"* has the highest `valence` in the dataset and also has a relatively high value for `speechiness`. It's an upbeat dance pop song that cemented Taylor Swift's transition from country music star to pop icon. The high `speechiness` is likely to be due to the spoken-word bridge in the song.

#### 5. *“long story short”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=rqQHa2HcGtM))
*“long story short”* has the highest value for `instrumentalness` out of all the songs considered (although it's not actually an instrumental song). The [Wikipedia page for the song](https://en.wikipedia.org/wiki/Long_Story_Short_(song)) describes *“long story short”* like this: *“Distinct from the album’s general softer pace,”Long Story Short” is an upbeat indie rock, folk-pop, and synth-pop song with elements of early 1980s pop, driven by energetic guitars, drums and strings.”* The song is fast-paced, with a tempo of 158 BPM and the drums were played by Bryan Devendorf of rock band The National.

#### 6. *"Cornelia Street"* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=bqJ9I-3MG1g))
According to the Spotify features, *"Cornelia Street"* is one of Taylor Swift's most *danceable* songs with a `danceability` score of 0.82. It's described as [*"a pop song instrumented by a keyboard line, pulsing synths in the background, and echoing vocals"*](https://en.wikipedia.org/wiki/Cornelia_Street_(song)).

#### 7. *"Stay Stay Stay"* from *Red* ([YouTube link](https://www.youtube.com/watch?v=QEYTzDJO_eM))
*"Stay Stay Stay"* is the song with the second-highest `valence` after *"Shake It Off"*. It's an upbeat country pop song.

#### 8. *“willow”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=RsEZmictANA))
*“willow”* has a relatively high value for `speechiness` for a Taylor Swift song (0.16) however that’s not the only reason it’s an outlier as there are other songs with higher `speechiness` values that have not made the list of the top 10 most unusual songs - it’s also one of her least danceable songs. The [Wikipedia page for the song](https://en.wikipedia.org/wiki/Willow_(song)) describes *“willow”* as a *“chamber folk ballad with Americana stylings, indie folk orchestration, tropical house accents…built around a glockenspiel, drum machines, cello, French horn, electric guitars, violin, flute, and orchestrations.”*

#### 9. *"Mean*" from *Speak Now* ([YouTube link](https://www.youtube.com/watch?v=jYa1eI1hpDE))
The country song *"Mean"* has a unique combination of `acousticness` and `valence` - other than *"closure"* it's the only song in this dataset where `valence` is greater than 0.75 and `acousticness` is greater than 0.35. Some critics suggest [it's the most country-leaning track from *Speak Now*](https://americansongwriter.com/taylor-swift-speak-now/).

#### 10. *"False God"* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=acQXa5ArHIk))
*“False God”* is another song with a high value for `speechiness`. Other than *"closure"* it's the only song in this dataset where `speechiness` is greater than 0.2 and `acousticness` is greater than 0.3. 

# Conclusion

In this post we’ve used anomaly detection to identify Taylor Swift’s most unusual songs and seen just how versatile she is as a musical artist.

# Further work

We could extend this approach to use a wider selection of musical features. We could also use the same approach to explore the catalogues of other musical artists.