---
title: "JavaFX Algorithm Visualizer"
date: 2025-10-01
media_subpath: /assets/posts/2025-10-01-javafx-algorithm-visualiser
published: true
layout: post
categories:
- software development
- algorithm visualisation
tags:
- java
- javafx
- algorithms
- dijkstra
- graph theory
- visualisation
---


## Repository

For running this project and documentation, visit the GitHub repo: [javafx-algorithm-visualiser](https://github.com/jbrowne7/javafx-algorithm-visualiser){:target="_blank"}

## Demo Video

{% include embed/youtube.html id='5Gv4n_MWxY8' %}

*If the embedded video doesn't load, you can watch the demo directly on [YouTube](https://www.youtube.com/watch?v=5Gv4n_MWxY8){:target="_blank"}*

## Overview

An interactive graph algorithm visualiser to help users better understand how algorithms work. At the time of writing this the repo currently only supports Dijkstras algorithm but I plan on extending this to other graph based algorithms.

## What I Built

I developed an interactive algorithm visualisation tool mainly for graph based algorithms which involves a visual representation of nodes and weighted directed edges. The application also has a step button to allow you to step throgh and algorithm and see what happens at each stage. There is also a table showing visited and distance to different nodes which is updated in real-time. The App also uses a modular architecture with clear seperation between algorithm logic, data models, UI, and utility functions making it easier to build on top of this code base.

## Technologies Used

- **Java+**: Core programming language with modern language features
- **JavaFX**: Package for building GUIs
- **Gradle**: Build automation and dependency management
- **Object-Oriented Design**: Clean architecture with interfaces, inheritance, and encapsulation
- **Data Structures**: Custom implementations of graphs, priority queues, and algorithm state management

## Key Features

- **Algorithm Visualisation**: Currently only supports Dijkstra's shortest path algorithm 
- **Interactive Controls**: Step button to manually step through algorithm iterations
- **Visual Feedback**: Color-coded nodes based on which node we are visiting and processing
- **Extensible Design**: Algorithm interface allows easy addition of new graph-based algorithms

## What I Learned

- **Algorithm Implementation**: Deepened understanding of graph algorithms by implementing them from scratch
- **JavaFX**: Improved skills with building Java GUIs, UI controls, event handling, and custom data structure implementation
- **Software Architecture**: Practiced separation of concerns with distinct packages for algorithms, models, UI, and utilities
- **State Management**: Learned to translate algorithm states into visual representations effectively
- **Cross-Platform Development**: Implemented build systems that work consistently across different operating systems
- **Problem Solving**: Overcame challenges in geometry for edge rendering and arrow head positioning (this was probably one of the most confusing parts of the project)
