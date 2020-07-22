# Brand Coolness Using Sppotify and Song Lyrics
[Full Code](https://github.com/sourwurm/Brand-Coolness-with-Spotify)

I remember being a kid and thinking it was pretty funny that the Supreme Court could refer to corporations as "persons". I recently had that thought again and it made me think, if a corporation can be a person, then why can't a person be a corporation? Needless to say I have now relinquished my humanity.

#### Dav Corp.
So here I am, David Lopez, the corporation. In order to see how Dav Corp. is doing, I'd like to understand how "cool" our brand is to the general public. "Brand Coolness" has been conceptualized many different ways. Though seemingly juvenile, the implications that coolness can have on a brand can be the difference between [Nike](https://www.businessinsider.com/how-nike-defines-cool-2017-3) and [Sketchers](https://www.reddit.com/r/AskReddit/comments/52mduq/whats_the_most_uncool_brand_or_company/d7lf1t1?utm_source=share&utm_medium=web2x). In an article published by [Warren (2019)](http://www.sundigital.uk/Journals-ABS-4star/Brand%20Coolness%20-%20Caleb%20Warren,%20Rajeev%20Batra,%20Sandra%20Maria%20Correia%20Loureiro,%20and%20Richard%20P.%20Bagozzi.pdf) brand coolness was conceptualized to have 10 sepearate criteria"

A cool brand is:
- Extraordinary/Useful                - High Status
- Aesthetically Appealing             - Rebellious
- Original                            - Authentic
- Subcultural                         - Popular
- Iconic                              - Energetic


Surely Dav Corp. could fit the bill. To see how we measure up, we chose to evaluate ourselves based on data provided by Spotify, and lyric data that could be found on Genius.com. Music can tell a lot about an individual (in this case: a corporation). In many ways, music is a reflection of our personalities, so what better way to measure our perceived coolness than with Spotify listening trends? Here at Dav Corp., we always strive for success, so we can't just be cool. We must be COOL-ER than the average spotify user. For this reason, I'll be going head-to-head with the top tracks on Spotify to see who comes out on top for coolness.


#### The Data
Spotify's API documentation is amazingly helpful. After getting my token, I was able to write a reusable function that requests and cleans Spotify playlist data. Conveniently, Spotify releases a yearly playlist documenting the users Top tracks for the year. They also release playlists that hold the top 100 songs on Spotify overall. For this analysis, I decided to take advantage of this, and got ahold of my Top tracks for the years 2016 - 2019, and Spotify's Top tracks for 2016 - 2019 as well. For each of these tracks, I would get valuable audio feature data such as danceability, energy, and popularity as well. The finalized datasets look like this:

[](/images/bc-dataframe.png)

Now, I could get started on answering the question: how cool is my brand?

#### Popular
Warren (2019) identified popularity as being trendy and liked by most people. To measure this, I looked at the data within the popularity and release data columns. Music that is popular contains a float value closer to 1, where as release date is simply a date. These features will help measure if my music is trendy, and generally well-liked by others as well. Visualization showed that the difference in popularity was clear.
[](/images/bc-pop_dist.png)
My Tracks peak at the 0 popularity mark which explains why I have trouble finding people who listen to the same music as me! On the bright side, My Tracks seem to cover a good spread of popularity ranges, having decent representation all the way up to the 60 mark on the popularity axis.

Of course it should be expected that the Spotify Top 100 should be comprised of songs with high popularity. Interestingly enough, we can observe a bimodal distribution for spotifys top track popularity through nodes peaking at both high and low popularity. This is because Spotify calculates popularity based off current spotify listening activity. Meaning, tracks may have had a high popularity score in their heyday, but have since fallen off. This exemplified by looking at exactly which popular songs fall under this category.

[](/images/bc-low_pop_top.png)
[](/images/bc-rel_mode.png)

On top of this, it seems like my mode release data for low-popularity tracks is 2 years older than the top tracks on spotify. So that was straight-forward, but with these findings, it seems like my brand is not popular. But low popularity could be indicative of other qualities, such as being part of a subculture (another measure of brand coolness).

#### Subcultural
Proving this one will require a bit more, so to get a rough idea of differences between several features at once, I chose to plot everything on a radar chart using the means of each audio feature.

[](/images/bc-radar.png)

As we saw before, the largest difference seems to be within popularity, followed by instrumentalness and danceability. While the names are somewhat intuitive, instrumentalness reflects how instrumental a song is (less instruments -> lower instrumental score). Danceability is how danceable a song is. This metric is based off tempo, time signature, and other factors. Differences in these specific columns can indicate larger differences that exist within the music libraries at large. Now we can look at the differences in danceability and instrumentalness distributions.

[](/images/bc-danceability_dist.png)

The two distributions are strikingly different and it seems like my music seems to encompass a wider variety of music. Most notably, top tracks seem not to be well represented under the low danceability score region. This fact is supporting evidence that my music could potentially be more subcultural.

Now to check instrumentalness.

[](/images/bc-instrumentalness_dist.png)

To clarify, Spotify specifies that instrumental scores over 0.5 denote songs with very little (if any) vocal. It seems like top tracks never go over an instrumental rating of ~0.3. While my music also has a prominent peak near 0, I also encompass a much larger spectrum of instrumentalness. Coupling these findings with the popualirty verdict, it seems like music has subcultural tendencies but to be sure, more is needed. We can use K-Means clustering to identify underlying clusters within the data, and how my tracks may be divided up.

Since I'd like to include as many features as possible, it would benefit me to use Principal Component Analysis to reduce dimensionality. First we must identify the optimal number of PC's by measuring explained variance by the number of PC's used.

[](bc-exp_var.png)

Generally speaking, 0.8 explained variance is a good marker to shoot for. Meaning I would choose six Principle Components, but after some tinkering I found that not much information was being stored within the last two PC's. For that reason, I will only be using four PC's, which cover nearly 65% of explained variance within the data. To get a better idea of what's being accounted for, we can load at the loadings for each component.
[](/images/pca_load.png)
From these four PC's , we can see that primarily variance in duration (in seconds), tempo, and popularity is being accounted for. Tempo and duration also happen to be key factors in calculating danceability, so this is alright. Interestingly enough, PC2 and PC3 seem to hold the information on the same features, but inverted. This will be important for the K-Means model.

Next we can test to see how many clusters are optimal for the K-Means model using the elbow method as well as sihouette score.
[](/images/bc-elbow_meth.png)
The elbow here occurs at 4 clusters.

[](/images/bc-silhouette.png)
This plot is in agreement with the previous plot, but also suggests three clusters might be a possibility. After some tinkering, it turns out that 4 is the optimal number of clusters for K-Means. The clusters looked like this:

[](/images/bc-kmeans.png)

Here we see four well-represented clusters using only 2 of the 4 PC's. While this is already suggestive of subcultures, it's important to recall that PC2 and PC3 were basically inverses of each other. With that in mind, it would be helpful to plot this in 3-Dimensions to view a clearer picture of what's going on!

[](/images/bc-3d_kmeans.png)

This plot shows us that the two central clusters are acturally layered on each other. As these clusters have shown, there are several different groups existing within my date based off tempo, duration, and popularity alone. With these findings it is safe to say that my music is sufficiently different from the top tracks on Spotify and is indeed subcultural.

#### Energetic
