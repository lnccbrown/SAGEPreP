# SAGEPreP: Semi-Automated GUI-based EEG Preprocessing.
EEG data are typically contaminated with artifacts (e.g., by eye movements, electrical contamination, and facial muscle movements), that must be removed from the data prior to analysis. Manual artifact rejection methods are largely time-consuming when applied to high-density or long-duration EEG data. Here, we present a semi-automated method to identify and delete temporal and spatial artifacts from EEG data. This pipeline utilizes parameters from the FASTER (Fully Automated Statistical Thresholding for EEG artifact Rejection) pipeline, with the addition of a user-friendly GUI and additional checkpoints for the reviewer to supervise artifact classification. Independent component analysis (ICA) is used to separate EEG data into neural activity and artifact. The semi-automated approach is particularly helpful with large datasets, and is designed to streamline manual review and target specific causes of artifact.

Before beginning, be sure that you have downloaded the EEGLab and FASTER software packages. Add FASTER to the plugins section of EEGLab and list the EEGLab software folder in your MATLAB search path. 

For info on FASTER: https://dx.doi.org/10.1016/j.jneumeth.2010.07.015
To download FASTER directly: https://sourceforge.net/projects/faster/

Preprocessing EEG data requires two steps: 
1. Run ProcessingPipeline to handle semi-automated preprocessing
2. Run ReviewEpochs to manually reject epochs that contain artifact  
Running ProcessingPipeline.m 
Before preprocessing, read through all recording notes to ensure that there were no issues with data collection and that tasks were flagged correctly. If tasks were incorrectly flagged, you may need to make changes to the EEG dataset before beginning. 

Specify the input variables in lines 22-53 of the code before running the pipeline. Variables include: 
* Initial reference: the channel(s) to which input data will be initially referenced (the nasion is suggested) 
* Average re-referencing: specify whether or not the data should be re-referenced to the average reference (after noisy  or failed channels are rejected) 
* Epoching: specify whether or not the data should be divided into epochs and if yes, at what epoch length (in seconds) 
* Filtering: whether lowpass, highpass, and notch filtering should be performed and if yes, at what frequencies 
* Task segmentation: whether or not the data should be segmented into individual files defined by event/task flags, and if yes, how those flags are represented in the data and what name they should be saved under in the outputs. For example, specifying ‘EyesClosed’ as an event name (for data output) and {'EyCl','eyec','eycl'} as the matching event flag(s) (found in the EEG data structure) will tell the pipeline to look for any of these three representations of an “eyes closed” flag in the data structure, segment from that flag to the next one, and save the preprocessed data in that interval in the ‘EyesClosed’ file. 

Once you have verified that data were adequately collected, tasks (if collected) are accurately flagged, and all input variables are specified, run ProcessingPipeline.m. Upon running the processing pipeline, a dialog box will pop up, asking you to select the “start directory.” This is the directory which holds all of the files you wish to process. Once you click “OK,” another dialog box will pop up asking you to select the “output directory.” This is the directory where all processed files will be sent. You must select these two directories before the pipeline can proceed (note: you can create a new output directory using the “New Folder” option in the lower left-hand side of the dialog box). For the purposes of our study, the start directory is the folder named “ToClean” and the output directory is the folder named “Completed.” Individual folders within the output directory will be created for each EEG dataset in the start directory, and will be given the same names as the EEG datasets. Note that the pipeline will process all .set (EEGLab), .raw (EGI: Electrical Geodesics Incorporated), .edf (European Data Format), .bdf (BioSemi Data Format), .sma (Snapmaster), and .cnt (Neuroscan) files in the start directory and all subfolders it contains, these being the most common EEG file formats. If you wish to load data from a file format not specified here, simply load it manually using the eeglab interface and save it as a “dataset.” This will create a .set file which can then be fed into the preprocessing pipeline. The program requires only one of these EEG data files, and a channel locations file if not included in the EEG data structure, in order to run.  
The pipeline will then proceed to bandpass filter and notch filter the data. By default, the data are bandpass filtered at 0.1-70 Hz and notch filtered at 60 Hz.
In the unlikely case that the channel locations file (included in the EEGLab software package download) is missing or cannot be found, the pipeline will ask you to provide one. In this case, navigate to the “sample_locs” folder within your EEGLab package download and select the appropriate channel locations file for your data. If the locations file is not in your EEGLab software download, navigate to ftp://sccn.ucsd.edu/pub/locfiles/ and download the appropriate file. More information on standardized channel locations can be found here. Channel locations for the commonly used EGI GSN 128-channel cap have been saved in the preprocessing pipeline for ease of use, and you may be prompted to select or reject this channel setup if your channel locations file cannot be found on first pass. Note that the channel locations file is required for ICA.  
At this point, processing will proceed with an automated search for bad channels, based on three statistical parameters calculated from the spread of the EEG data:
1. Variance (a contaminated channel will have higher variance) 
2. Mean correlation (the mean of the channel’s correlation coefficients with all other channels) 
3. Hurst exponent (The Hurst exponent is a measure of the long-term memory of a series, related to its autocorrelation. Neural data are expected to have a Hurst exponent of about 0.7)
When the automated channel analysis is complete, an EEG data scroll will pop up, along with a “Visual data inspection” dialogue box in which channels flagged as artefactual will be listed (screenshot included on page 3). These same channels which the automated search has identified as artefactual will appear in red in the data scroll. Scroll through the data, making sure to scroll up and down to visualize all channels and right and left to visualize all time points over which the recording was collected. Inspect all channels and edit the channels stated in the dialog box to reflect your manual inspection of the data. Use this step of the process to both review suggested rejections and manually search for bad channels that automatic detection may have missed. Note that sweat artifact is most commonly found in the temporal and occipital electrodes and those nearest the forehead. A diagram of the sensor layout for the 128-channel EGI HydroCelTM Geodesic Sensor Net is shown on the following page for reference. Be sure to manually scroll through all channels at all time points, as the quality of data collection may change over the length of the recording. As the algorithm calculates channel data metrics which cannot be easily visualized, as you review the channel data, utilize the algorithm’s suggestions to evaluate data quality on not only on the immediate visual appearance of the channel data, but also on the effects of its spatial organization. Note that although the sensor layout map identifies a reference channel in the middle of the scalp, the EEG channel data are re-referenced to the nasion (electrode 17) during preprocessing before visualization. Keep in mind that the suggested rejections are based on the statistical distribution of the data: you will likely have to accept more channels than are suggested for relatively clean data, and reject more than are suggested for relatively messy data. If there are no bad channels, simply leave the box empty and click “OK”. As soon as you click “OK”, processing will resume. All selected and verified bad channels will at this point be pulled out of the dataset, and processing will continue. These channels will be spline interpolated from neighboring channels with clean data after ICA processing and component rejection is complete. 

