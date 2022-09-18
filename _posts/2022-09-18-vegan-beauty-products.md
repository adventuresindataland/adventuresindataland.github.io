---
title: "Are cruelty-free and vegan beauty products more expensive?"
classes: wide
tags:
    - python, data analysis
excerpt_separator: <!--more-->
---

Vegan diets are on the rise in many countries, with consumers choosing to change the way they eat due to concerns about the environment, animal rights, or health. However, it's not just food that may contain animal products - veganism can extend beyond food. Beauty products such as makeup and skincare may also contain animal products. Ingredients that may be sourced from animal by-products include [collagen, beeswax, carmine, keratin, guanine, lanolin, and squalene](https://www.superdrug.com/blog/ask-the-experts/guide-cruelty-free-vegan-makeup). 

Rightly or wrongly, veganism can have a reputation for being expensive. In this post I'll investigate whether vegan beauty products are actually more expensive than their non-vegan counterparts. 

<!--more-->

In fact, I'll be looking at products that are both vegan and cruelty-free. This is based on my assumption that the majority of consumers interested in veganism also do not condone animal testing. To learn more about cruelty-free and vegan beauty products, check out the [Cruelty-Free Kitty website](https://www.crueltyfreekitty.com/).

I'll be using the [Sephora dataset from Kaggle](https://www.kaggle.com/raghadalharbi/all-products-available-on-sephora-website) which contains details of over 9,000 products sold at Sephora, a multinational luxury retailer of beauty products with [over 1,000 stores in the US](https://www.scrapehero.com/location-reports/Sephora-USA/). 

The question of interest is:
**Are cruelty-free and vegan beauty products at Sephora more expensive (than their non cruelty-free/non-vegan counterparts)?**

The focus will be on two specific types of beauty products - face serums and mascaras.

# Preparing the data
The available dataset contains product information from Sephora, but this analysis requires additional data. I won't include all of the details here but if you are interested, the full code will be available on GitHub. To summarise:

- 💸 The dataset contains a `price` column but for an accurate price comparison we want to take product size into account. I extracted the product size from the `size` column, and created the new columns `price_per_oz`, `price_per_g`, `price_per_ml` which give the prices per ounce, gram, and millilitre, respectively. This allows us to make a fairer price comparison between products. There were some null values in the product types of interest and I had to manually look up the product size for these.
- 🐇 I created a new column called `cruelty_free` to indicate whether the product is cruelty-free. This was a manual process and involved looking up all of the brands listed in the dataset and checking which ones are cruelty-free according to the [*Cruelty-Free Kitty* website](https://www.crueltyfreekitty.com/). Other sites I found useful were [Ethical Elephant](https://ethicalelephant.com/) and [Logical Harmony](https://logicalharmony.net/).
- 🌱 The column `vegan_inferred` checks whether the word *vegan* occurs in the product description or the brand belongs to a list of manually curated vegan brands. If either criteria is satisfied, the value of this column is True, and null otherwise. So a null means we're not sure whether the product is vegan (a vegan product won't necessarily have the word vegan in the product description).
- 🐇🌱 Finally, we create a column `cruelty_free_and_vegan`, which is the column we'll be using for the analysis. This is True if both the columns `cruelty_free` and `vegan_inferred` are True, False if `cruelty_free` is False, and null otherwise. For the two product categories of interest (face serums and mascaras), I manually go through the products which are null to check whether they are vegan. All remaining products in these categories take the value False.

All in all, there was more manual work involved than I initially expected. I did this iteratively within a Jupyter notebook, then moved it all into a separate Python script called `prepare_data.py`. This script reads in `sephora_website_dataset.csv` and saves the processed version as `sephora_clean.csv`.

Please note: it is possible that some products have been incorrectly labelled in terms of their cruelty-free or vegan status. 


```python
!python prepare_data.py
df = pd.read_csv('sephora_clean.csv')
```

# Are cruelty-free and vegan face serums more expensive?

With the data preparation out of the way, let's look at the first product category - face serums. This is a product category where we might expect ingredients such as collagen and squalene which are often derived from animals.


```python
face_serums['cruelty_free_and_vegan'].value_counts(dropna=False)
```

```output
False    214
True     170
Name: cruelty_free_and_vegan, dtype: int64
```

```python
face_serums['cruelty_free_and_vegan'].value_counts(normalize=True)
```

```output
False    0.56
True     0.44
Name: cruelty_free_and_vegan, dtype: float64
```

