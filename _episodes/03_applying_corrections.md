---
title: "What is pileup and how to model it correctly?"
teaching: 10
exercises: 20
questions:
- "How to apply the pileup corrections in a specific analysis?"
objectives:
- "TODO"
keypoints:
- "TODO"
---

**ALL TEXT BELOW IS JUST COPY-PASTE FROM ABCD LESSON; TO BE UPDATED**

## Signal and control regions

By **signal region**, we mean the region in the phase space defined by our **signal selection**, i.e. the trigger and all offline selections that we use in the analysis. 

In addition to signal region, often we need one or several **control regions**. These are usually obtained by changing some of the cuts w.r.t. our signal selection, to define regions that are in some aspects **similar to signal region**, but they are **signal-depleted**, i.e. the signal-to-background ratio is very tiny or even zero. Typically we want to define control regions that are **enriched in a particular background process**, and have **sufficient statistics**, i.e. there is enough events that enter the control region to give us sufficient statistical precision.

Sometimes control regions are also referred to as **sidebands**, especially in cases where the signal shows up as a resonance peak, so the signal region is defined by selecting some mass window, and the control regions are defined as sidebands on the left and right side of the mass window.

## Signal and control regions in the ABCD method

In the ABCD method, region **D** is our signal region, whereas regions **A**, **B** and **C** are all control regions. 

In the ABCD method, the  **region C is used to estimate the shape of the background process**, as a function of one or several variables. 
Therefore we should aim to select a region where we can safely assume the background process to take similar shape as in the signal region D.

Next let us see what this means in the context of the Higgs to tau tau analysis.

## Definition of control region C in the Higgs to tau tau analysis example

In the [histograms.py script](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/histograms.py#L120com) you can find the following lines:
~~~
        # Book histograms for the signal region
        df1 = df.Filter("q_1*q_2<0", "Require opposite charge for signal region")
        df1 = filterGenMatch(df1, label)
        hists = {}
        for variable in variables:
            hists[variable] = bookHistogram(df1, variable, ranges[variable])
        report1 = df1.Report()

        # Book histograms for the control region used to estimate the QCD contribution
        df2 = df.Filter("q_1*q_2>0", "Control region for QCD estimation")
        df2 = filterGenMatch(df2, label)
        hists_cr = {}
        for variable in variables:
            hists_cr[variable] = bookHistogram(df2, variable, ranges[variable])
        report2 = df2.Report()
~~~
{: .python}

As you can see, here **define the signal regions by requiring that hadronic tau and the muon have opposite signs*** (as they should if they are produced in a decay of a neutral Higgs boson), i.e. `q_1*q_2<0`, while the **control region is defined by requiring them to have the same sign**, i.e. `q_1*q_2>0`.

> ## Challenge
> Task: run the Higgs to tau tau analysis up to the step where you produce the histograms (python histograms.py) according to [the instructions on this page](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis). 
> 
> Then inspect the histograms with ROOT TBrowser. Look at the histograms for the largest signal process (ggH), and compare the histograms showing the signal region (no postfix in the histogram name) and those showing the control region (`_cr` postfix in the histogram name). Which region has more signal? 
>
> The scroll through the selection of histograms to see all the different processes contained in this root file.
{: .challenge}

## Estimating the QCD background in control region C 

Often our control **region C is not completely pure**, so that it would contain only events produced by the background process we want to estimate. Instead, our data sample is a mixture of different processes. This is also the case in the Higgs to tau tau analysis example.

In order to estimate the yield and the shape of the QCD multijet background, we need to **estimate all other processes that enter the control region, and subtract their contribution from the data**. Luckily, we know how to estimate all the other relevant background processes -- from simulation! 

The subtraction of simulated processes (which are normalized to the integrated luminosity and the cross section), is done in the potting sxcript [plot.py](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/plot.py#L155), where you can find the following lines:
~~~
    # Data-driven QCD estimation
    QCD = getHistogram(tfile, "dataRunB", variable, "_cr")
    QCDRunC = getHistogram(tfile, "dataRunC", variable, "_cr")
    QCD.Add(QCDRunC)
    for name in ["W1J", "W2J", "W3J", "TT", "ZLL", "ZTT"]:
        ss = getHistogram(tfile, name, variable, "_cr")
        QCD.Add(ss, -1.0)
    for i in range(1, QCD.GetNbinsX() + 1):
        if QCD.GetBinContent(i) < 0.0:
            QCD.SetBinContent(i, 0.0)
~~~
{: .python}

The point here is that this way, with the ABCD method, we can estimate the process that is difficult to simulate (QCD) by using data, as well as simulated samples for the background processes for which the simulations are relatively reliable (here: W+Jets, ttbar, Drell-Yan). 

Soon we get to run this script and inspect the resulting QCD background estimate.

{% include links.md %}