Note: During this checkpoint, you should also confirm the data are not artefactual to the point that the recording is unusably bad. If you do find the data to be so contaminated that it will be of no reliable use, stop here and report the session unusable. 

You may navigate to “Settings” > “Number of channels to display” to change the number of channels displayed on the y-axis, allowing easier identification of channel activity, should channel data overlap in the data scroll (default: 32 channels). Note that the channel, the time, and the value in μV of the data point over which the mouse is positioned are displayed at the bottom of the figure, indicated below in red). You may also change the amplitude scaling factor of channel data by changing the value in the dialogue box in the lower right hand side of the GUI or by pressing the “+” or “-“ keys (default: 50 μV). You should reject channels that are contaminated by sweat (slow, oscillating sweat artifact) or scalp movement/electrical artifact (large amplitude, non-neural waveforms).

If you have specified in the input variables that you would like the data to be epoched, the pipeline will then segment the data into epochs (default: 1 second), and automatically search for bad epochs to reject, based again on three statistical parameters calculated: 
1. Amplitude range (artifacts can be identified by abnormally high amplitude) 
2. Variance (muscle artifact produces high variance) 
3. Channel deviation (shifting electrodes may affect the data transiently, over the span of one or more epochs)  
The pipeline will then reject these bad epochs, re-reference the EEG data to the average reference, and run ICA to compute individual components. After calculating individual components (ICs), the pipeline will automatically search for ICs to suggest for rejection, based on five statistical parameters: 
1. Correlation with EOG channels (identifies components contaminated by blink or otherwise ocular artifact) 
2. Spatial kurtosis (measures the “peakedness” of data, identifying a short, high-amplitude, single-electrode offset, often  termed a “pop-off,” which is a common type of artifact singled out by ICA) 
3. Slope in filter band (white noise has a close-to-linear frequency power spectrum and can be identified by calculating  the slope of the spectrum over the low-pass filter band) 
4. Hurst exponent (neural data are expected to have a Hurst exponent of about 0.7) 
5. Median gradient (above threshold if the individual component contains considerable high-frequency content)  
Having identified ICs recommended for rejection, a series of windows displaying multiple ICs will pop up. Recommended rejections are colored red in the pop-up window. You should click on the numbered red or green buttons to review the recommended rejections and then change the classification to “ACCEPT” or REJECT” in the window that pops up as necessary. As individual components are simply spatial reorganizations of the time series data collected through EEG channels, you should review the component features in this pop-up window as criteria for rejection as you would untransformed EEG channel data. At the top left of the pop-up figure will be a topological map of component activations: this will be particularly helpful in identifying ocular artifacts. On the topological map, a component with ocular artifact will have the greatest activation in the electrodes surrounding the eyes, with a linear gradient from the eyes back over the scalp – in other words, red at the eyes becoming progressively bluer towards the back of the head. Ocular artifact may also appear as a concentrated red dot at the eyes. At the bottom of the pop-up figure will be a frequency-power plot. Here, you should look for a typical 1/f distribution, as you would with untransformed EEG channel data. A bump in the power-frequency spectrum around 20-30 Hz may suggest muscular artifact, and heightened power at 10 or more Hz which exceeds the power at 1-2Hz coincident with a topological plot suggesting heightened activity near the eyes on the topological map most likely suggests ocular artifact. In the top right corner of the pop-up figure will be a spectrogram, showing frequency-power distributions per epoch, epochs being ordered along on the x-axis, and below that a scroll of the component time series data. You may pan through epochs using the scroll bar below the time series plot; as you move through epochs, the red vertical bar appearing in the spectrogram will adjust its position to indicate the epoch that you are viewing. Here, again, as you review the component time series data consider the same rejection criteria that you would use to review untransformed EEG channel data. If you categorize the IC as non-neural, change/maintain the classification as “REJECT”, and click “OK”. The numbered button above this IC in the master window displaying the group of ICs will then be colored red, identifying to the reviewer that this component will be rejected when processing continues. Otherwise, if you categorize the IC as neural data and change/maintain the classification as “ACCEPT”, the numbered button above the IC will be colored green and the component data will remain in the dataset. Note that because the individual components are ordered by decreasing variance, the majority of the components that will need to be rejected are most likely to appear within the first few display windows 