We see that 44% of the face serums in our dataset are cruelty free and vegan, showing that there are actually a lot of options in the face serums category.

```python
face_serums['brand'].value_counts().head(10)
```
```output
The Ordinary         32
The INKEY List       16
Dior                 13
Murad                12
Dr. Barbara Sturm    12
Perricone MD         11
Kate Somerville      10
Peter Thomas Roth     9
Origins               9
Estée Lauder          9
Name: brand, dtype: int64
```

In terms of brands, we see that *The Ordinary* sells a whopping 32 different face serums at Sephora! And as it turns out, all 32 of them are cruelty-free and vegan.

## Price - Face serums

Let's look at how the prices for cruelty-free and vegan face serums compare to non cruelty-free/non-vegan face serums. We'll start by looking at some summary statistics and a box plot.

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
  <thead>
    <tr style="text-align: center;">
      <th>cruelty_free_and_vegan</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>True</th>
      <td>170.0</td>
      <td>72.74</td>
      <td>81.60</td>
      <td>3.95</td>
      <td>14.99</td>
      <td>58.0</td>
      <td>84.25</td>
      <td>495.0</td>
    </tr>
    <tr>
      <th>False</th>
      <td>214.0</td>
      <td>90.21</td>
      <td>68.03</td>
      <td>9.00</td>
      <td>49.00</td>
      <td>76.0</td>
      <td>98.00</td>
      <td>460.0</td>
    </tr>
  </tbody>
</table>
</div>

![Price distribution of face serums - bar chart]({{ site.url }}{{ site.baseurl }}/images/vegan-beauty/face-serum-price-distribution-box-plot.png)

If we look at the median price we see that the average cruelty-free vegan face serum costs **\$58** whereas the average non cruelty-free/non-vegan face serum costs **\$76**. The mean price is also lower - **\$73** compared to **\$90** and the lower quartile is *much* lower - only \$14.99 compared to \$49. This shows that:

**Cruelty-free and vegan face serums at Sephora are actually *cheaper* on average than non cruelty-free/non-vegan face serums.**

This is not what I expected to see!

So what are these inexpensive cruelty-free and vegan serums? All of the face serums below the first quartile of \$14.99 are from just two brands - *The Ordinary* and *The INKEY List*. Both of these brands are fairly new to the beauty scene, are cruelty-free, and offer skincare products at low prices. On top of that, the majority of their products are vegan. There were only two products below \$14.99 in the non cruelty-free/non-vegan group - these were both from Sephora's own product line.

If we remove *The Ordinary* and *The INKEY List* from our selection, the results look very different. The good news is that the median price is still lower in the cruelty-free vegan group (\$68 vs \$76). However, the mean is now higher compared to the non cruelty-free/non-vegan group (\$97 vs \$90) and so is the lower quartile (\$55 vs \$49). So in terms of the mean price, it's really the products from *The Ordinary* and *The Inkey List* that are pushing prices down.

**Sephora's wide range of products from low-cost cruelty-free brands *The Ordinary* and *The INKEY List* mean that cruelty-free vegan face serums at Sephora are actually *cheaper* on average than non cruelty-free/non-vegan face serums.**

We can also visualise the price distributions using a violin plot, which provides more detail.

![Price distribution of face serums - violin plot]({{ site.url }}{{ site.baseurl }}/images/vegan-beauty/face-serum-price-distribution-violin-plot.png)

For both groups, the majority of products are within the \$0-150 range. However, the cruelty-free and vegan face serum group actually shows a *bimodal* distribution - this is indicated by two "bumps" in the violin plot, the first one in the \$0-150 range, and the second in the \$200-400 range. There also appear to be more outliers at the higher end of prices. In comparison, the non-cruelty-free/non-vegan group appears to be unimodal.

Inspecting the data shows that the second "bump" in the cruelty-free and vegan group consists of products from the more luxurious brands *Dr. Barbara Sturm*, *Perricone MD*, and *Tata Harper*. For instance, there is a \$300 hyaluronic acid serum from *Dr. Barbara Sturm* even though *The Ordinary* offers a hyaluronic acid serum for \$7. For those interested in understanding the differences between these products, there's actually [an article about this on *The Huffington Post*](https://www.huffingtonpost.co.uk/entry/hyaluronic-acid-difference_l_5e138fdde4b0b2520d260b53).

