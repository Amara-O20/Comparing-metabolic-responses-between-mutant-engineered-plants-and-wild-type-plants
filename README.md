# Comparing-metabolic-responses-between-mutant-engineered-plants-and-wild-type-plants
# Import the libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Step 1: Load the data
df = pd.read_csv('plant_metabolites.csv', index_col=0)

# Step 2: Grab the samples 
# We're comparing WT and Mutants between 24h pesticide treatment and DMSO control

wt_dmso = df.loc['WT_DMSO_1']
wt_24h = df.loc['WT_pesticide_24h_1']
mut_dmso = df.loc['mutant_DMSO_1']
mut_24h = df.loc['mutant_pesticide_24h_1']

# Step 3: Calculate ΔM (difference between 24h and DMSO for each)
delta_wt = wt_24h - wt_dmso
delta_mut = mut_24h - mut_dmso

# Step 4: Scatter plot of ΔM for WT vs. Mutant
plt.figure(figsize=(10, 6))

# x = WT, y = Mutant
x = delta_wt.values
y = delta_mut.values

# Draw the y = x line (slope 1, intercept 0)
plt.plot(x, x, 'k--', label='y = x')

# Step 5: Calculate residuals (difference from y = x)
residuals = y - x
res_cutoff = 0.3  # this is the threshold we're choosing

# Step 6: Color points based on residuals
colors = ['grey' if abs(res) <= res_cutoff else 'salmon' for res in residuals]

# Plot the points
plt.scatter(x, y, c=colors)
plt.xlabel('ΔM WT (24h - DMSO)')
plt.ylabel('ΔM Mutant (24h - DMSO)')
plt.title('Metabolite response: WT vs. Mutant (ΔM)')
plt.grid(True)
plt.legend()

# Step 7: Find and print metabolites that are outliers
metabolites = delta_wt.index
outliers = [met for i, met in enumerate(metabolites) if abs(residuals[i]) > res_cutoff]

print("\nMetabolites outside residual cutoff (+/- 0.3):")
for met in outliers:
    print(met)

plt.show()

# Step 8: Line plots for 6 outlier metabolites
# We'll pick the first 6 just to keep it simple
timepoints = ['DMSO', '8h', '24h']

# We'll extract the values for these timepoints
wt_time = [df.loc['WT_DMSO_1'], df.loc['WT_pesticide_8h_1'], df.loc['WT_pesticide_24h_1']]
mut_time = [df.loc['mutant_DMSO_1'], df.loc['mutant_pesticide_8h_1'], df.loc['mutant_pesticide_24h_1']]

# Pick 6 metabolites to plot
selected_outliers = outliers[:6]

# Plotting them
fig, axs = plt.subplots(2, 3, figsize=(15, 8))
axs = axs.flatten()

for i, met in enumerate(selected_outliers):
    wt_vals = [t[met] for t in wt_time]
    mut_vals = [t[met] for t in mut_time]
    
    axs[i].plot(timepoints, wt_vals, marker='o', label='WT')
    axs[i].plot(timepoints, mut_vals, marker='s', label='Mutant')
    axs[i].set_title(met)
    axs[i].legend()
    axs[i].grid(True)

plt.suptitle("Time-course of selected outlier metabolites", fontsize=14)
plt.tight_layout()
plt.show()
