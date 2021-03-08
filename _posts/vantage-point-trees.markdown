---
layout: default
title:  "Vantage Point Trees"
date:   2020-03-21 19:26:29 +0800
categories: datastructures trees
---

# What is a Vantage Point Tree ?

A Vantage Point Tree (or VP-tree for short) is a special kind binary tree that can be used to store spatial objects. In this tree, a non-leaf node has a special point called (you guessed it) the vantage point, that is used to partition space in two. Spatial objects closer to the node's vantage point than to a certain threshold will end up in one of its two children, while all other objects will end up on the other children. In 2D, this effectively partition space using circles.

**TODO : insert image**

While VP-trees can be used in several settings, they particularly shine when performing a *k nearest-neighbors search*. 


# Coding a VP-tree : an example in Python

## Pre-requisites

* numpy

## VP-tree basics

```python
def __init__(self, depth: int, points: List[np.array]):
    if depth == 0:
		self.points = points
		return
	
	self.vpoint = random.choice(points)
	dists = [numpy.linalg.norm(p - self.vpoint) for p in points]
    self.threshold = median(dists) #from statistics import median
	self.lchild = VPTree(
        depth - 1,
		[p for p, d in zip(points, dists) if d <= self.threshold],
	)
	self.rchild = VPTree(
	    depth - 1,
		[p for p, d in zip(points, dists) if d > self.threshold],
	)
```

```python
def is_leaf(self):
    return self.vpoint is None
```

## K Nearest-neighbor Search

```python
class KNNSearchResult:
    def __init__(self, point: np.array, dist: float):
	    self.point, self.dist = point, dist
```

The goal of our search is, given $$ k $$ and an input point $$ p $$, to find the $$ k $$ closest points to $$ p $$ in a specific set of points.

```python
def knn_search(self, center: np.array, k: int) -> List[KNNSearchResult]:
    if self.is_leaf():
	    return sorted([KNNSearchResult(p, np.linalg.norm(p - center)) for p in self.points])[:k]
		return
		
	# recursion
	dist_to_vpoint = np.linalg.norm(self.vpoint - center)
	
	# we are inside the circle : search in left tree
	if dist_to_vpoint <= threshold:
		inside_results = self.lchild.knn_search(center, k)
		
		# points left to search
		dist_to_outer_edge = threshold - dist_to_vpoint
		left_to_search_nb = len(
			[_ for result in inside_results if result.dist > dist_to_outer_edge]
		)
		if left_to_search_nb == 0:
			return inside_results
		outside_results = self.rchild.knn_search(center, left_to_search_nb)
		# merge results
		return sorted(inside_results + outside_results)[:k]

	# we are outside the circle : search in right tree
	else:
		outside_results = self.rchild.knn_search(center, k)
		
		# pointesleft to searcg
		dist_to_outer_edge = dist_to_vpoint - threshold
		left_to_search_nb = len(
			[_ for result in outside_results if result.dist > dist_to_outer_edge]
		)
		if left_to_search_nb == 0:
			return outside_results
		inside_results = self.lchild.knn_search(center, left_to_search_nb)
		return sorted(outside_results + inside_results)[:k]
```
