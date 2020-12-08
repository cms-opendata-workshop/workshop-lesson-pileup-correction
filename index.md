---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---
{% comment %} ![](assets/img/abcd_letters.png) {% endcomment %}

> ## Prerequisites
> In order to complete this lesson you need
> - [ROOT CERN](https://root.cern/) 6.16 or later installed. You can set up ROOT as recommended [here](https://cms-opendata-workshop.github.io/workshop-lesson-root/02-get-root/index.html).
> - The [Higgs to tau tau analysis example code](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis) installed and running.
> A stable internet connection to access the input data.
{: .prereq}

> ## Helpline
> Instructions written by **Anniina Kinnunen** (University of Helsinki) **Santeri Laurila** (CERN). All example code we use is written by **Anniina Kinnunen** and **Stefan Wunsch** (CERN). For questions, please contact Santeri Laurila (firstname.lastname@cern.ch).
{: .callout}

Welcome! This lesson will familiarize you with the concept of pileup and explain how to derive and use so-called pileup corrections to ensure correct pileup modeling in your analysis. We will demonstrate this by applying pileup corrections in the Higgs to tau tau analysis example.

<!-- this is an html comment -->

This is a comment in Liquid 

{% include links.md %}

> ## Input data 
> The data used in this lesson are in pre-processed "NanoAOD" datasets. The method to access them will be explained during the exercise itself.
{: .callout}
