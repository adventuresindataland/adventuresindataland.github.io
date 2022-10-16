---
title: "What movies do critics and audiences disagree on?"
classes: wide
tags:
    - R
    - data analysis
    - movies
---

*Rotten Tomatoes* is a film site that shows reviews from both critics
and audiences. Movies are classed as “fresh” or “rotten” depending on
the critics’ reviews. However, critics and audiences aren’t always in
agreement. In this post, I’ll be analysing the data to find the movies that critics
and audiences disagree on the most.

To see the code for this project, please check out the associated repo on GitHub.

Let’s get started!

# Data

I’ll be using [this Rotten Tomatoes dataset from
Kaggle](https://www.kaggle.com/stefanoleone992/rotten-tomatoes-movies-and-critic-reviews-dataset).
It looks like some movies are missing from the dataset (there’s less
than 18k rows but over 22k movies on the *Rotten Tomatoes* site).
However, because this project is just for fun, we won’t worry about this
and just accept that we haven’t got an exhaustive list.

# Data pre-processing

The dataset includes both feature films and documentaries. I want to
focus on feature films - there are a lot of political documentaries that
have polarising reviews on Rotten Tomatoes, but that’s not the focus of
this post.

To do this, I’ll **remove movies where “Documentary” is the primary
genre**. The dataset has a column called `genres` which contains a list
of comma-separated genres, with the first genre being the primary one. I
use this to create a new column called `primary_genre` and filter
appropriately.

I **take the top 5,000 movies** according to `audience_count`, the
number of audience members that reviewed the movie. This means that the
rating won’t get too swayed by a few scores and we should be able to
recognise most of the movies. I save this to a new dataframe called
`top_rt_movies`.

Now all of the movies in `top_rt_movies` have over 17,000 audience
reviews and at least 5 critic’s reviews.

# Exploratory data analysis

## Score distribution

Let’s look at the distribution of critics and audience scores. The
`tomatometer_rating` variable gives the average critic score and the
`audience_rating` variable gives the average audience score. Both scores
are on a scale from 0 to 100.

![Distribution of scores]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/movie-scores-distribution.png)

The density and box plots both show that the critic and audiences scores
follow different distributions. The critics’ scores have a much wider
distribution and take on more extreme values than audience scores. On
the other hand, audience scores have a much narrower distribution. Very
few films achieve an audience score below 25%. Similarly, not many
movies have an average audience score above 90%. In fact, out of the
5,000 movies considered, only 317 achieved an audience score above 90%,
but 644 achieved a critics’ score above 90%. It’s important to keep
these differences in mind as we continue our analysis.

# What movies do critics and audiences disagree on?

Having done the pre-processing and some exploratory analysis, it’s time
to find the movies that critics and audiences disagree on the most!

We define a new variable called **critical disconnect** as

$$\text{critical_disconnect = tomatometer_rating - audience_rating}$$

where a *positive* value means the *critics rated the movie higher than
audiences*, and a *negative* value means the *audiences rated the movie
higher than the critics*.

![Distribution of critical disconnect scores]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/disconnect-histogram.png)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##  -76.00  -19.00   -4.00   -6.94    6.00   47.00

The median “critical disconnect” is negative, meaning that the majority
of movies were rated higher by audiences than by critics. Furthermore,
the distribution is left-skewed, meaning there were some movies that
audiences rated *a lot* higher than critics.

### What movies do critics rate much higher than audiences?

To find the top 10 movies that critics rated higher than audiences, we
look at movies in decreasing order of *critical disconnect*.

![Movies that critics rated higher than audiences]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/movies-critics-higher.png)

We make the following observations:

-   A few of the films - *Spy Kids*, *Antz* and *Stuart Little 2* - are
    aimed at kids. It’s interesting to see that there’s so much
    disagreement on these between critics and audiences.
