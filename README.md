## Overview
This repository contains all the files (2 apps and 3 scripts) necessary to run the Analysis Tool. The objective of this tool is to allow for a quick and easy way to analyze EEG/MEG data (pre-processing and processing) based on functions from the [Brainstorm](https://neuroimage.usc.edu/brainstorm/Introduction) software. It also allows to convert data or save pipeline in an easy to share format. This tool is still in development (adding file format, processes, etc.).

## 1. Applications
### 1.1 Analysis Tool
<p float="left">
    <img src="https://user-images.githubusercontent.com/88212708/141159087-339d6ff9-8404-4e87-ad3d-972087b1b93b.PNG" width="475"/>
    <img src="https://user-images.githubusercontent.com/88212708/141159544-cc9ad06c-11aa-47fa-97d8-49318bfa2910.PNG" width="475"/>
</p>

The Analysis Tool App is the main app. It allows the user to perform MEG or EEG analysis. The user can create or delete protocols/subjects/studies, import anatomy/data and apply multiple processes. There is two ways to perfom an analysis. 

One way is to apply each process individually, which allows the user to view the data between each process. The user can also view and modify (create, rename or delete) the events or consult the history of each study.

The other way is to run a pipeline. From the main app, the user can import a pipeline previously created with the Pipeline Builder (explained later) and apply it to multiple studies.

The code in the main app serves the purpose of running the interface, asking the user for information, organizing files and folders, throwing errors when needed, etc. When the user applies a process, the main app calls the appropriate script (EEG_Pipeline, MEG_Pipeline or Utility_Pipeline) that contains all the processes.

#### Supported File Format
Here is the list of the supported recording software and file format that can be imported. We will be adding new software and file format as we go!
- BrainVision (.eeg, .vhdr, .vmrk)
- BioSemi (.eeg, .vhdr, .vmrk)
- NeuroLite - Coherance Software (.bin, .elc)
- CTF MEG 275 (.meg4)

#### Anatomy
When creating a new subject, there is the possibility to use a default anatomy or import a specific anatomy. 

The default anatomy is a template provided by Brainstorm. Multiple templates are available (template for babies, children, young adults, adults, etc.). 

There is also the possibility to import the specific anatomy of the subject. For estimating the brain sources of the MEG/EEG signals, the anatomy of the subject must include at least three (3) files: a T1-weighted MRI volume, the envelope of the cortex and the surface of the head. The user will then have to mark the fiducials points (nasion, left ear, right ear, anterior commissure, posterior commissure, inter-hemispheric point).


### 1.2 Pipeline_Builder
<p float="left">
    <img src="https://user-images.githubusercontent.com/88212708/141160093-495c6247-2fd3-48f9-8f65-996f0d22fab9.PNG" width="475"/>
    <img src="https://user-images.githubusercontent.com/88212708/141160092-d03bebcb-c83f-4299-9984-1df2f7377410.PNG" width="475"/>
</p>
This app allows the user to create, save and modify pipelines. It can be opened from the main app. To build the pipeline, the user select the processes and enters the information needed for each process. The pipeline is then saved as a MatLab structure (.mat) that can be imported in the main app and applied on studies. 

#### MatLab Structure
Here is an example of a pipeline structure:
```
├── DateOfCreation
│   └── YYYY-MM-DD-HH-MM-SS
├── Folder
│   └── C:\...
├── Name
│   └── PipelineName
├── NumberOfProcesses
│   └── 4
├── Processes
│   ├── AddEEGPosition
│   │   ├── FileType
│   │   │   └── Use Default Pattern
│   │   └── Cap
│   │       └── Colin27: BrainProducts EasyCap 128
│   ├── NotchFilter
│   │   └── Frequence
│   │       └── 60, 120, 180
│   ├── BandPassFilter
│   │   ├── LowFreq
│   │   │   └── 20
│   │   └── HighFreq
│   │       └── 140
│   └── ICA
│       └── NumberOfComponents
│           └── 32
└── Type
    └── EEG 
```

## 2. Scripts
### 2.1 EEG_Pipeline
This script contains all the process that can be applied on an EEG study. When an EEG process is applied from the main app, this script is called. It takes as input (1) a cell of the studies (the path to the study .mat file) on which to apply the process and (2) a MatLab structure with the parameters of the process to apply.

Processes available:
- _**Add EEG Position:**_ Import the positions of the electrodes.
- _**Refine Registration:**_ Finds a better registration between the head shape defined by the electrodes and the head surface coming from the MRI. Note that this works only if have accurate head shapes (i.e. for a specific anatomy) and electrodes positions.
- _**Project Electrode on Scalp:**_ Ensures all the electrodes touch the skin surface (only for specific anatomy).
- _**Notch Filter:**_ Notch filters are adapted for removing well identified contaminations from systems oscillating at very stable frequencies.
- _**Band Pass Filter:**_ A band-pass filter is the combination of a low-pass filter and a high-pass filter, it removes all the frequencies outside of the frequency band of interest.
- _**Power Spectrum Density:**_ This process evaluates the power of the MEG/EEG signals at different frequencies, using the Welch's method.
- _**Average Reference**_ Creates a linear projector that re-references the EEG.
- _**ICA:**_ Identifies spatial topographies (components that areindependent in time) specific to an artifact and then removes them from the recordings.

### 2.2 MEG_Pipeline
This script contains all the process that can be applied on a MEG study. When a MEG process is applied from the main app, this script is called. It takes as input (1) a cell of the studies (the path to the study .mat file) on which to apply the process and (2) a MatLab structure with the parameters of the process to apply.

Processes available:
- _**Convert Epoch To Continue:**_ Brainstorm automatically imports data as epoched files. This process convert epoched files to continous files.
- _**Notch Filter:**_ Notch filters are adapted for removing well identified contaminations from systems oscillating at very stable frequencies.
- _**Band Pass Filter:**_ A band-pass filter is the combination of a low-pass filter and a high-pass filter, it removes all the frequencies outside of the frequency band of interest.
- _**Power Spectrum Density:**_ This process evaluates the power of the MEG/EEG signals at different frequencies, using the Welch's method.
- _**Detect Artifact:**_ This process can be used for detecting any kind of event (heartbeat, blink or other) based on the signal power in a specific frequency band.
- _**Remove Simultaneous Events:**_ In order to clean the signal effectively (using SSP), each artifact should be defined precisely and as independently as possible from the other artifacts. This means that we should try to avoid having two different artifacts marked at the same time.
- _**SSP:**_ The general SSP objective is to identify the sensor topographies that are typical of a specific artifact, then to create spatial projectors to remove the contributions of these topographies from the recordings.
- _**ICA:**_ Identifies spatial topographies (components that are independent in time) specific to an artifact and then removes them from the recordings.

### 2.3 Utility_Pipeline
This script contains all the process needed to run the basic operations that do not affect the data (import anatomy, review raw files, etc.). It takes as input (1) a MatLab structure with the parameters of the process to apply.

Processes available:
- _**Import Anatomy:**_ Creates a specific (or default) anatomy for the subject.
- _**Review Raw Files:**_ Creates a link to the original EEG files.
- _**Convert to BIDS:**_ Export EEG and MEG files following the standard data organization of the Brain Imaging Data Structure (BIDS). The data files will previously be converted to .edf.
- _**Import in database:**_ Extracts events from processed files and import them in the database.

#### Conversion to BIDS
Before the conversion to BIDS, the data files are converted to .edf. When converting to BIDS, a function is ran to create 4 files:
- <label>_events.tsv: List of all the occurence of an event.
- <label>_events.json: Meta data about every event.
- <label>_provenance.json: History of all the process applied on the data.
- <label>_channelCoordinates.json: Channel coordinates.

Here is the structure of a BIDS folder (based on the [BIDS specifications for Electroencephalography](https://bids-specification.readthedocs.io/en/stable/04-modality-specific-files/03-electroencephalography.html)):
    
```
BIDS
├── derivatives
│   └── bst_db_mapping.mat
├── sub-<label>
│   └── ses-YYYYMMDD
│       ├── anat
│       │   └── sub-<label>_ses-YYYYMMDD_T1w.nii.gz
│       ├── eeg
│       │   ├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<index>]_eeg.edf
│       │   ├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<index>]_eeg.json
│       │   ├── <label>_events.tsv
│       │   ├── <label>_events.json
│       │   ├── <label>_provenance.json
│       │   └── <label>_channelCoordinates.json
│       └── sub-<label>_ses-YYYYMMDD_scans.tsv
├── dataset_description.json
└── README
```
