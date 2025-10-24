# Membrane-Materials-Optimization-MD
Multi-criteria materials selection framework for membrane distillation applications integrating ANSYS GRANTA materials informatics with life cycle assessment. Systematic evaluation of 22 polymeric, bioplastic, and ceramic materials across thermal, mechanical, transport, environmental, and economic performance dimensions.
"""
Performance Index Calculator for Membrane Materials
Calculates multi-criteria performance index for membrane distillation material selection
Based on ANSYS GRANTA data analysis
"""

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# ============================================================================
# STEP 1: INPUT YOUR DATA
# ============================================================================

# Create dataframe with your 22 materials and their properties
# Replace these values with your actual GRANTA data
data = {
    'Material': [
        'PE', 'PVC', 'PP', 'PS', 'ABS', 'SAN', 'PET', 'PLA', 'CN', 'CA',
        'PA12', 'PVDF', 'PTFE', 'PSU', 'PES', 'PEEK', 'ECTFE', 'PI',
        'SBS', 'Nylon', 'Alumina', 'Zirconia', 'Titania'
    ],
    
    # Maximum service temperature (°C)
    'MaxTemp': [
        80, 60, 130, 95, 90, 85, 115, 55, 80, 75,
        100, 150, 260, 180, 200, 250, 150, 280,
        70, 120, 1800, 2400, 1600
    ],
    
    # Tensile strength (MPa)
    'TensileStrength': [
        25, 45, 35, 45, 45, 60, 55, 60, 45, 50,
        40, 45, 35, 70, 85, 105, 45, 75,
        7, 70, 250, 200, 240
    ],
    
    # Water vapor transmission rate (g·mm·m⁻²·day⁻¹)
    'WVTR': [
        0.15, 0.5, 0.2, 3.5, 2.0, 3.0, 0.6, 5.0, 0.8, 0.5,
        0.3, 1.2, 0.2, 5.0, 10.0, 0.15, 3.0, 0.25,
        200, 0.3, 0.15, 0.5, 0.3
    ],
    
    # Embodied energy (MJ/year) - for 10,000 m³/day facility
    'EmbodiedEnergy': [
        2.7e6, 2.0e6, 2.2e6, 2.3e6, 2.7e6, 3.2e6, 3.5e6, 1.6e6, 0.35e6, 2.4e6,
        2.1e6, 5.4e6, 5.8e6, 5.4e6, 8.1e6, 8.5e6, 3.5e6, 9.0e6,
        2.9e6, 3.5e6, 1.4e6, 1.6e6, 1.6e6
    ],
    
    # Material class for visualization
    'Class': [
        'Plastic', 'Plastic', 'Plastic', 'Plastic', 'Plastic', 'Plastic', 
        'Plastic', 'Bioplastic', 'Bioplastic', 'Bioplastic',
        'Plastic', 'Plastic', 'Plastic', 'Plastic', 'Plastic', 'Plastic', 
        'Plastic', 'Plastic', 'Plastic', 'Plastic',
        'Ceramic', 'Ceramic', 'Ceramic'
    ]
}

df = pd.DataFrame(data)

# ============================================================================
# STEP 2: DEFINE NORMALIZATION FUNCTION
# ============================================================================

def normalize_property(values, property_type='beneficial'):
    """
    Normalize property values to 0-1 scale using min-max normalization
    
    Parameters:
    -----------
    values : array-like
        Property values to normalize
    property_type : str
        'beneficial' if higher values are better (e.g., temperature, strength)
        'detrimental' if lower values are better (e.g., cost, energy)
    
    Returns:
    --------
    normalized : array
        Normalized values between 0 and 1
    """
    min_val = np.min(values)
    max_val = np.max(values)
    
    if property_type == 'beneficial':
        # Higher values get higher scores
        normalized = (values - min_val) / (max_val - min_val)
    else:  # detrimental
        # Lower values get higher scores (inverted)
        normalized = (max_val - values) / (max_val - min_val)
    
    return normalized

# ============================================================================
# STEP 3: NORMALIZE ALL PROPERTIES
# ============================================================================

print("="*70)
print("PERFORMANCE INDEX CALCULATION FOR MEMBRANE MATERIALS")
print("="*70)
print("\nStep 1: Normalizing properties to 0-1 scale...")

# Normalize beneficial properties (higher = better)
df['Norm_MaxTemp'] = normalize_property(df['MaxTemp'], 'beneficial')
df['Norm_TensileStrength'] = normalize_property(df['TensileStrength'], 'beneficial')
df['Norm_WVTR'] = normalize_property(df['WVTR'], 'beneficial')

# Normalize detrimental properties (lower = better)
df['Norm_EmbodiedEnergy'] = normalize_property(df['EmbodiedEnergy'], 'detrimental')

