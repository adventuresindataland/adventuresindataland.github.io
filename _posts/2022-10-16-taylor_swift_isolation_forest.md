---
title: "What are Taylor Swift's most unusual songs?"
classes: wide
tags:
    - R, anomaly detection, music
---

I recently [wrote a blog post analysing the musical evolution of Taylor
Swift]({{ site.url }}{{ site.baseurl }}/analysing_taylor_swift/).
Writing this post gave me a newfound appreciation for an artist that is
constantly evolving. Some of my favourite findings involved finding
outliers - songs that were in some way different to the majority. These outliers give us examples of
where Taylor Swift has branched out the most. In this post, I’ll be using the Spotify data on Taylor Swift songs again. However,
this time the goal is to identify Taylor Swift’s most unusual songs.
Along the way, we’ll learn about an anomaly detection technique known as
isolation forests.

# Getting the data

I will reuse the dataset of 136 Taylor Swift songs from my previous
post. For simplicity, I’ll again focus on the seven Spotify audio
features of acousticness, danceability, energy, instrumentalness,
liveness, speechiness, and valence.

    artist <- read.csv(file = 'taylor_swift_tracks.csv')
    artist_matrix <- artist %>%
      select(album_name, track_name, acousticness, danceability, energy, instrumentalness, liveness, speechiness, valence)
    data <- artist_matrix %>% select(where(is.numeric))

# Isolation forests

To find the outliers, I’ll use an unsupervised anomaly detection
technique known as an **isolation forest**. Most anomaly detection
algorithms work by modelling the “normal” data points and then labelling
anything outside of that as anomalies. The isolation forest is
different - it looks for the most isolated points. The reasoning behind
this is that it should be easier to “isolate” an outlier from other
points than it is to isolate a normal point from other points.

Suppose you were to build a binary decision tree with random splits,
setting arbitrary thresholds for various features. Intuitively, if a
point is an outlier we would expect that it would be easier to separate
from the other points. This is captured by the “isolation depth”, the
number of partitions it takes to isolate a point. The lower the
isolation depth, the more likely the point is to be an outlier. If you
were just calculating one such “isolation tree”, the results could vary
wildly between runs, so we ensemble isolation trees to get an “isolation
forest”. From this, we calculate the anomaly score of each point.