The most expensive face serum in the cruelty-free and vegan group is the *Neuropeptide Smoothing Facial Conformer* by *Perricone MD*, costing \$495. For the non cruelty-free/non-vegan group, it's the *Orchidée Impériale The Cream* by *Guerlain* at \$460. 

While cruelty-free and vegan face serums cost less on average, products in this group can still set you back a pretty penny!

## Price per ounce - Face serums

But what if the vegan face serums just contain less product? We can compare the two groups by looking at the price per ounce (`price_per_oz`) rather than product price. This should give us a fairer comparison.

There is a handful of products that are still missing the `size` column - these are products that are non-standard and won't be included here.

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
  <thead>
    <tr style="text-align: center;">
      <th>cruelty_free_and_vegan</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>True</th>
      <td>168.0</td>
      <td>70.82</td>
      <td>76.36</td>
      <td>2.28</td>
      <td>14.99</td>
      <td>58.00</td>
      <td>78.82</td>
      <td>450.0</td>
    </tr>
    <tr>
      <th>False</th>
      <td>209.0</td>
      <td>82.46</td>
      <td>67.43</td>
      <td>5.13</td>
      <td>45.64</td>
      <td>68.24</td>
      <td>98.00</td>
      <td>560.0</td>
    </tr>
  </tbody>
</table>
</div>

![Price distribution of face serums per ounce - violin plot]({{ site.url }}{{ site.baseurl }}/images/vegan-beauty/face-serum-price-per-ounce-distribution-violin-plot.png)

Again, cruelty-free vegan face serums are cheaper, although the gap has closed somewhat. In terms of the median, the average cruelty-free vegan face serum costs **\$58** per ounce whereas the average non cruelty-free/non-vegan face serum costs **\$68** per ounce.

If we exclude products from *The Ordinary* and *The INKEY List*, the difference in median is trivial (\$67 vs \$68) and as before, the mean and lower quartile prices are actually higher (\$95 vs \$82 and \$54 vs \$46, respectively). This highlights again that it's the products from *The Ordinary* and *The INKEY List* that are keeping the average price down in the cruelty-free and vegan group.

In terms of price per ounce, the most expensive face serum is the *Elixir Vitae Serum Wrinkle Solution* by *Tata Harper* (\$450 per ounce) in the cruelty-free and vegan group and the *TIME RESPONSE Skin Reserve Serum* by *AMOREPACIFIC* (\$560 per ounce) in the non cruelty-free/non-vegan group.

## Summary - Face serums

Cruelty-free and vegan face serums at Sephora are actually cheaper on average than non cruelty-free/non-vegan face serums. This is thanks to a wide product offering from *The Ordinary* and *The INKEY List*, which offer cruelty-free and vegan products at low prices. 

Of course, this analysis assumes that all face serums are created equal. But in reality, face serums may be created to target specific skin concerns. Furthermore, some products may be of higher quality than others, or better suited to certain skin types.

However, it's worth considering why certain items or brands are more expensive than others. [In skincare, things like marketing and packaging play a large role in determining the price of a product](https://www.byrdie.com/difference-between-expensive-and-affordable-skincare-4800708).

# Are cruelty-free vegan mascaras more expensive?

Next, let's look at mascaras - this is a product category where we might expect beeswax, a non-vegan ingredient. I'll exclude mini versions and travel sizes from consideration. I'll also exclude lash serums as these products are different to mascaras and we would expect them to be more expensive.

## Price - Mascaras

```python
mascaras['cruelty_free_and_vegan'].value_counts(dropna=False)
```

```output
False    113
True      25
Name: cruelty_free_and_vegan, dtype: int64
```

```python
mascaras['cruelty_free_and_vegan'].value_counts(normalize=True,dropna=False)
```

```output
False    0.82
True     0.18
Name: cruelty_free_and_vegan, dtype: float64
```

Only 18% of mascaras in our dataset are cruelty free and vegan, so for this product category our options are more limited. 

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
  <thead>
    <tr style="text-align: center;">
      <th>cruelty_free_and_vegan</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>True</th>
      <td>25.0</td>
      <td>23.36</td>
      <td>2.83</td>
      <td>12.0</td>
      <td>23.0</td>
      <td>24.0</td>
      <td>24.0</td>
      <td>29.0</td>
    </tr>
    <tr>
      <th>False</th>
      <td>113.0</td>
      <td>26.94</td>
      <td>7.67</td>
      <td>10.0</td>
      <td>24.0</td>
      <td>27.0</td>
      <td>29.5</td>
      <td>70.0</td>
    </tr>
  </tbody>