When you click the final “OK” box, the pipeline will then proceed to reject designated components and spatially transform the data back into EEG channel data.

After ICA rejection, you will not need to interact with the ProcessingPipeline.m script again. The pipeline will interpolate channels (those selected for interpolation after manual review as well as those identified by the algorithm). Then, if you have specified so in the input variables, the pipeline will segment the data into task-specific segments, as identified by the event flags annotated during data collection. If you have also specified that the data should be epoched, then for each task-specific segment interpolate channels within epochs (for these channels with transient issues, affecting only some epochs and not the entirety of the recording). Similar to before, the automated search for bad channels is based on four statistical parameters calculated from the spread of the EEG data: 
1. Variance (a contaminated channel will have higher variance) 
2. Median gradient (the median slope of the channel within the epoch) 
3. Amplitude range (to detect pop-offs) 
4. Channel deviation (shifting electrodes may affect the data transiently, over the span of one or more a few epochs)  
The channels selected for interpolation within channels are saved in log file format in the subdirectory “Channel interpolations by epoch” under the name “<subject name>_<task>_channel_interpolations_by_epoch.txt”. 
The final outputs of preprocessing are found in the output directory, by the task specified by event markers and the original file name (ex: S0064_1_EyesClosed_1.set). The original, raw file can be found here as well, under the name “<subject name>_original.set. Also in this output directory are the subdirectories “Component maps”, “Channel interpolations by epoch”, and “Intermediate”, which contains iterations of the EEG data saved at intermediate points during processing. 

A log file is recorded and saved in the output directory for each set of EEG data processed, titled by the subject/session name with the extension “.log” (ex. “S0064_1.log”). The log file details every step of processing that was completed, including 
* the frequencies at which the EEG data were filtered, the channels that were rejected and re-interpolated (along with the reasons for their rejection) 
* the number of epochs that were rejected 
* the number and indices of independent components that were rejected  See the screenshot below for an example of log file content – numbers beginning each line represent time elapsed in seconds since processing began:  As mentioned previously, in a separate file, titled by the name of the EEG (per task) and “_channel_interpolations_by_epoch.txt”, you will find lists of the channels that were interpolated in specific epochs, ordered by the indices of these epochs.

The final step of preprocessing is to manually search for bad epochs (the algorithm used in ProcessingPipeline has about 60% epoch sensitivity). 
* First, enter “ReviewEpochs” into the command line, and in the window that pops up, select the appropriate “.set” file: one of the outputs of the above processing. 
* Once you’ve selected the EEG file, a data scroll will pop up in which you can click on epochs to manually mark them for rejection. Using this interface, review all epochs and mark those exhibiting muscle, ocular, or electrical artifact. Epochs designated for rejection will be colored yellow in the interface. After reviewing and classifying all epochs, click the “REJECT” button in the bottom-righthand corner to reject all artefactual epochs at once. 
* In the window that pops up, save the final output EEG data in the “Epochs reviewed” folder within the ouput directory. The save path and file name will be suggested for you. See below for further details on the epoch rejection process. 
* Repeat this process for every “.set” file created by the output of the previous preprocessing step (segmented by task / data type).
 
Final output EEG data from the last stage of processing, ReviewEpochs, will be saved within the folder “Epochs reviewed” under their original file names, with “_epochs_reviewed” appended: 

Please email Lauren Ostrowski at lauren_ostrowski@brown.edu with any bugs, questions, or concerns.
