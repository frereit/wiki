---
title: Heapsort Visualization
description: A visualisation of Heap Sort created with 3b1b's Manim
published: true
date: 2022-06-22T20:28:58.196Z
tags: manim, algorithms
editor: markdown
dateCreated: 2022-06-22T20:28:02.070Z
---

# Heapsort

This is my first time using manim, and I mostly used it to experiment with the framework, but here's my visualisation of Heapsort, created with manim:

<iframe width="560" height="315" src="https://www.youtube.com/embed/dQw4w9WgXcQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```python
%%manim -qh HeapSort
from random import shuffle
from manim import *

config.media_width = "75%"
config.verbosity = "WARNING"

class HeapSort(Scene):
    def sift_down(self, verts, start, end, g):
        self.play(Indicate(g[verts[start]]))
        root = start
        while (root*2 + 1) <= end:
            child = root*2 + 1
            swap = root
            if verts[swap] < verts[child]:
                swap = child
            if child+1 <= end and verts[swap] < verts[child+1]:
                swap = child + 1
            if swap == root:
                return
            else:
                self.play(Swap(g[verts[root]], g[verts[swap]]))
                verts[root], verts[swap] = verts[swap], verts[root]
                root = swap
    
    def heapify(self, verts, count, g):
        start = count//2 - 1
        while start >= 0:
            self.sift_down(verts, start, count - 1, g)
            self.wait(0.5)
            start -= 1
    
    def get_edges(self, verts):
        edges = []
        for i in range(len(verts)//2):
            if (2*i) + 2 < len(verts):
                edges.append((verts[i], verts[2*i+2]))
            edges.append((verts[i], verts[2*i+1]))
        return edges

    def construct(self):
        verts = list(range(1,8))
        shuffle(verts)
        g = Graph(verts, [(verts[i], verts[i+1]) for i in range(len(verts)-1)], layout="tree", layout_scale=3, 
                  labels=True, root_vertex=verts[0])
        self.play(Create(g))
        self.wait(1)
        self.play(g.animate.remove_edges(*[(verts[i], verts[i+1]) for i in range(len(verts)-1)]),
                  g.animate.add_edges(*self.get_edges(verts)),
                  g.animate.change_layout("tree", root_vertex=verts[0]))
        self.play(g.animate.scale(2))
        self.wait(2)
        self.heapify(verts, len(verts), g)
        self.wait(1)
        end = len(verts) - 1
        while end > 0:
            self.play(Swap(g[verts[end]], g[verts[0]]))
            verts[end], verts[0] = verts[0], verts[end]
            self.play(g[verts[end]].animate.set_opacity(0.6))
            end -= 1
            self.sift_down(verts, 0, end, g)
            self.wait(0.5)
        opacity_back = []
        for vert in verts:
            opacity_back.append(g[vert].animate.set_opacity(1))
        self.play(*opacity_back)
        self.play(
            g.animate.remove_edges(*list(g.edges.keys())),
            g.animate.add_edges(*((vert, vert+1) for vert in verts[:-1]))
        )
        self.wait(1)
        self.play(g.animate.scale(1/2))
        self.play(g.animate.change_layout("tree", root_vertex=verts[0]))
        self.wait(2)
```