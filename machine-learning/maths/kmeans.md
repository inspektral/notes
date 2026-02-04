---
layout: default
title: K-means
parent: Mathematics
---

# K-means

It is a method to cluster data, which is de-facto quantization, and divides the space into voronoi cells, starting from random displaced centroids moving them around until they're good. It is a discrete latent variable model.

## Algorithm

```python
def kmeans(k, points):
    centroids = init_random_points(k)
    converged = False

    while not converged:
        clusters = [[] for _ in range(k)]

        for point in points:
            cluster = argmin([distance(point, centroid) for centroid in centroids])
            clusters[cluster].append(point)

        new_centroids = [mean(cluster) for cluster in clusters]

        if new_centroids == centroids:
            converged = True
        else:
            centroids = new_centroids

    return centroids, clusters
```
