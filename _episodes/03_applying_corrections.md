---
title: "Adding pileup corrections in the HTT analysis example"
teaching: 10
exercises: 20
questions:
- "How to apply the pileup corrections in a specific analysis?"
objectives:
- "TODO"
keypoints:
- "TODO"
---

## Skimming

In order to add the pileup corrections, we need to use simulated events that contain the variable 'pileup'. For this we will need to add some code to [skim.cxx](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/skim.cxx). Start by adding the following code to line 29. This will be used to get the simulated events while the original path in the analysis will be used for the data events.

```cpp
/*
 * Path to new version of simulations
 */
const std::string newSamplesBasePath = "root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws1.0/";
```


To use the correct path we will add a boolean variable to the main function that tells which path to use and how to process the dataset. The addition will be presented later.

Next we add the boolean variable to the function DeclareVaribles (line 214). The function should take 'isData' as a parameter.

```cpp
auto DeclareVariables(T &df, bool isData)
```


Because the data events do not contain pileup variables, we will add the following code to line 257 to make sure pileup variables are declared only for simulated events.

```cpp
if(!isData){ // for simulated events add also pileup variables
      return df.Define("pt_1", "Muon_pt[idx_1]")
                .Define("eta_1", "Muon_eta[idx_1]")
                .Define("phi_1", "Muon_phi[idx_1]")
                .Define("m_1", "Muon_mass[idx_1]")
                .Define("iso_1", "Muon_pfRelIso03_all[idx_1]")
                .Define("q_1", "Muon_charge[idx_1]")
                .Define("pt_2", "Tau_pt[idx_2]")
                .Define("eta_2", "Tau_eta[idx_2]")
                .Define("phi_2", "Tau_phi[idx_2]")
                .Define("m_2", "Tau_mass[idx_2]")
                .Define("iso_2", "Tau_relIso_all[idx_2]")
                .Define("q_2", "Tau_charge[idx_2]")
                .Define("dm_2", "Tau_decayMode[idx_2]")
                .Define("pt_met", "MET_pt")
                .Define("phi_met", "MET_phi")
                .Define("p4_1", add_p4, {"pt_1", "eta_1", "phi_1", "m_1"})
                .Define("p4_2", add_p4, {"pt_2", "eta_2", "phi_2", "m_2"})
                .Define("p4", "p4_1 + p4_2")
                .Define("mt_1", compute_mt, {"pt_1", "phi_1", "pt_met", "phi_met"})
                .Define("mt_2", compute_mt, {"pt_2", "phi_2", "pt_met", "phi_met"})
                .Define("m_vis", "float(p4.M())")
                .Define("pt_vis", "float(p4.Pt())")
                .Define("npv", "PV_npvs")
                .Define("goodJets", "Jet_puId == true && abs(Jet_eta) < 4.7 && Jet_pt > 30")
                .Define("njets", "Sum(goodJets)")
                .Define("jpt_1", get_first, {"Jet_pt", "goodJets"})
                .Define("jeta_1", get_first, {"Jet_eta", "goodJets"})
                .Define("jphi_1", get_first, {"Jet_phi", "goodJets"})
                .Define("jm_1", get_first, {"Jet_mass", "goodJets"})
                .Define("jbtag_1", get_first, {"Jet_btag", "goodJets"})
                .Define("jpt_2", get_second, {"Jet_pt", "goodJets"})
                .Define("jeta_2", get_second, {"Jet_eta", "goodJets"})
                .Define("jphi_2", get_second, {"Jet_phi", "goodJets"})
                .Define("jm_2", get_second, {"Jet_mass", "goodJets"})
                .Define("jbtag_2", get_second, {"Jet_btag", "goodJets"})
                .Define("jp4_1", add_p4, {"jpt_1", "jeta_1", "jphi_1", "jm_1"})
                .Define("jp4_2", add_p4, {"jpt_2", "jeta_2", "jphi_2", "jm_2"})
                .Define("jp4", "jp4_1 + jp4_2")
                .Define("mjj", compute_mjj, {"jp4", "goodJets"})
                .Define("ptjj", compute_ptjj, {"jp4", "goodJets"})
                .Define("jdeta", compute_jdeta, {"jeta_1", "jeta_2", "goodJets"})
                .Define("pileup_tot", "Pileup_total_number") // new pileup variables
                .Define("pileup_true","Pileup_true_number");
    }
```


The final variables are listed after line 376. Those should be changed to 'finalVariables' for simulated events and 'finalVariablesData' for data events.

