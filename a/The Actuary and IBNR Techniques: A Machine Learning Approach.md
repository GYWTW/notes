# The Actuary and IBNR Techniques: A Machine Learning Approach

source: https://www.actuaries.org.uk/system/files/field/document/2.%20Ronald%20Richman_Caesar%20Balona_The%20Actuary%20and%20IBNR%20Techniques%20A%20Machine%20Learning%20Approach.pdf

Caesar Balona∗ Ronald Richman†

14 August 2020

## Abstract

Actuarial reserving techniques have evolved from the application
of algorithms, like the chain-ladder method, to stochastic models of
claims development, and, more recently, have been enhanced by the
application of machine learning techniques. Despite this proliferation
of theory and techniques, there is relatively little guidance on which
reserving techniques should be applied and when. In this paper, we
revisit traditional reserving techniques within the framework of supervised learning to select optimal reserving models. We show that the
use of optimal techniques can lead to more accurate reserves and investigate the circumstances under which different scoring metrics should
be used.

## 1 Introduction

Since Bornhuetter and Ferguson (1972) encouraged actuaries to pay attention to the topic of Incurred But Not Reported (IBNR) reserves, a vast literature on loss reserving techniques has emerged, see, for example Schmidt
(2017) and Wuthrich and Merz (2012). Several main strands of this literature can be identified: improved reserving techniques for calculating best
estimates, reformulating these techniques statistically to calculate measures
of uncertainty, and, most recently, the application of machine learning techniques to the problem of IBNR reserving.

Many new IBNR reserving techniques have been developed since the development of the Chain Ladder (CL) and Bornhuetter-Ferguson (BF) techniques, with notable examples that are often used in common practice being
the Cape Cod (CC) method from Buhlmann and Straub (1983) and its
generalization due to Gluck (1997), and the Incremental Loss Ratio (ILR)
