---
title: "How has Taylor Swift's music evolved over time?"
classes: wide
tags:
    - R
    - data analysis
    - music
excerpt_separator: <!--more-->
---

In 2020, Taylor Swift surprised many of her fans with the album
*folklore*, which was a big departure from her usual style - think indie
folk music rather than the upbeat pop Swift is perhaps most known for.
It wasn’t the first time Swift had changed music styles though - she
started out as a country music star before becoming known as a pop icon.
Ahead of her upcoming release of the new album *Midnights* on October
21, I thought it would be interesting to explore how Taylor Swift’s
musical style has evolved over time - using data!

<!--more-->

![Taylor Swift albums]({{ site.url }}{{ site.baseurl }}/images/analysing-taylor-swift/taylor-swift-albums.png)

# Getting the data

For this project, I’ll be exploring data from Spotify using the
[spotifyr wrapper for the Spotify
API](https://www.rcharlie.com/spotifyr/). We can use the
`get_artist_audio_features` function from `spotifyr` to get the Spotify
metadata and audio features of all the tracks for a given artist.

To keep things simple, I’ll only include songs from studio albums, and
exclude songs from deluxe versions 💿, live versions 🎤, and
re-recordings ♻️. I also exclude karaoke versions, remixes, and radio
edits. If there are multiple songs with the same track name, I’ll take
the one with the earliest release date. This leaves us with 136 tracks
in total.

Now we’ve got a dataset of over a hundred Taylor Swift songs. The data
includes standard metadata for each track (album name, album release
date, duration, etc) as well as several audio features which we will
explore in the next section.

# Exploring the audio features 🎵

Here are the results we get back for one song:

    ## Rows: 1
    ## Columns: 39
    ## $ artist_name                  <chr> "Taylor Swift"
    ## $ artist_id                    <chr> "06HL4z0CvFAxyc27GXpf02"
    ## $ album_id                     <chr> "7mzrIsaAjnXihW3InKjlC3"
    ## $ album_type                   <chr> "album"
    ## $ album_images                 <list> [<data.frame[3 x 3]>]
    ## $ album_release_date           <chr> "2006-10-24"
    ## $ album_release_year           <dbl> 2006
    ## $ album_release_date_precision <chr> "day"
    ## $ danceability                 <dbl> 0.58
    ## $ energy                       <dbl> 0.491
    ## $ key                          <int> 0
    ## $ loudness                     <dbl> -6.462
    ## $ mode                         <int> 1
    ## $ speechiness                  <dbl> 0.0251
    ## $ acousticness                 <dbl> 0.575
    ## $ instrumentalness             <dbl> 0
    ## $ liveness                     <dbl> 0.121
    ## $ valence                      <dbl> 0.425
    ## $ tempo                        <dbl> 76.009
    ## $ track_id                     <chr> "0Om9WAB5RS09L80DyOfTNa"
    ## $ analysis_url                 <chr> "https://api.spotify.com/v1/audio-analysi…
    ## $ time_signature               <int> 4
    ## $ artists                      <list> [<data.frame[1 x 6]>]
    ## $ available_markets            <list> <"CA", "US">
    ## $ disc_number                  <int> 1
    ## $ duration_ms                  <int> 232106
    ## $ explicit                     <lgl> FALSE
    ## $ track_href                   <chr> "https://api.spotify.com/v1/tracks/0Om9WA…
    ## $ is_local                     <lgl> FALSE
    ## $ track_name                   <chr> "Tim McGraw"
    ## $ track_preview_url            <lgl> NA
    ## $ track_number                 <int> 1
    ## $ type                         <chr> "track"
    ## $ track_uri                    <chr> "spotify:track:0Om9WAB5RS09L80DyOfTNa"
    ## $ external_urls.spotify        <chr> "https://open.spotify.com/track/0Om9WAB5R…
    ## $ album_name                   <chr> "Taylor Swift"
    ## $ key_name                     <chr> "C"
    ## $ mode_name                    <chr> "major"
    ## $ key_mode                     <chr> "C major"

For each track, the Spotify dataset gives us audio features such as the
track’s tempo, key, loudness, and time signature. It also gives us the
following features which all take a value between 0 and 1:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Metric</th>
<th>Description</th>
<th>Values</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>acousticness</td>
<td>whether the track is acoustic</td>
<td>1.0 indicates a high confidence that the track is acoustic</td>
</tr>
<tr class="even">
<td>danceability</td>
<td>how suitable the track is for dancing based on a combination of
musical elements</td>
<td>0.0 is least danceable, 1.0 is most danceable</td>
</tr>
<tr class="odd">
<td>energy</td>
<td>measure of intensity and activity based on characteristics like
dynamic range and loudness</td>
<td>0.0 for low energy, 1.0 for high energy</td>
</tr>
<tr class="even">
<td>instrumentalness</td>
<td>whether the track contains no vocals</td>
<td><span class="math inline"> &gt; 0.5</span> represents songs that are
probably instrumental</td>
</tr>
<tr class="odd">
<td>liveness</td>
<td>whether there is an audience in the recording</td>
<td><span class="math inline"> &gt; 0.8</span> represents strong
likelihood that it is a live track</td>
</tr>
<tr class="even">
<td>speechiness</td>
<td>whether there are spoken words in a track</td>
<td>values between 0.33 and 0.66 represent tracks that contain both
music and speech, <span class="math inline"> &gt; 0.66</span> represents
tracks that are likely to be entirely spoken word</td>
</tr>
<tr class="odd">
<td>valence</td>
<td>musical positiveness conveyed by the track</td>
<td>high values sound more positive, low values sound more negative</td>
</tr>
</tbody>
</table>

For more information on all the features available, take a look at the
[Spotify API
reference](https://developer.spotify.com/documentation/web-api/reference/#/operations/get-several-audio-features).
Note that the documentation does not explain *how* Spotify calculates
these features, but my understanding is that they are all based on the
audio alone and do not take into account the song lyrics.

To keep things simple, I’ve decided to only consider these seven
features. Let’s look at these audio features in more detail using a
[raincloud
plot](https://www.cedricscherer.com/2021/06/06/visualizing-distributions-with-raincloud-plots-and-how-to-create-them-with-ggplot2/).
Using a raincloud, we see at a violin plot, box plot, and scatter plot all
in the same graph. This gives us a better understanding of the
distribution than looking a violin plot or box plot alone.

![Raincloud plot showing audio features of Taylor Swift songs]({{ site.url }}{{ site.baseurl }}/images/analysing-taylor-swift/1-raincloud-plot.png)

### Acousticness 🪕

There is a wide spread for the `acousticness` variable with songs taking
values between 0.00 and 0.97. The distribution is bimodal with two
peaks - the largest peak is at very low values of acousticness, but
there’s another peak at around 0.80.

### Danceability 🕺

The distribution of `danceability` is symmetric with a median value of
0.60 and an IQR of (0.53, 0.66). There are outliers on either end.

### Energy ⚡

The `energy` variable takes a wide range of values (from 0.18 to 0.95)
but the distribution is left-skewed and the majority of songs take
values above 0.5. **The most energetic song by this measure is
*“Haunted”*, a rock song from *Speak Now*. The least energetic song is
the acoustic ballad *“New Year’s Day”* from *reputation*.**

### Instrumentalness 🎹

I wasn’t sure whether to consider this variable because Taylor Swift
songs aren’t instrumental - they have all got vocals in them. In the
end, I decided to keep it out of interest to see how the Spotify
algorithm interprets different songs. As expected, most songs have very
low values (below 0.0001), however there are a few outliers.

### Liveness 🫶

This is another variable I wasn’t sure whether to include. `liveness` is
a confidence measure of whether there’s an audience in the song, but my
dataset doesn’t include live versions of songs. So it’s interesting to
see that, nevertheless, some songs have higher values of `liveness` than
others, with values as high as 0.38. That said, the Spotify
documentation says that only songs with a `liveness` above 0.8 are
likely to be live.

### Speechiness 💬

`speechiness` is a measure of whether there are spoken words in the
song. As expected, these values tend to be very low for Taylor Swift
songs. However, there is one song with a value above 0.33, which is the
threshold at which a song is likely to contain both music and speech.
**That song is *“I Forgot That You Existed”* from *Lover* with a
`speechiness` value of 0.52, suggesting it contains a mix of both music
and spoken word, and in fact, in parts of this song Taylor Swift is
talking rather than singing!**

### Valence ➕

The `valence` variable takes a wide range of values. The electropop
ballad *“Delicate”* from *reputation* has the lowest valence (0.05),
suggesting that it has a “negative” sound, while the upbeat pop song
*“Shake It Off”* from 1989 has the highest valence (0.94), suggesting
that it has a “positive” sound.

# How have these audio features changed over time? ⏱

Now that we’ve got an idea of the features we’re working with, let’s
look at how they’ve changed over time. The figure below shows a
scatterplot for each of the seven features of interest. Each point is a
track and the columns are arranged in order of album release date so we
can visualise changes over time. The diamond markers indicate the median
value of the attribute for that album and the error bars indicate the
interquartile range. There’s *a lot* of information in these plots!

![Audio features over time]({{ site.url }}{{ site.baseurl }}/images/analysing-taylor-swift/2-music-over-time.png)

So what can we learn about Taylor Swift’s musical evolution from these
plots?

### Acousticness 🪕

***folklore* and *evermore* are by far Taylor Swift’s most acoustic
albums, while *Red*, *1989*, and *reputation* are her least acoustic
albums.** The median values for acousticness were 0.83 and 0.74 for
*folklore* and *evermore*, respectively, but only 0.04, 0.06 and 0.07
for *Red*, *1989* and *reputation*.

### Danceability 🕺

**Taylor Swift albums released between 2012 and 2019 - *Red*, *1989*,
*reputation*, and *Lover* - are characterised by more danceable songs on
average.**

Interestingly, *Lover* has some of Taylor Swift’s most danceable as well
as least danceable songs - the most danceable song on the album is *“I
Think He Knows”*, while the least danceable song is *“The Archer”*. *evermore* is Taylor Swift’s least danceable album.

### Energy ⚡

*folklore* is Taylor Swift’s lowest energy album (with a median energy
of 0.40) while *1989* has the highest average energy (with a median
energy of 0.74). There are differences between *folklore* and
*evermore* - *evermore* is more energetic on average than *folklore*,
with a median value of 0.51.

### Instrumentalness 🎹

Most songs tend to have near-zero values for `instrumentalness`. The
exceptions are *“long story short”* and *“gold rush”* from *evermore*
(at 0.18 and 0.11 respectively). However, these are fairly low values
for `instrumentalness` and still represent songs with vocals.

### Liveness 🫶

*Speak Now* is the album with the highest average value for `liveness`
but *“This Is Why We Can’t Have Nice Things”* from *reputation* is the
song with the highest value of `liveness`.

### Speechiness 💬

`speechiness` tends to be near-zero, however there are some outliers.
**From *1989* (2014) onwards, we start to see some songs with
speechiness values above 0.10. Remember the spoken-word bridge in
*“Shake It Off”*? Since then, Taylor Swift has experimented with spoken
word on other songs, such as *“I Forgot That You Existed”* and *“False
God”* on *Lover*, and *“closure”* on *evermore*.**

### Valence ➕
*reputation* has the lowest median value for `valence`, suggesting that it has the most "negative" sound, while *Red*, *1989* and *Lover* are the albums with the most "positive" sound, on average.

### Most eclectic album

***Lover* is Taylor Swift’s most eclectic album.** This album takes the
greatest interquartile range out of all the albums for several of the
variables. It also contains outliers for features including
acousticness, energy, danceability, and speechiness. The album’s
eclectic style was noted by critics, with New York Times saying [*“there
isn’t a consistent musical throughline so much as a slate of options,
some familiar and some
new”*](https://www.nytimes.com/2019/08/23/arts/music/taylor-swift-lover-review.html),
and NME describing it as
[*“sprawling”*](https://www.nme.com/reviews/taylor-swift-lover-review-2541084).

# Visualising Taylor Swift songs using PCA

We’ve made several plots showing how the audio features have changed
over time. To summarise these features more succinctly in one plot, I’ll
use Principal Component Analysis (PCA). I have chosen not to standardise
the variables as they are all measured on the same scale (0 to 1).

We can see that the first component captures 65% of the variation of the
data - not bad! The second PC captures nearly 20% of variation. Let’s
take a look at the principal components themselves. We’ll focus on the
first two principal components, which together explain over 80% of the
variation.

![Principal components for Taylor Swift songs]({{ site.url }}{{ site.baseurl }}/images/analysing-taylor-swift/3-principal-components.png)

The table below is my attempt to label the two PCs. I’ve taken
electronic to be the opposite of acoustic.

<table>
<colgroup>
<col style="width: 9%" />
<col style="width: 25%" />
<col style="width: 32%" />
<col style="width: 32%" />
</colgroup>
<thead>
<tr class="header">
<th>PC</th>
<th>Label</th>
<th>Highest track</th>
<th>Lowest track</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PC1</td>
<td>Electronic and high energy</td>
<td><em>The Story Of Us</em> from <em>Speak Now</em></td>
<td><em>New Year’s Day</em> from <em>reputation</em></td>
</tr>
<tr class="even">
<td>PC2</td>
<td>High valence</td>
<td><em>closure</em> from <em>evermore</em></td>
<td><em>The Last Time</em> from <em>Red</em></td>
</tr>
</tbody>
</table>

The presence of *“closure”* from *evermore* as a song with high valence
shows the limitation of our approach, which uses automatically
calculated features from Spotify and does not take into account song
lyrics. Based on this, one might think that *closure* is a positive
song, but the lyrics certainly suggest otherwise.

The next plot shows all 136 songs in PC1-PC2 space, colour-coded by
album. This plot is interactive, so have an explore! Click onto the
graph and then if you hover over a point it will bring up the name of
the song.

<iframe src="{{ site.url }}{{ site.baseurl }}/interactive_plots/p1.html" height="600px" width="100%" style="border:none;"></iframe>

Using the PC1-PC2 plot, we can make the following observations:

-   Most tracks from *folklore* and *evermore* occupy the bottom-left
    area of the plot. Some exceptions include *“closure”* from
    *evermore* and *“the last great american dynasty”* from *folklore*.
-   Songs from *Lover* are all over the place on this graph, with *“It’s
    Nice to Have A Friend”* at PC1 = -0.72 to *“Paper Rings”* at PC1 =
    0.42. Again, this suggests that *Lover* is an eclectic album.
-   There are several songs from earlier albums which are close in
    PC1-PC2 space to songs from *folklore* and *evermore*. We can think
    of these songs as being in some ways similar to one another and
    we’ll explore this in more detail in the next section.
-   *Red* and *1989* are very similarly distributed in PC1-PC2 space,
    with most songs in the bottom-right quadrant.
-   Songs from *reputation* are in the bottom-right quadrant, with the
    exception of *“New Year’s Day”*. The data is clearly suggesting this
    song is different to other songs from the album. And in fact, this
    is confirmed by the following quote from [the Wikipedia page for
    this
    song](https://en.wikipedia.org/wiki/New_Year%27s_Day_(Taylor_Swift_song)):
    *“New Year’s Day” is an acoustic piano ballad with delicate piano
    riffs and occasional guitar and synth notes, a stark contrast with
    Reputation’s heavily electronic production.”*
-   A note of warning: just because two songs are next to each other on
    the PC1-PC2 plot does not mean they necessarily sound similar. For
    instance, *“A Place in this World”* from *Taylor Swift* and *“…Ready
    For It?”* from *reputation* are right next to each other but sound
    very different - one is a country pop song and the other is an
    electro pop song with heavy synths and rapping. However, both of
    these songs have very similar values for acousticness, danceability,
    energy, instrumentalness, and valence.

# Exploring Taylor Swift’s songs using K-means clustering

Using PCA, we were able to visualise Taylor Swift’s songs in
two-dimensional space. We started to explore the similarities (and
differences) between songs. However, it can be difficult to see which
songs should be grouped together. K-means clustering can give us the
“clusters” of similar songs, taking into account all seven variables.
This technique requires us to choose a value for *K*, the number of
clusters. Using the elbow method, I found the optimal number of clusters
to be 3.

If we look at the output of K-means, we see the average values for each
feature within each cluster.

<table>
<colgroup>
<col style="width: 3%" />
<col style="width: 15%" />
<col style="width: 15%" />
<col style="width: 8%" />
<col style="width: 20%" />
<col style="width: 10%" />
<col style="width: 14%" />
<col style="width: 9%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;"></th>
<th style="text-align: right;">acousticness</th>
<th style="text-align: right;">danceability</th>
<th style="text-align: right;">energy</th>
<th style="text-align: right;">instrumentalness</th>
<th style="text-align: right;">liveness</th>
<th style="text-align: right;">speechiness</th>
<th style="text-align: right;">valence</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">1</td>
<td style="text-align: right;">0.07</td>
<td style="text-align: right;">0.64</td>
<td style="text-align: right;">0.75</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">0.16</td>
<td style="text-align: right;">0.05</td>
<td style="text-align: right;">0.58</td>
</tr>
<tr class="even">
<td style="text-align: left;">2</td>
<td style="text-align: right;">0.14</td>
<td style="text-align: right;">0.59</td>
<td style="text-align: right;">0.56</td>
<td style="text-align: right;">0.00</td>
<td style="text-align: right;">0.14</td>
<td style="text-align: right;">0.06</td>
<td style="text-align: right;">0.28</td>
</tr>
<tr class="odd">
<td style="text-align: left;">3</td>
<td style="text-align: right;">0.76</td>
<td style="text-align: right;">0.55</td>
<td style="text-align: right;">0.43</td>
<td style="text-align: right;">0.01</td>
<td style="text-align: right;">0.12</td>
<td style="text-align: right;">0.05</td>
<td style="text-align: right;">0.36</td>
</tr>
</tbody>
</table>

It’s difficult to assign labels to these clusters because they do not
correspond neatly to music genres. However, we can say that cluster 1 is
the most upbeat, cluster 2 less so, and cluster 3 is the most acoustic.

The plot below shows the three clusters in PC1-PC2 space, with
confidence ellipses drawn around each cluster.

![Cluster plot for Taylor Swift songs where k=3]({{ site.url }}{{ site.baseurl }}/images/analysing-taylor-swift/4-cluster-plot.png)

Songs from *folklore* and *evermore* sit predominantly in cluster 3, with only three exceptions.
However, cluster 3 isn’t *just* songs from *folklore* and *evermore* -
of the 43 songs in the highly acoustic cluster 3, fifteen are from
earlier albums. These can be thought of as the songs most similar to
*folklore* and *evermore*. The table below show these songs in order of
release date.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Album</th>
<th style="text-align: left;">Track</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Taylor Swift</td>
<td style="text-align: left;">Tim McGraw</td>
</tr>
<tr class="even">
<td style="text-align: left;">Taylor Swift</td>
<td style="text-align: left;">Tied Together with a Smile</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Taylor Swift</td>
<td style="text-align: left;">Invisible</td>
</tr>
<tr class="even">
<td style="text-align: left;">Fearless</td>
<td style="text-align: left;">The Best Day</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Speak Now</td>
<td style="text-align: left;">Never Grow Up</td>
</tr>
<tr class="even">
<td style="text-align: left;">Speak Now</td>
<td style="text-align: left;">Last Kiss</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Red</td>
<td style="text-align: left;">Sad Beautiful Tragic</td>
</tr>
<tr class="even">
<td style="text-align: left;">1989</td>
<td style="text-align: left;">This Love</td>
</tr>
<tr class="odd">
<td style="text-align: left;">reputation</td>
<td style="text-align: left;">New Year’s Day</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lover</td>
<td style="text-align: left;">Lover</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lover</td>
<td style="text-align: left;">Cornelia Street</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lover</td>
<td style="text-align: left;">Soon You’ll Get Better (feat. The
Chicks)</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lover</td>
<td style="text-align: left;">False God</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lover</td>
<td style="text-align: left;">It’s Nice To Have A Friend</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lover</td>
<td style="text-align: left;">Daylight</td>
</tr>
</tbody>
</table>

This suggests that while *folklore* and *evermore* were a departure for
Taylor Swift, there are earlier songs that are similar in terms of audio
features. Interestingly, Taylor Swift [has described "folklore" as "sad, beautiful, tragic"](https://taylorswifts.fandom.com/wiki/Folklore) which is also the name of one of the songs on this list. And some Taylor Swift fans online regard *"It's Nice To Have A Friend"* as a "precursor" to *folklore*. 

# Limitations

As always it’s important to be aware of the limitations of this
analysis:

-   We have not taken into account song lyrics at all.
-   The principal components and clusters do not seem able to capture
    the differences between genres, such as country pop and electro pop.
    For instance, we found some songs that are very similar in terms of
    the first two principal components but actually sound very
    different.

# Conclusion

In this post, we used Spotify data to explore how Taylor Swift’s music
has changed over time.

This post was really fun to write! I learned a lot about the musical
stylings of Taylor Swift, which I wasn’t too familiar with when I first
started exploring the data. I’d heard that *folklore* (and subsequently
*evermore*) had taken people by surprise, so it was interesting to see
that these albums do in fact look different to other albums according to
the data. But what is perhaps even more interesting is discovering that
there are earlier songs, such as *“New Year’s Day”* from *reputation*,
that are similar.

In addition to finding Taylor Swift’s most danceable albums or most
speechy songs, my favourite findings included finding that *Lover* is
Taylor Swift’s most eclectic album and has been described as “sprawling”
and that the song *“New Year’s Day”* is different to other songs on the
album *reputation*.