-   *Star Wars: The Last Jedi* is tied as the top movie critics rated
    higher than audiences (along with *Spy Kids*) and this was a
    divisive film among fans of *Star Wars*. However, it is also
    possible that [this movie fell victim to a “review-bombing campaign”
    on *Rotten Tomatoes*, with some users giving a negative rating
    without having even seen the
    movie](https://www.theverge.com/2019/3/7/18254548/film-review-sites-captain-marvel-bombing-changes-rotten-tomatoes-letterboxd).
-   *Hail Caesar!* is a movie by the Coen brothers about the Hollywood
    film industry and stars big names such as George Clooney. Despite
    being loved by the critics, it was received poorly by audiences. A
    review from IGN said *“it might not engage viewers beyond Los
    Angeles or those who truly understand or work in the film
    industry—but it’s nevertheless a fun, charming, and oft-hilarious
    take on Hollywood’s Golden Age.”*
-   Both *Arachnophobia* and *Nurse Betty* are described as “black
    comedy” films on Wikipedia, which typically cover darker subject
    matter than other comedies.
-   *About a Boy*, a rom-com starring Hugh Grant, also achieved a much
    higher score from critics than audiences. Interestingly, on IMDb the
    audience score for this movie was high and the majority of top user
    reviews there are highly favourable. This raises the question of
    whether *Rotten Tomatoes* users are different to users on *IMDb*.

### What movies do audiences rate much higher than critics?

We can also look at the lowest negative values of *critical disconnect*
to get the movies that audiences rated much higher than critics.

![Movies that audiences rated higher than critics]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/movies-audiences-higher.png)

We make the following observations:

-   The majority of these movies are comedies.
-   *A Low Down Dirty Shame* is [one of very few films with a 0%
    tomatometer
    rating](https://en.wikipedia.org/wiki/List_of_films_with_a_0%25_rating_on_Rotten_Tomatoes).
-   Both *Belly* and *Drop Dead Fred* have achieved somewhat of a cult
    following.
-   Three of the movies on the list are by
    actor/director/producer/screenwriter Tyler Perry.

# Critical disconnect by movie genres

We saw that the majority of the top 10 movies that audiences rated
higher than critics were comedies. So is there a relationship between
the genre of a movie and the level of critical disconnect?

Let’s look at the average *critical disconnect* for each genre. Note
that we are still just looking at the sample of the top 5,000 movies.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 20%" />
<col style="width: 22%" />
<col style="width: 5%" />
<col style="width: 26%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: center;">Primary genre</th>
<th style="text-align: center;">Average critic score</th>
<th style="text-align: center;">Average audience score</th>
<th style="text-align: center;">Count</th>
<th style="text-align: center;">Average critical disconnect</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">Classics</td>
<td style="text-align: center;">86.9</td>
<td style="text-align: center;">84.0</td>
<td style="text-align: center;">172</td>
<td style="text-align: center;">2.9</td>
</tr>
<tr class="even">
<td style="text-align: center;">Art House &amp; International</td>
<td style="text-align: center;">77.3</td>
<td style="text-align: center;">77.4</td>
<td style="text-align: center;">211</td>
<td style="text-align: center;">-0.1</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Animation</td>
<td style="text-align: center;">65.0</td>
<td style="text-align: center;">67.2</td>
<td style="text-align: center;">190</td>
<td style="text-align: center;">-2.2</td>
</tr>
<tr class="even">
<td style="text-align: center;">Horror</td>
<td style="text-align: center;">42.8</td>
<td style="text-align: center;">48.6</td>
<td style="text-align: center;">285</td>
<td style="text-align: center;">-5.8</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Drama</td>
<td style="text-align: center;">62.9</td>
<td style="text-align: center;">69.9</td>
<td style="text-align: center;">995</td>
<td style="text-align: center;">-7.0</td>
</tr>
<tr class="even">
<td style="text-align: center;">Action &amp; Adventure</td>
<td style="text-align: center;">53.6</td>
<td style="text-align: center;">61.0</td>
<td style="text-align: center;">1600</td>
<td style="text-align: center;">-7.5</td>
</tr>
<tr class="odd">
<td style="text-align: center;">Comedy</td>
<td style="text-align: center;">51.0</td>
<td style="text-align: center;">60.5</td>
<td style="text-align: center;">1424</td>
<td style="text-align: center;">-9.5</td>
</tr>
</tbody>
</table>

In the plot below, each point represents the average critics and
audience score for that genre. The dotted line is the line where average
critics score equals the average audience score.

![Critical disconnect by movie genre]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/disconnect-by-genre.png)

This suggests that the magnitude of the *critical disconnect* does vary
by genre. The genres with the greatest *critical disconnect* are
“Comedy”, “Action & Adventure”, and Drama”. Movies in these genres
received on average 9.5, 7.4, and 7 more points from audiences compared
to critics, respectively. For the small “Art House & International”
genre, audiences and critics average scores differed by only 0.1 points
on average.

In fact, every genre had a higher average audience score, with the
exception of the small “Classics” genre which contains older movies from
a variety of genres. When I replaced “Classics” with the secondary genre
for these, the results were similar to those described above.

So on average, audiences are more generous than critics for every major
movie genre. However, they are more generous for some movie genres than
others.

# Critical disconnect by decade

Let’s also take a look at movie decades.