</table>
</div>

![Price distribution of mascaras]({{ site.url }}{{ site.baseurl }}/images/vegan-beauty/mascaras-price-violin-plot.png)

The median price for a mascara is \$24 in the cruelty-free and vegan category and \$27.25 in the non cruelty-free/non-vegan category, so cruelty-free and vegan mascaras are cheaper on average. However, the relative difference in price between the two groups is not as big as it was for face serums. Perhaps the main difference is that the non cruelty-free/non-vegan group contains some much more expensive mascaras, with the maximum price being \$70.

Nine of the twelve cruelty-free and vegan mascaras priced \$23 or below are from the brand *tarte*. The most expensive mascara in the cruelty-free and vegan category is the *Caution Extreme Lash Mascara* from *Hourglass* at \$29, but in the non-cruelty-free/non-vegan category the maximum is much higher at \$70. The most expensive mascara in the latter group is the *Lash Amplifying Lacquery* from *Christian Louboutin* (\$70).

## Price per ounce - Mascaras

Again, let's look at price per ounce to ensure that the analysis is controlled for product size.

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
  <thead>
    <tr style="text-align: center;">
      <th>cruelty_free_and_vegan</th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>True</th>
      <td>24.0</td>
      <td>81.74</td>
      <td>19.59</td>
      <td>57.14</td>
      <td>72.19</td>
      <td>75.83</td>
      <td>92.00</td>
      <td>152.94</td>
    </tr>
    <tr>
      <th>False</th>
      <td>111.0</td>
      <td>103.83</td>
      <td>42.24</td>
      <td>26.67</td>
      <td>77.88</td>
      <td>97.17</td>
      <td>120.26</td>
      <td>304.35</td>
    </tr>
  </tbody>
</table>
</div>

![Price distribution of mascaras per ounce]({{ site.url }}{{ site.baseurl }}/images/vegan-beauty/mascaras-price-per-ounce-violin-plot.png)

Looking at price per ounce, we still see that cruelty-free and vegan mascaras are cheaper on average than non cruelty-free/non-vegan mascaras, both in terms of mean and median.

In terms of price per ounce, the cheapest cruelty-free and vegan mascaras at Sephora are actually from *FENTY BEAUTY*, *Too Faced*, *Smashbox*, *Pretty Vulgar*, and *MILK MAKEUP*. While *tarte* offers a lot of lower-priced mascaras, unfortunately these actually have a smaller product size, making them some of the most expensive mascaras in the cruelty-free and vegan group. Note that here we assume that the same amount of product is used when applying each mascara, however this is an oversimplification.

If you are just looking for a bargain, there are nine mascaras in the non cruelty-free/non-vegan group that are cheaper than the cheapest cruelty-free/vegan mascara - six of these are from Sephora's own brand, *SEPHORA COLLECTION*.

## Summary - Mascaras

Cruelty-free and vegan mascaras at Sephora are cheaper on average than non cruelty-free/non-vegan mascaras, even if we take product size into account. However, there is limited choice - only 18% of all mascaras are cruelty-free and vegan.

# Conclusion

**In this post, we showed that for two product categories (face serums and mascaras) at Sephora, the cruelty-free and vegan options are actually cheaper on average. This is the case both in terms of the mean and median price. This also holds if we take the product size into account and compare the price per ounce instead.**

This means that choosing cruelty-free and vegan options doesn't have to be more expensive!

44% of face serums considered were cruelty-free and vegan, so there is a lot of choice in this product category. On the other hand, only 18% of mascaras considered were cruelty-free and vegan, so here the options are much more limited. 

The choice of brands and products that a retailer stocks has a big impact on the results. For face serums, the lower average price can be attributed to a wide product offering from two brands in particular - *The Ordinary* and *The INKEY List*. 

The analysis was focussed on the luxury beauty retailer *Sephora* so these results may not hold for other stores, particularly because they depend a lot on what brands are stocked. It would be interesting to see what the situation looks like in drugstores where products are generally more accessible in terms of price - are there still a lot of cruelty-free and vegan options?

While preparing the dataset for this project, I noticed that a lot of the sites that sell beauty products don't make it very clear which products are cruelty-free and vegan. Thankfully, nowadays there are websites such as [Cruelty-Free Kitty](https://www.crueltyfreekitty.com/) that can help consumers make more informed decisions. This analysis shows that choosing a more ethical option doesn't have to break the bank.