# Display normalization ranges
print(f"\nProperty Ranges:")
print(f"  Max Temperature:    {df['MaxTemp'].min():.0f} - {df['MaxTemp'].max():.0f} °C")
print(f"  Tensile Strength:   {df['TensileStrength'].min():.0f} - {df['TensileStrength'].max():.0f} MPa")
print(f"  WVTR:              {df['WVTR'].min():.2f} - {df['WVTR'].max():.2f} g·mm·m⁻²·day⁻¹")
print(f"  Embodied Energy:    {df['EmbodiedEnergy'].min()/1e6:.2f} - {df['EmbodiedEnergy'].max()/1e6:.2f} MJ/year (×10⁶)")

# ============================================================================
# STEP 4: CALCULATE PERFORMANCE INDEX FOR DIFFERENT SCENARIOS
# ============================================================================

print("\n" + "="*70)
print("Step 2: Calculating Performance Indices for different scenarios...")
print("="*70)

# Scenario 1: Equal weighting (standard desalination)
w1_equal = {'temp': 0.25, 'tensile': 0.25, 'wvtr': 0.25, 'energy': 0.25}
df['PI_Equal'] = (
    w1_equal['temp'] * df['Norm_MaxTemp'] +
    w1_equal['tensile'] * df['Norm_TensileStrength'] +
    w1_equal['wvtr'] * df['Norm_WVTR'] +
    w1_equal['energy'] * df['Norm_EmbodiedEnergy']
)

# Scenario 2: High Temperature Priority (harsh chemical environment)
w2_hightemp = {'temp': 0.35, 'tensile': 0.30, 'wvtr': 0.20, 'energy': 0.15}
df['PI_HighTemp'] = (
    w2_hightemp['temp'] * df['Norm_MaxTemp'] +
    w2_hightemp['tensile'] * df['Norm_TensileStrength'] +
    w2_hightemp['wvtr'] * df['Norm_WVTR'] +
    w2_hightemp['energy'] * df['Norm_EmbodiedEnergy']
)

# Scenario 3: High Flux Priority (productivity-focused)
w3_highflux = {'temp': 0.20, 'tensile': 0.20, 'wvtr': 0.40, 'energy': 0.20}
df['PI_HighFlux'] = (
    w3_highflux['temp'] * df['Norm_MaxTemp'] +
    w3_highflux['tensile'] * df['Norm_TensileStrength'] +
    w3_highflux['wvtr'] * df['Norm_WVTR'] +
    w3_highflux['energy'] * df['Norm_EmbodiedEnergy']
)

# Scenario 4: Sustainability Priority (environmental focus)
w4_sustainable = {'temp': 0.20, 'tensile': 0.20, 'wvtr': 0.25, 'energy': 0.35}
df['PI_Sustainable'] = (
    w4_sustainable['temp'] * df['Norm_MaxTemp'] +
    w4_sustainable['tensile'] * df['Norm_TensileStrength'] +
    w4_sustainable['wvtr'] * df['Norm_WVTR'] +
    w4_sustainable['energy'] * df['Norm_EmbodiedEnergy']
)

# ============================================================================
# STEP 5: RANK MATERIALS FOR EACH SCENARIO
# ============================================================================

df['Rank_Equal'] = df['PI_Equal'].rank(ascending=False).astype(int)
df['Rank_HighTemp'] = df['PI_HighTemp'].rank(ascending=False).astype(int)
df['Rank_HighFlux'] = df['PI_HighFlux'].rank(ascending=False).astype(int)
df['Rank_Sustainable'] = df['PI_Sustainable'].rank(ascending=False).astype(int)

# ============================================================================
# STEP 6: DISPLAY RESULTS
# ============================================================================

print("\n" + "="*70)
print("TOP 10 MATERIALS - EQUAL WEIGHTING SCENARIO")
print("="*70)
top10_equal = df.nsmallest(10, 'Rank_Equal')[['Material', 'Class', 'PI_Equal', 'Rank_Equal']]
print(top10_equal.to_string(index=False))

print("\n" + "="*70)
print("TOP 10 MATERIALS - HIGH TEMPERATURE PRIORITY")
print("="*70)
top10_hightemp = df.nsmallest(10, 'Rank_HighTemp')[['Material', 'Class', 'PI_HighTemp', 'Rank_HighTemp']]
print(top10_hightemp.to_string(index=False))

print("\n" + "="*70)
print("TOP 10 MATERIALS - HIGH FLUX PRIORITY")
print("="*70)
top10_highflux = df.nsmallest(10, 'Rank_HighFlux')[['Material', 'Class', 'PI_HighFlux', 'Rank_HighFlux']]
print(top10_highflux.to_string(index=False))

print("\n" + "="*70)
print("TOP 10 MATERIALS - SUSTAINABILITY PRIORITY")
print("="*70)
top10_sustainable = df.nsmallest(10, 'Rank_Sustainable')[['Material', 'Class', 'PI_Sustainable', 'Rank_Sustainable']]
print(top10_sustainable.to_string(index=False))

# ============================================================================
# STEP 7: SAVE RESULTS TO CSV
# ============================================================================

