---
layout: post
author: Grant Nichol
color:  green
title:  "OhHi Solver"
date:   2020-04-29 20:50:00 -0600
categories: ["Game", "Projects"]
---

I created a Python program that can solve puzzles in the game [OhHi](https://play.google.com/store/apps/details?id=com.q42.ohhi) by Q42. My code uses a hybrid solving technique where it loops through all possible moves to find those that are obvious, then it performs a depth first recursive search.

## Rules of OhHi

 * No three tiles of the same color can be in a line
 * All rows and columns have the same number of each color
 * No two rows or columns can be the same

## Components

 * A class for solving the board
 * A wxPython GUI for playing with and testing the solver.
 * A program to connect to an android device through ADB to directly play the game on device.

## GitHub

The repo for this project is in [gwnichol/OhHiSolver](https://www.github.com/gwnichol/OhHiSolver)