```cpp
const std::vector<std::string> finalVariables = {
    "njets", "npv",
    "pt_1", "eta_1", "phi_1", "m_1", "iso_1", "q_1", "mt_1",
    "pt_2", "eta_2", "phi_2", "m_2", "iso_2", "q_2", "mt_2", "dm_2",
    "jpt_1", "jeta_1", "jphi_1", "jm_1", "jbtag_1",
    "jpt_2", "jeta_2", "jphi_2", "jm_2", "jbtag_2",
    "pt_met", "phi_met", "m_vis", "pt_vis", "mjj", "ptjj", "jdeta",
    "gen_match", "run", "weight",
    "pileup_tot","pileup_true"
};

const std::vector<std::string> finalVariablesData = {
    "njets", "npv",
    "pt_1", "eta_1", "phi_1", "m_1", "iso_1", "q_1", "mt_1",
    "pt_2", "eta_2", "phi_2", "m_2", "iso_2", "q_2", "mt_2", "dm_2",
    "jpt_1", "jeta_1", "jphi_1", "jm_1", "jbtag_1",
    "jpt_2", "jeta_2", "jphi_2", "jm_2", "jbtag_2",
    "pt_met", "phi_met", "m_vis", "pt_vis", "mjj", "ptjj", "jdeta",
    "gen_match", "run", "weight",
};
```


Next, we create the boolean variable in the main function and choose the path to correct files. Add the following code to line 416 and delete the old line for creating the path (ROOT::RDataFrame...).

```cpp
bool dataRun = sample.find("Run") != std::string::npos;

std::string path;

if(dataRun){
  path = samplesBasePath;
} else {
  path = newSamplesBasePath;
}

ROOT::RDataFrame df("Events", path + sample + ".root");
```


Give 'dataRun' as a parameter to DeclareVaribles (line 434).

```cpp
auto df7 = DeclareVariables(df6, dataRun);
```


Lastly, let's choose the correct variables (add to line 442):

```cpp
if(dataRun){
  dfFinal.Snapshot("Events", sample + "Skim.root", finalVariablesData);
} else {
  dfFinal.Snapshot("Events", sample + "Skim.root", finalVariables);
}
```


## Histograms

Next, we are going to create some histograms with [histograms.py](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/histograms.py). First, add the pileup variables to the end of the list of ranges (to lines 54 and 55).

```py
"pileup_tot":(50, 0, 50),
"pileup_true":(50, 0, 50),
```


Then we are going to add a function for getting the pileup corrections from the file. Make sure you have the correct path to the file in the function. We will also add a function that finds the correct pileup correction value for each event. Add the code to line 81.

```py
# Getting pileup corrections using gInterpreter
ROOT.gInterpreter.Declare("""
TFile *f = new TFile("pileupCorrection.root");
TH1F *puCorrectionFactors = (TH1F*)f->Get("puCorrectionFactors");
""")

# Function for finding correct pileup correction
ROOT.gInterpreter.Declare("""
float findPuCorrection(float pileup_true) {
    if(pileup_true < 5){
        return 1.0;
    } else if(pileup_true > 50){
        return 0;
    }

    auto puWeigth = puCorrectionFactors->GetBinContent(pileup_true);
    return puWeigth;
}
""")
```


After adding the pileup corrections we need to scale back the histograms. The following code will create dataframes marked 'old' that will be used for scaling. We will add the 'if' statement used here a few more times later on. It prevents using the pilup variables for data events.

```py
# Used for scaling new histograms with pileup corrections
df_old = df.Filter("q_1*q_2<0", "Require opposited charge for signal region")
df_old = filterGenMatch(df_old, label)
hists_old = {}
for variable in variables:
  if "Run" in name and "pileup" in variable:
    continue
  hists_old[variable] = bookHistogram(df_old, variable, ranges[variable])
report_old = df_old.Report()
```


Next, we will change the for-loop at line 154 to use the added correction. Again, we are skipping pileup variables for data.

```py
for variable in variables:
  if "Run" in name:
    if "pileup" in variable:
      continue
    # Data events
    hists[variable] = bookHistogram(df1, variable, ranges[variable])
  else:
    # Simulated events
    hists[variable] = df1.Define("total_weight", "findPuCorrection(pileup_true)*weight").Histo1D(ROOT.ROOT.RDF.TH1DModel(variable, variable, ranges[variable][0], ranges[variable][1], ranges[variable][2]), variable, "total_weight")
```        


Add the scaling to line 166.

```py
for variable in variables:
  if "Run" in name and "pileup" in variable:
    continue
  n_old = hists_old[variable].Integral()
  n_new = hists[variable].Integral()
  hists[variable].Scale(n_old/n_new)
```


Lastly, we add the 'if' statement a few more times:

```py
# Book histograms for the control region used to estimate the QCD contribution
df2 = df.Filter("q_1*q_2>0", "Control region for QCD estimation")
df2 = filterGenMatch(df2, label)
hists_cr = {}
for variable in variables:
  if "Run" in name and "pileup" in variable:
    continue
  hists_cr[variable] = bookHistogram(df2, variable, ranges[variable])
report2 = df2.Report()

# Write histograms to output file
for variable in variables:
  if "Run" in name and "pileup" in variable:
    continue
  writeHistogram(hists[variable], "{}_{}".format(label, variable))
for variable in variables:
  if "Run" in name and "pileup" in variable:
    continue
  writeHistogram(hists_cr[variable], "{}_{}_cr".format(label, variable))
```


## Plotting

Finally, let's plot the histograms. For this we need to make changes to [plot.py](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/plot.py). Again, we start by adding the pileup variables. This time to the 'labels'-list that begins from line 12.

```py
"pileup_true": "Pileup_true_number",
"pileup_tot": "Pileup_total_number",
```


A boolean variable will make things easier with plots as well, so let's create one at the beginning of the main function (line 82).

```py
isPileup = "pileup" in variable
```


We need to make sure pileup variables aren't used for data and QCD. To do this, put the code that starts from line 152 under an if statement as follows:

```py
if(not isPileup):
  # Data
  data = getHistogram(tfile, "dataRunB", variable)
  dataRunC = getHistogram(tfile, "dataRunC", variable)
  data.Add(dataRunC)

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
  QCDScaleFactor = 0.80
  QCD.Scale(QCDScaleFactor)

  data.SetMarkerStyle(20)
  data.SetLineColor(ROOT.kBlack)

# Draw histograms
ggH.SetLineColor(colors["ggH"])
qqH.SetLineColor(colors["qqH"])
```


Because separating ZTT and ZLL events does not work correctly for the new simulated events, we combine the events to one variable titled Z &#8594; *ll*. Add the following lines right after the code above.

```py
# Combine ZTT and ZLL
ZTT.Add(ZLL)
```


Next, we need to remove the 'ZLL' variable, use some more if statements and add titles for the pileup variables. Change the code that begins from line 189 to following:

```py
if(isPileup):
  for x, l in [(TT, "TT"), (ZTT, "ZTT"), (W, "W")]: # remove ZLL and QCD
      x.SetLineWidth(0)
      x.SetFillColor(colors[l])
else:
    for x, l in [(QCD, "QCD"), (TT, "TT"), (ZTT, "ZTT"), (W, "W")]: # remove ZLL
        x.SetLineWidth(0)
        x.SetFillColor(colors[l])

stack = ROOT.THStack("", "")
if(isPileup):
  for x in [TT, W, ZTT]: # remove ZLL and QCD
      stack.Add(x)
else:
  for x in [QCD, TT, W, ZTT]: # remove ZLL
      stack.Add(x)

c = ROOT.TCanvas("", "", 600, 600)
stack.Draw("hist")
if(variable == "pileup_true"):
  stack.GetXaxis().SetTitle("Pileup_true_number") # add titles for pileup variables
  stack.SetMaximum(7000) # to prevent legend from going on top of distribution
  stack.SetMinimum(1.0)
elif(variable == "pileup_tot"):
  stack.GetXaxis().SetTitle("Pileup_total_number")
  stack.SetMaximum(7000)
  stack.SetMinimum(1.0)
else:
  name = data.GetTitle() # titles for other variables
  if name in labels:
    title = labels[name]
  else:
    title = name
  stack.GetXaxis().SetTitle(title)
stack.GetYaxis().SetTitle("N_{Events}")
if(not isPileup):
  stack.SetMaximum(max(stack.GetMaximum(), data.GetMaximum()) * 1.4)
  stack.SetMinimum(1.0)

ggH.Draw("HIST SAME")
qqH.Draw("HIST SAME")

if(not isPileup):
  data.Draw("E1P SAME")
```


Same changes are needed for legends as well.

```py
# Add legend
legend = ROOT.TLegend(0.4, 0.73, 0.90, 0.88)
legend.SetNColumns(2)
legend.AddEntry(ZTT, "Z#rightarrowll", "f") # new title for ZTT and ZLL combined
legend.AddEntry(W, "W+jets", "f")
legend.AddEntry(TT, "t#bar{t}", "f")
legend.AddEntry(ggH, "gg#rightarrowH (x{:.0f})".format(scale_ggH), "l")
legend.AddEntry(qqH, "qq#rightarrowH (x{:.0f})".format(scale_qqH), "l")
if(not isPileup):
    legend.AddEntry(data, "Data", "lep")
    legend.AddEntry(QCD, "QCD multijet", "f")
legend.SetBorderSize(0)
legend.Draw()
```

{% include links.md %}
