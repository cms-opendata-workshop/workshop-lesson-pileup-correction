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

Next we add this boolean variable to the function DeclareVaribles (line 214). The function should take 'isData' as a parameter.

```cpp
auto DeclareVariables(T &df, bool isData)
~~~
```

Because the data events do not contain pileup-variables, we will add the following code (to line 257) to make sure pileup-variables are declared only for simulated events.

```cpp
if(!isData){ // for simulated events add also pileup-variables
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
                .Define("pileup_tot", "Pileup_total_number") // new pileup-variables
                .Define("pileup_true","Pileup_true_number");
    }
```

From line 376 are the final variables. Those should be changed to 'finalVariables' for simulated events and 'finalVariablesData' for data events.

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

Now we create the boolean variable in the main function and choose the path to correct files. Add the following code to line 416.

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

Last, let's choose the correct variables (add to line 442):

```cpp
if(dataRun){
  dfFinal.Snapshot("Events", sample + "Skim.root", finalVariablesData);
} else {
  dfFinal.Snapshot("Events", sample + "Skim.root", finalVariables);
}
```

## histograms.py

Let's create some histograms.

## plot.py



{% include links.md %}