To use an isolation forest in R, I’ll use the [isotree R
package](https://github.com/david-cortes/isotree). We run the isolation
forest using the `isolation.forest` command. This supports some of the
different variants of isolation forests that have been developed over
the years. In this case, I’ve chosen to use an **extended isolation
forest**. In `isotree`, this is done by setting `ndim` to a value
greater than 1. I’ve set `ndim=2` which means that each split uses two
features. This approach is more robust than the original isolation
forest.

    iso <- isolation.forest(
        data,
        ndim=2, 
        sample_size="None",
        ntrees=100
    )
    isotree.export.model(iso, 'taylor_swift_isolation_forest', add_metadata_file = TRUE)

    anomaly_scores <- predict(iso, data)
    artist_and_anomaly_scores <- cbind(artist_matrix, anomaly_scores)
    write.csv(artist_and_anomaly_scores, 'taylor_swift_anomaly_scores.csv')

# Taylor Swift’s most unusual songs

Now for the moment we’ve been waiting for! Let’s get the top 10
outliers. The `predict` method gives us the anomaly scores, so all we
have to do is sort the songs in descending order of the scores.

![Taylor Swift
GIF]({{ site.url }}{{ site.baseurl }}/images/taylor-swift-isolation-forest/giphy.gif)


    outliers <- artist_and_anomaly_scores %>% 
      arrange(desc(anomaly_scores)) %>%
      head(10)

And *voilá*, here are Taylor Swift’s top 10 most unusual songs!

<table>
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
<tr class="header">
<th style="text-align: center;">Album</th>
<th style="text-align: center;">Track</th>
<th style="text-align: center;">Acousticness</th>
<th style="text-align: center;">Danceability</th>
<th style="text-align: center;">Energy</th>
<th style="text-align: center;">Instrumentalness</th>
<th style="text-align: center;">Liveness</th>
<th style="text-align: center;">Speechiness</th>
<th style="text-align: center;">Valence</th>
<th style="text-align: center;">Anomaly Score</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">evermore</td>
<td style="text-align: center;">long story short</td>
<td style="text-align: center;">0.67</td>
<td style="text-align: center;">0.55</td>
<td style="text-align: center;">0.73</td>
<td style="text-align: center;">0.18</td>
<td style="text-align: center;">0.10</td>
<td style="text-align: center;">0.04</td>
<td style="text-align: center;">0.57</td>
<td style="text-align: center;">0.641</td>
</tr>
<tr class="even">
<td style="text-align: center;">Lover</td>
<td style="text-align: center;">I Forgot That You Existed</td>
<td style="text-align: center;">0.30</td>
<td style="text-align: center;">0.66</td>
<td style="text-align: center;">0.32</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.08</td>
<td style="text-align: center;">0.52</td>
<td style="text-align: center;">0.54</td>
<td style="text-align: center;">0.621</td>
</tr>
<tr class="odd">
<td style="text-align: center;">evermore</td>
<td style="text-align: center;">gold rush</td>
<td style="text-align: center;">0.81</td>
<td style="text-align: center;">0.50</td>
<td style="text-align: center;">0.47</td>
<td style="text-align: center;">0.11</td>
<td style="text-align: center;">0.12</td>
<td style="text-align: center;">0.04</td>
<td style="text-align: center;">0.35</td>
<td style="text-align: center;">0.582</td>
</tr>
<tr class="even">
<td style="text-align: center;">evermore</td>
<td style="text-align: center;">closure</td>
<td style="text-align: center;">0.83</td>
<td style="text-align: center;">0.69</td>
<td style="text-align: center;">0.70</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.13</td>
<td style="text-align: center;">0.25</td>
<td style="text-align: center;">0.92</td>
<td style="text-align: center;">0.576</td>
</tr>
<tr class="odd">
<td style="text-align: center;">evermore</td>
<td style="text-align: center;">willow</td>
<td style="text-align: center;">0.84</td>
<td style="text-align: center;">0.39</td>
<td style="text-align: center;">0.58</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.14</td>
<td style="text-align: center;">0.16</td>
<td style="text-align: center;">0.55</td>
<td style="text-align: center;">0.535</td>
</tr>
<tr class="even">
<td style="text-align: center;">Lover</td>
<td style="text-align: center;">False God</td>
<td style="text-align: center;">0.74</td>
<td style="text-align: center;">0.74</td>
<td style="text-align: center;">0.32</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.11</td>
<td style="text-align: center;">0.24</td>
<td style="text-align: center;">0.35</td>
<td style="text-align: center;">0.531</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Lover</td>
<td style="text-align: center;">The Archer</td>
<td style="text-align: center;">0.12</td>
<td style="text-align: center;">0.29</td>
<td style="text-align: center;">0.57</td>
<td style="text-align: center;">0.01</td>
<td style="text-align: center;">0.07</td>
<td style="text-align: center;">0.04</td>
<td style="text-align: center;">0.17</td>
<td style="text-align: center;">0.525</td>
</tr>
<tr class="even">
<td style="text-align: center;">reputation</td>
<td style="text-align: center;">This Is Why We Can’t Have Nice Things</td>
<td style="text-align: center;">0.02</td>
<td style="text-align: center;">0.57</td>
<td style="text-align: center;">0.79</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.38</td>
<td style="text-align: center;">0.12</td>
<td style="text-align: center;">0.44</td>
<td style="text-align: center;">0.513</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Speak Now</td>
<td style="text-align: center;">Better Than Revenge</td>
<td style="text-align: center;">0.01</td>
<td style="text-align: center;">0.52</td>
<td style="text-align: center;">0.92</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.36</td>
<td style="text-align: center;">0.08</td>
<td style="text-align: center;">0.64</td>
<td style="text-align: center;">0.512</td>
</tr>
<tr class="even">
<td style="text-align: center;">reputation</td>
<td style="text-align: center;">Call It What You Want</td>
<td style="text-align: center;">0.19</td>
<td style="text-align: center;">0.60</td>
<td style="text-align: center;">0.50</td>
<td style="text-align: center;">0.00</td>
<td style="text-align: center;">0.34</td>
<td style="text-align: center;">0.07</td>
<td style="text-align: center;">0.25</td>
<td style="text-align: center;">0.503</td>
</tr>
</tbody>
</table>

We see that the single most unusual song for Taylor Swift is *“long
story short”* from *evermore*.

Let’s look at each of these songs in more detail. I’ll give the most
likely reasons *why* the song was identified as an outlier - this is
based on the exploratory data analysis from the previous post as well as
an analysis of the Shapley values, a technique from interpretable
Machine Learning (not included in this post for the sake of
readability). Where relevant, I’ll also add quotes from critics and my
own comments.

##### 1. *“long story short”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=rqQHa2HcGtM))

*“long story short”* is Taylor Swift’s most unusual song according to
the model and the reason for this is that it has the highest value for
`instrumentalness` out of all the songs considered (although it is not
actually an instrumental song). The [Wikipedia page for the
song](https://en.wikipedia.org/wiki/Long_Story_Short_(song)) describes
*“long story short”* like this: *“Distinct from the album’s general
softer pace,”Long Story Short” is an upbeat indie rock, folk-pop, and
synth-pop song with elements of early 1980s pop, driven by energetic
guitars, drums and strings.”* The song is fast-paced, with a tempo of
158 BPM and the drums were played by Bryan Devendorf of rock band The
National.

##### 2. *“I Forgot That You Existed”* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=p1cEvNn88jM))

*“I Forgot That You Existed”* is Taylor’s most “speechy” song with a
`speechiness` value of 0.52. For a lot of the song, Taylor isn’t so much
singing as she is talking.

##### 3. *“gold rush”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=Pz-f9mM3Ms8))

*“gold rush”* is the song with the second highest value for
`instrumentalness`. The first 25 seconds of the song feel very ethereal.

##### 4. *“closure”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=AIFnKqIeEdY))

*“closure”* is another song with a high value for `speechiness`. It’s
been [described as “a wild industrial-folk number with Nine Inch
Nails-style drums”](https://genius.com/Taylor-swift-closure-lyrics).

##### 5. *“willow”* from *evermore* ([YouTube link](https://www.youtube.com/watch?v=RsEZmictANA))

*“willow”* also has a relatively high value for `speechiness` for a
Taylor Swift song (0.16) however that’s not the only reason it’s an
outlier as there are other songs with higher `speechiness` values that
have not made the list of the top 10 most unusual songs - it’s also one
of her least danceable songs. The [Wikipedia page for the
song](https://en.wikipedia.org/wiki/Willow_(song)) describes *“willow”*
as a *“chamber folk ballad with Americana stylings, indie folk
orchestration, tropical house accents…built around a glockenspiel, drum
machines, cello, French horn, electric guitars, violin, flute, and
orchestrations.”*

##### 6. *“False God”* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=acQXa5ArHIk))

*“False God”* is another song with a high value for `speechiness`.
Unlike *“willow”*, it’s got a relatively high value for `danceability`
(it’s in the top 12%) and a relatively low value for `energy`.

##### 7. *“The Archer”* from *Lover* ([YouTube link](https://www.youtube.com/watch?v=8KpKc3C9V3w))

At a `danceability` of 0.29, *“The Archer”* is a very un-danceable song
for Taylor Swift.
[Wikipedia](https://en.wikipedia.org/wiki/The_Archer_(song)) describes
it as *“a 1980s-influenced ambient ballad combining synth-pop,
synthwave, and dream pop, incorporating heavy skittering synthesizers,
soft house beats, minimalistic elements and a slow groove.”*

##### 8. *“This Is Why We Can’t Have Nice Things”* from *reputation* ([YouTube link](https://www.youtube.com/watch?v=6Z3QJ4L1Bg0))

*“This Is Why We Can’t Have Nice Things”* is characterised by relatively
high values for `liveness` and `speechiness`. There are definitely parts
of the song where Taylor is talking rather than singing. There are also
sounds of people cheering in the background, which may have contributed
to a high score for `liveness`.

##### 9. *“Better Than Revenge”* from *Speak Now* ([YouTube link](https://www.youtube.com/watch?v=ziRR-doZAtg))

“Better than Revenge” is one of Taylor Swift’s most energetic songs with
an `energy` score of 0.92 and it has a relatively high value for
`liveness`. It’s [“an electric guitar-driven pop punk
song”](https://en.wikipedia.org/wiki/Better_than_Revenge) and was a big
departure from her typical country pop style at the time. It’s been
compared to the song *“Misery Business”* by Paramore.

##### 10. *“Call It What You Want”* from *reputation* ([YouTube link](https://www.youtube.com/watch?v=V54CEElTF_U))

*“Call It What You Want”* has a relatively high score for `liveness` for
a song that isn’t actually live. It features Taylor Swift rapping
although it does not have a particularly high value for `speechiness`.

# Conclusion

In this post we’ve used anomaly detection to identify Taylor Swift’s
most unusual songs and seen just how versatile she is as a musical
artist.

# Further work

We could extend this approach to use a wider selection of musical
features. We could also use the same approach to explore the catalogues
of other musical artists.