output_columns = [
    'Material', 'Class', 'MaxTemp', 'TensileStrength', 'WVTR', 'EmbodiedEnergy',
    'Norm_MaxTemp', 'Norm_TensileStrength', 'Norm_WVTR', 'Norm_EmbodiedEnergy',
    'PI_Equal', 'Rank_Equal', 'PI_HighTemp', 'Rank_HighTemp',
    'PI_HighFlux', 'Rank_HighFlux', 'PI_Sustainable', 'Rank_Sustainable'
]

df_output = df[output_columns].sort_values('Rank_Equal')
df_output.to_csv('membrane_performance_index_results.csv', index=False)
print("\n✓ Results saved to: membrane_performance_index_results.csv")

# ============================================================================
# STEP 8: CREATE VISUALIZATIONS
# ============================================================================

print("\nGenerating visualizations...")

# Set style
sns.set_style("whitegrid")
plt.rcParams['figure.figsize'] = (14, 10)

# Figure 1: Performance Index Comparison Across Scenarios
fig, axes = plt.subplots(2, 2, figsize=(16, 12))

scenarios = [
    ('PI_Equal', 'Equal Weighting', axes[0, 0]),
    ('PI_HighTemp', 'High Temperature Priority', axes[0, 1]),
    ('PI_HighFlux', 'High Flux Priority', axes[1, 0]),
    ('PI_Sustainable', 'Sustainability Priority', axes[1, 1])
]

for pi_col, title, ax in scenarios:
    top10 = df.nlargest(10, pi_col).sort_values(pi_col)
    colors = ['red' if c == 'Ceramic' else 'magenta' if c == 'Bioplastic' else 'blue' 
              for c in top10['Class']]
    ax.barh(top10['Material'], top10[pi_col], color=colors, alpha=0.7)
    ax.set_xlabel('Performance Index', fontsize=12)
    ax.set_title(title, fontsize=14, fontweight='bold')
    ax.grid(axis='x', alpha=0.3)

plt.tight_layout()
plt.savefig('performance_index_comparison.png', dpi=300, bbox_inches='tight')
print("✓ Saved: performance_index_comparison.png")

# Figure 2: Ranking Stability Analysis
fig, ax = plt.subplots(figsize=(14, 8))

materials_to_plot = df.nsmallest(15, 'Rank_Equal')['Material'].values

for material in materials_to_plot:
    ranks = df[df['Material'] == material][
        ['Rank_Equal', 'Rank_HighTemp', 'Rank_HighFlux', 'Rank_Sustainable']
    ].values[0]
    
    material_class = df[df['Material'] == material]['Class'].values[0]
    color = 'red' if material_class == 'Ceramic' else 'magenta' if material_class == 'Bioplastic' else 'blue'
    
    ax.plot([1, 2, 3, 4], ranks, marker='o', label=material, color=color, alpha=0.7, linewidth=2)

ax.set_xticks([1, 2, 3, 4])
ax.set_xticklabels(['Equal', 'High Temp', 'High Flux', 'Sustainable'], fontsize=11)
ax.set_ylabel('Rank', fontsize=12)
ax.set_xlabel('Scenario', fontsize=12)
ax.set_title('Material Ranking Stability Across Application Scenarios', fontsize=14, fontweight='bold')
ax.invert_yaxis()  # Lower rank number = better
ax.grid(alpha=0.3)
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize=9)

plt.tight_layout()
plt.savefig('ranking_stability_analysis.png', dpi=300, bbox_inches='tight')
print("✓ Saved: ranking_stability_analysis.png")

# Figure 3: Normalized Properties Heatmap
fig, ax = plt.subplots(figsize=(12, 10))

top15 = df.nlargest(15, 'PI_Equal').sort_values('PI_Equal', ascending=False)
heatmap_data = top15[['Norm_MaxTemp', 'Norm_TensileStrength', 'Norm_WVTR', 'Norm_EmbodiedEnergy']].T

sns.heatmap(heatmap_data, 
            xticklabels=top15['Material'],
            yticklabels=['Thermal\nStability', 'Mechanical\nStrength', 'Vapor\nTransport', 'Environmental\nSustainability'],
            cmap='RdYlGn', 
            annot=True, 
            fmt='.2f',
            cbar_kws={'label': 'Normalized Score'},
            ax=ax)

ax.set_title('Normalized Property Performance - Top 15 Materials', fontsize=14, fontweight='bold')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.savefig('normalized_properties_heatmap.png', dpi=300, bbox_inches='tight')
print("✓ Saved: normalized_properties_heatmap.png")

print("\n" + "="*70)
print("ANALYSIS COMPLETE!")
print("="*70)
print("\nGenerated files:")
print("  1. membrane_performance_index_results.csv")
print("  2. performance_index_comparison.png")
print("  3. ranking_stability_analysis.png")
print("  4. normalized_properties_heatmap.png")
print("\n" + "="*70)