![Decade distribution of movies]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/movies-decade-distribution.png)

The majority of movies in our dataset are from the 21st century,
however, we do also have older movies, particularly from the 80s and
90s. We can do a similar analysis looking at the decade the movie was
released. Remember, the majority of the movies in our dataset are from
the 21st century.

<table>
<colgroup>
<col style="width: 23%" />
<col style="width: 20%" />
<col style="width: 22%" />
<col style="width: 5%" />
<col style="width: 27%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: center;">Original release decade</th>
<th style="text-align: center;">Average critic score</th>
<th style="text-align: center;">Average audience score</th>
<th style="text-align: center;">Count</th>
<th style="text-align: center;">Average critical disconnect</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: center;">1960</td>
<td style="text-align: center;">86.2</td>
<td style="text-align: center;">82.6</td>
<td style="text-align: center;">107</td>
<td style="text-align: center;">3.6</td>
</tr>
<tr class="even">
<td style="text-align: center;">1970</td>
<td style="text-align: center;">79.0</td>
<td style="text-align: center;">76.7</td>
<td style="text-align: center;">152</td>
<td style="text-align: center;">2.3</td>
</tr>
<tr class="odd">
<td style="text-align: center;">1980</td>
<td style="text-align: center;">65.5</td>
<td style="text-align: center;">68.9</td>
<td style="text-align: center;">431</td>
<td style="text-align: center;">-3.4</td>
</tr>
<tr class="even">
<td style="text-align: center;">2010</td>
<td style="text-align: center;">57.9</td>
<td style="text-align: center;">62.0</td>
<td style="text-align: center;">1205</td>
<td style="text-align: center;">-4.1</td>
</tr>
<tr class="odd">
<td style="text-align: center;">1990</td>
<td style="text-align: center;">53.3</td>
<td style="text-align: center;">63.0</td>
<td style="text-align: center;">968</td>
<td style="text-align: center;">-9.7</td>
</tr>
<tr class="even">
<td style="text-align: center;">2000</td>
<td style="text-align: center;">50.3</td>
<td style="text-align: center;">60.4</td>
<td style="text-align: center;">1977</td>
<td style="text-align: center;">-10.1</td>
</tr>
</tbody>
</table>

![Critical disconnect by movie decade]({{ site.url }}{{ site.baseurl }}/images/movies-critical-disconnect/disconnect-by-decade.png)

![](movies-critics-audiences-disagree_files/figure-markdown_strict/unnamed-chunk-26-1.png)

The plot above shows the average *critical disconnect* for each decade
of movie release. We see that on average, movies from the 60s and 70s
were rated more favourably by the critics than by audiences (as
indicated by the positive average *critical disconnect*). On the other
hand, movies from the 90s and 00s received higher scores from audiences
than from critics.

# Conclusion

In this post we used data from Rotten Tomatoes to identify which movies
have the highest disagreement between critics and audiences. We did this
by defining a metric we refer to as *critical disconnect*.

The findings suggests that critics do not always understand what will
appeal to audiences. For instance, it looks like the audiences disagreed
with the critics on movies such as *Spy Kids*, *Star Wars: The Last
Jedi*, *Antz*, *Hail Caesar!* and *About a Boy*.

The differences in average critical disconnect among different genres
and movie release decades suggests that there are certain categories of
movies where critics are in some ways more “out of touch”.

However, a lot of these observations may actually be explained by
**selection bias**, as suggested in [this *New York Times* article by
Catherine
Rampell](economix.blogs.nytimes.com/2013/08/14/reviewing-the-movies-audiences-vs-critics/).
Unlike critics, audiences *choose* which movies they will watch and so
are more likely to watch movies that they will probably enjoy.

This selection bias could explain the difference in score distributions.
Critics scores take on a wider range of values because they are paid to
watch all sorts of movies, including ones they wouldn’t choose to watch
otherwise. This is why some critics scores are particularly low. On the
other hand, audiences will generally only watch movies they think they
will enjoy, meaning most scores will be fairly positive. This point
feels particularly relevant for the movies that audiences rated much
higher than the critics - the people watching those movies are probably
already likely to enjoy those kinds of movies. So in some sense it is
more interesting to look at the movies that the critics scored higher
than audiences - these are the movies that audiences chose to watch but
were disappointed with.

# Further questions

As is often the case with data analysis, this leaves us with further
questions such as:

-   What movies do *critics* disagree on?
-   What movies do *audience members* disagree on?
-   Which critic has the lowest “critical disconnect”, or in other
    words, most reflects audiences’ opinions?
