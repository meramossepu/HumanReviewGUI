# Graphical User Interface (GUI) to aid human review of processed-ground-motions.
## By University of California, Los Angeles

The geotechnical group in the University of California, Los Angeles (UCLA) prepared a graphical user interface (GUI) to further improve records processed by USGS' automated ground motion processing software, gmprocess. Currently, gmprocess is limited due to the dificulties of automating an universal-noise-threshold. In order to overcome these limitations, the user can use this GUI to review filtered and unfiltered traces to verify the quality of the processed data. Acceleration and displacement time series, as well as response spectra can be visually inspected to find any potential errors. The GUI will allow the user to interact with the data (i.e. improve corner frequency selection or reject noisy records). In the example below we describe the procedure to run and interact with the GUI. 

## Prerequisites
- The GUI is designed to read the workspace.h5 output from gmprocess. Hence, the user needs to previously process the data in gmprocess. For example `gmrecords --eventid nc73291880 download`, `gmrecords assemble` and `gmrecords process_waveforms` must be ran before going through this review step. 
- Python script named `HumanReviewGUI.py` that contains several functions that will be called out in the Jupyter Notebook. 
- Create the Jupyter Notebook with the lines shown below to call in the class.

```ruby
%matplotlib widget
import HumanReviewGUI_working as HRG
GUI = HRG.HumanReviewGUI()
```

## Plots description
The interface is shown in Figure 1. Running the Jupyter Notebook will plot acceleration and displacement traces on the left panels while Fourier transform and response spectra will be plotted on the right. Information such as user, magnitude and epicentral distance (computed from `gps2dist_azimuth`), high-pass corner frequency (fchp) and low-pass corner frequency (fclp) are shown in the header. 
<br/><br/>
The orange traces depict the unfiltered record. In this context, "unfiltered" refers to the trace that has not gone trough the Butterworh filter, nonetheless, it is trimmed, demeaned and tapered. The blue trace ("filtered GUI") is filtered using the `apply_filter` function inside the python script and it will change if any of the corner frequencies are modified. The blue time series includes the minimum between 60 seconds of noise and the maximum noise lenght of a record while the traces called from gmprocess have 2 seconds (default in config file). The translucent red ("filtered gmprocess") is the record filtered inside gmprocess. This trace will not change if the corner frequencies are modified. The purpose of including both filtered traces (blue and red) is so the user can compare any improvements done in the GUI. The red trace was set to transparent to aid the visual comparison. As a result of overimposing the red over the blue, this trace will seem purple whenever "filtered GUI" overlaps "filtered gmprocess". The smoothed noise spectra is shown in green and only for the Fourier transform spectra.  The black dotted line is the theoretical acceleration decay at low frequency according to the $f^2$ model (Brune 1970; Boore and Bommer 2005), the user can use it as reference. The dashed-black lines depict corner frequencies in the fourier spectra and the highest-usable period (factor of 0.8) in the response spectra. 

![interface](interface.png)*Figure 1: Interface of Graphic User Interface for human review*

## Procedure
The GUI will not update the `provenance` folder in the workspace.h5. Instead, the GUI will write a section in the `auxiliary_data` folder named `review`. In the `review` section you can find which records were accepted or rejected and the correspondant corner frequency values. To integrate these changes into the `provenance` and update the `waveforms`, the user needs to run `gmrecords process -r`.
- Choose, from the dropdown menu, the gmprocess version used to process the data.
- Load workspace.h5
- Evaluate acceleration, displacement, amplitude and response plots.
- Increase values of fchp if necessary.
- Apply value of fclp if necessary.
- Accept or reject the record accordingly. The GUI will automatically proceed to the next trace.
- Run `gmrecords process -r` to overwrite the traces objects in the workspace.h5.

### Changes in high-pass corner frequencies
The high-pass corner frequency (fchp) can be modified to improve the automated selection. The automated selection of fchp is done by `auto_fchp` inside gmprocess. Sometimes, the automated fchp value is not high enough to discard lingering long period noise (which can be observed as a wobbly displacement trace, shown in Figure 3). 

![fchp_before](fchp_before.png)*Figure 3: Example trace with an automated selected fchp resulting in an irregular displacement time series*

![fchp_after](fchp_after.png)*Figure 4: Example trace shown in Figure 3 with a modified fchp resulting in a regular displacement time series*

### Changes in low-pass corner frequencies
Depending on how the user configured their gmprocess, the low-pass corner frequency (fclp) could be applied every time or never. In the GUI, by default, fclp is not applied to the blue trace ("filtered GUI"). The value shown in the header of the plots is the fclp selected by gmprocess which is chosen to be the minimum between $0.7*Nyquist$ frequency and the highest frequency where SNR undergoes certain threshold. Short period noise could be reflected as an irregular peak in the response spectra at short periods (Figure 5). The noise can be excluded from the trace by checking the box to the right and applying an fclp of the same value  (Figure 6).

![fclp_before](fclp_before.png)*Figure 5: Example trace without fclp resulting in an irregular response spectra shape*

![fclp_after](fclp_after.png)*Figure 6: Example trace shown in Figure 5 with fclp resulting in a flat response spectra at short periods*

### Example of rejected record
Some records contaminated with significant noise might pass the "failing criteria" of gmprocess. Usually, these records are easy to spot in the acceleration time series. An example is shown in Figure 7.

![rejected](rejected.png)*Figure 7: Example trace with significant noise that should be rejected*

## Limitations
1. The GUI is written in a way that the blue traces will not be updated if both corner frequency values do not change from one record to the next. Which occationally occurs. In such case, slightly alter one of the corner frequency values so the plots update. Alternatively, go to the next record and navigate back to the one of interest (most likely, one of the corner frequencies will change and you will obtain the updated figures). Indicators of this shortcoming are (a) if the blue trace is missing in the response spectra, or (b) if the "filtered GUI" and "filtered gmprocess" response spectra do not overlap for the majority of the trace. Figure 8 shows the figures that fail to update and Figure 9 shows the actual traces once it was succesfully updated.

![Same_fc_before](Same_fc_before.png)*Figure 8: Example trace that failed to update the blue trace because the previous plotted record had the exact same corner frequencies. This results in the red trace disapearing or not overlapping with the blue trace in the response spectra*

![Same_fc_after](Same_fc_after.png)*Figure 9: Example trace shown in Figure 8 once the blue trace was succesfully updated by going back and forth to a record with different corner frequencies*

2. Gmprocess might misscompute the location of the p-wave arrival or the coda. None of those issues could be further improved using the GUI. 
3. The GUI will not update the `provenance` folder in the workspace.h5. If the user wants to pass this information from the `auxiliary_data` folder to the `provenance` folder and recompute the `waveforms`, the user needs to run `gmrecords process -r`.
4. The GUI will convert the units of unprocessed traces into $m/s^2$ if the raw data is in g units or in a mseed format. Otherwise, the traces in the plots will be offset and the user will have to go to the python code and apply the correspondant conversion.
