---
title: "Editing Colours in Images with KMeans Clustering"
date: 2022-09-05
author: jordanjoewatson
---

## Introduction

The code for this is on my github at https://github.com/jordanjoewatson/kmeans-colour-converter.

The script uses KMeans clustering to modify the colour scheme of images as shown below.

![Pepsi original](https://github.com/jordanjoewatson/jordanjoewatson.github.io/blob/main/images/kmeans_image_converter/pepsi.png)
![Pepsi modified](https://raw.githubusercontent.com/jordanjoewatson.github.io/blob/main/images/kmeans_image_converter/output11.png)

## Methodology

My initial idea behind this was that RGB values (or colours), when considered to be in a 3 dimensional space would cluster together with other similar colours. For example, if you had an image that had 2 different shades of blue and 2 different shades of red and used KMeans clustering with 2 clusters, the result would be 1 blue cluster and 1 red cluster.

Therefore, if you have an image which has `N` main colours, using KMeans with `N` clusters would result in each cluster representing one of the main colours of the image. In the example provided there are four main colours:
- Dark blue
- Light blue
- White
- Red

After running KMeans with 4 clusters, each of the cluster will now represent those colours, but we are unable without further analysis to determine which cluster represents which colour.

```
NEW_COLOURS = [
        (255,255,255),
        (159,119,224),
        (32,50,85),
        (216,201,241)
]

img = Image.open('myimage.png')

# read in all pixel values
pixel_values = []
for h in range(0, img.height):
    for w in range(0, img.width):
        pxl = img.getpixel((w,h))
        pixel_values.append((pxl))

km = KMeans(n_clusters=N, init='k-means++', random_state=42)
km.fit(pixel_values)

y = km.fit_predict(pixel_values)
```

`y` now contains the predicted values `0,1,2,3` representing each possible cluster. We can then create all possible permutations of mappings based on the amount of colours provided.

```
mappings = []
for (a,b,c,d) in itertools.permutations(range(0,len(COLORS)), len(COLORS)):
    mappings.append({ 0:a, 1:b, 2:c, 3:d })
```

This means that we now have a mapping from each of those four colours (the key in the mapping dictionaries) to a cluster value (the key in the mapping dictionaries). The `mappings` list now contains a list of dictionaries which defines this mapping, from the `Nth` cluster to the `Mth` new colour. This is implemented by using integers to refer to each KMeans cluster to each new possible colour by creating all permutations of `N` values. 

The KMeans algorithm and fitting the data gives us the following mapping:

$\text{initialColour}\rightarrow\text{Cluster}$

The `itertools.permutations` code above gives us the following mapping:

$\text{Cluster}\rightarrow\text{newColour}$

Together, of course these result in the following mapping:

$\text{initialColour}\rightarrow\text{Cluster}\rightarrow\text{newColour}$

We can then iterate over all possible mappings, for each mapping. It is important to note for this we require the following values:
- An iterator to index into `y`, our results list from KMeans.
- Iterators to iterate over the pixels in the image `w` and `h`.
- Each mapping dictionary.

Therefore for each `mapping` we do the following:
```
for h in range(0, img.height):
    for w in range(0, img.width):

        group = mapping[y[itr]]
        rgb = NEW_COLOURS[group]

        img.putpixel((w,h), rgb)

        itr += 1
```

For each value in the `y` results list which contains each pixels `Nth` cluster, we get the `ith` result with `y[itr]`. This value will be one of `0,1,2,3` as we are using KMeans with 4 clusters.

We then use this result as an index into our mapping list `mapping[X]` to get the new colour, using the cluster to new colour mapping. This is possible as the mapping dictionary contains one of multiple mappings created earlier in the `itertools.permutations` function.

After this, we can use our new value `mapping[y[itr]]` to get the new RGB value from the `NEW_COLOURS` list we provided at the start. As we're now iterating over each of the pixels using the `w` and `h`, we can use these indeces to write our new colour into those pixel values.

The following breakdown may make it easier to understand what is happening.
1. Create list of predictions called `y`, containing the pixels predicted Cluster value
2. Get a specific mapping (dictionary) of integers, representing `Nth` cluster (keys) to new colour (values)
3. For each pixel, find what it's predicted cluster was calculated as from `y`, use that predicted cluster to find a new colour based on the mapping with `mapping[pixelPrediction]`, this gives a new index value `group`.
4. Using `group`, index into the possible new colours and write that new colour into the pixel value.

