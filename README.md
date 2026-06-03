TCSPC Normalized Fitting Notebook
A Jupyter notebook for importing, calibrating, normalizing, fitting, plotting, and exporting TCSPC decay data.
This notebook is designed for TCSPC `.txt`, `.dat`, `.asc`, or `.csv` files that contain a channel/count table and a time calibration line, such as:
```text
Time calibration: 0.1097394ns/ch

Chan    Data
1       4
2       15
3       6
...
```
The notebook automatically converts channel number to time using the file-specific calibration, normalizes the decay to a maximum value of 1, and fits the decay using mono-, bi-, or tri-exponential models.
---
Features
Reads TCSPC files with `Chan` and `Data` columns
Extracts the `Time calibration: ... ns/ch` value from each file
Converts channel number to calibrated time in nanoseconds
Shifts the decay so that the peak occurs at `t = 0`
Normalizes the y-axis to 1 before fitting
Supports:
Mono-exponential fitting
Bi-exponential fitting
Tri-exponential fitting
Mono + bi comparison mode
Uses Poisson-style weighting during fitting
Outputs:
Fit plots
Residual plots
Calibrated trace CSV files
Fit parameter CSV files
Summary CSV file
Includes an interactive Jupyter widget interface for uploading files and selecting fit settings
---
Fit Models
Mono-exponential
```text
y(t) = A exp(-t / tau) + bg
```
Bi-exponential
```text
y(t) = A1 exp(-t / tau1) + A2 exp(-t / tau2) + bg
```
Tri-exponential
```text
y(t) = A1 exp(-t / tau1) + A2 exp(-t / tau2) + A3 exp(-t / tau3) + bg
```
For normalized data, the fit amplitudes are also normalized. For a good bi-exponential fit, the value:
```text
A1 + A2 + bg
```
should often be near 1, especially if the fit starts close to the decay peak. If the fit starts later in time, this value may deviate from 1 because the amplitudes are extrapolated back to `t = 0`.
---
Requirements
The notebook requires Python 3 and the following packages:
```text
numpy
pandas
scipy
matplotlib
ipywidgets
```
Install them with:
```bash
pip install numpy pandas scipy matplotlib ipywidgets
```
If you are using JupyterLab or Anaconda and the upload button does not respond, update widget support:
```bash
pip install --upgrade ipywidgets jupyterlab_widgets widgetsnbextension
```
Then restart Jupyter.
---
Recommended Environment
This notebook was written for use in:
Jupyter Notebook
JupyterLab
Anaconda Python environment
If using Anaconda, it is recommended to launch Jupyter from Anaconda Navigator or Anaconda Prompt to ensure the notebook uses the correct Python environment.
---
Input File Format
The notebook expects files with a metadata section followed by a channel/count table.
Example:
```text
Item name: Decay

Real time: 744.076
Live time: 0
Time calibration: 0.1097394ns/ch

Comment:


Chan    Data
1       4
2       15
3       6
4       9
...
```
The critical line is:
```text
Time calibration: 0.1097394ns/ch
```
This value is used to convert channels into time:
```text
time_ns = (channel - first_channel) * ns_per_channel
```
The notebook then shifts the time axis so that the decay peak is at:
```text
t = 0 ns
```
---
How to Use
1. Open the notebook
Open the notebook in Jupyter:
```text
TCSPC_Normalized_Fitting_Notebook.ipynb
```
2. Run all setup cells
Run the notebook cells from top to bottom until the upload interface appears.
3. Upload TCSPC files
Use the file upload widget to select one or more TCSPC files.
Supported extensions:
```text
.txt
.csv
.dat
.asc
```
4. Choose the fit model
Use the dropdown menu to select:
```text
mono-exponential
bi-exponential
tri-exponential
mono + bi comparison
```
5. Set fit window
The notebook uses two fit window settings:
```text
Fit start ns
Fit end ns
```
Example:
```text
Fit start ns = 0.2
Fit end ns = -1
```
A fit end value of `-1` means the fit continues to the end of the trace.
Increasing `Fit start ns` can help avoid fitting the prompt peak or instrument response region.
6. Run the fit
Click:
```text
Fit uploaded files
```
The notebook will process each uploaded file, generate plots, and save output files.
---
Output Files
The notebook saves outputs to:
```text
~/Downloads/TCSPC_fits
```
For each input file, the notebook may create:
```text
sample_calibrated_trace.csv
sample_fit.csv
sample_fit_parameters.csv
sample_fit.png
sample_residuals.png
```
For mono + bi comparison mode, it also creates:
```text
sample_mono_bi_comparison.png
sample_mono_bi_residuals.png
sample_mono_bi_comparison_parameters.csv
```
A combined summary file is also saved:
```text
TCSPC_fit_summary.csv
```
---
Understanding the Outputs
Main fit plot
The main plot shows:
Normalized TCSPC data
Fitted decay curve
Fit parameters
Reduced chi-square value
For mono + bi comparison:
Mono fit is plotted in black
Bi fit is plotted in red
Residual plot
Residuals are calculated as:
```text
residual = measured normalized counts - fitted normalized counts
```
A good fit should have residuals randomly scattered around zero.
Structured residuals, such as waves, slopes, or long regions above/below zero, may indicate:
The model is too simple
The background is incorrect
The fit starts too close to the prompt peak
Instrument response function effects are important
Reduced chi-square
The reduced chi-square value is:
```text
reduced chi-square = chi-square / degrees of freedom
```
General interpretation:
Reduced chi-square	Interpretation
~1	Very good fit
1-2	Usually acceptable
2-5	Possible model mismatch or underestimated noise
>5	Model likely inadequate
A reduced chi-square of 3 is not automatically bad, but residuals should be inspected carefully.
---
Notes on Normalization
The notebook normalizes each decay before fitting:
```text
normalized_counts = counts / max(counts)
```
This makes the peak intensity equal to 1.
For a normalized bi-exponential model:
```text
y(t) = A1 exp(-t / tau1) + A2 exp(-t / tau2) + bg
```
At `t = 0`:
```text
y(0) = A1 + A2 + bg
```
Therefore, for a well-behaved normalized fit:
```text
A1 + A2 + bg approx 1
```
If this value is far from 1, possible causes include:
Fit starts too far after the peak
Background is poorly estimated
The decay model is not appropriate
The prompt/IRF region is influencing the fit
The decay is not well described by simple exponentials
---
Instrument Response Function
This notebook performs tail fitting using exponential decay models. It does not currently perform IRF-convolved fitting.
The instrument response function, or IRF, is the response of the TCSPC system to an effectively instantaneous signal. The measured decay is approximately:
```text
measured decay = IRF convolved with true decay
```
IRF-convolved fitting may be needed if:
The lifetime is close to the instrument response width
Residuals are poor near `t = 0`
Changing `Fit start ns` changes the fitted lifetime significantly
Very short lifetimes are being analyzed
An IRF can often be measured using scattered laser light from a non-fluorescent scatterer or blank substrate.
---
Suggested Workflow
Start with mono-exponential fitting.
Check the residuals and reduced chi-square.
Try bi-exponential fitting.
Use mono + bi comparison mode to compare models directly.
Only use tri-exponential fitting if the improvement is physically meaningful.
Adjust `Fit start ns` to avoid the prompt/IRF region.
Save and compare fit summaries across samples.
---
Troubleshooting
Upload button does nothing
Run this test cell:
```python
import ipywidgets as widgets
from IPython.display import display

button = widgets.Button(description="Test button")
output = widgets.Output()

def on_click(b):
    with output:
        print("Button works!")

button.on_click(on_click)
display(button, output)
```
If the button does not respond, update widgets:
```bash
pip install --upgrade ipywidgets jupyterlab_widgets widgetsnbextension
```
Then restart Jupyter completely.
File does not load
Make sure the file contains:
```text
Time calibration: ... ns/ch
```
and a table header:
```text
Chan    Data
```
Fit fails
Try:
Increasing `Fit start ns`
Reducing the fit window
Trying a simpler model
Checking that the uploaded file has valid counts
Making sure the decay peak is present in the trace

License
This project is intended for internal research and educational use only.
---
Shlok Joseph Paul 2026
Developed for TCSPC decay fitting and analysis workflows.
