# SABER_fast Integration Setup Guide

## Overview
This guide shows how to integrate the R-based SABER_fast package with your TriOS AWR processing pipeline for bio-optical parameter inversion.

## Prerequisites

### 1. R Installation and System Dependencies
Ensure R and required system libraries are installed:
```bash
# Ubuntu/Debian - Install R and development libraries
sudo apt-get update
sudo apt-get install -y \
    r-base r-base-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    libfontconfig1-dev \
    libfreetype6-dev \
    libxml2-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libpng-dev \
    libtiff5-dev \
    libjpeg-dev \
    pkg-config

# CentOS/RHEL
sudo yum install R R-devel \
    libcurl-devel \
    openssl-devel \
    fontconfig-devel \
    freetype-devel \
    libxml2-devel \
    harfbuzz-devel \
    fribidi-devel

# Or download R from: https://cran.r-project.org/
```

### 2. Python Dependencies
Install required Python packages:
```bash
pip install -r requirements_saber.txt
```

### 3. R Package Dependencies
The integration module will automatically install required R packages, but you can install them manually if needed:

```r
# In R console:
install.packages(c("devtools", "BayesianTools", "tibble", "dplyr"))
devtools::install_github("homas01123/SABER_fast")
```

## Quick Start

### 1. Basic Integration
```python
from trios.saber_integration import SABERFastIntegrator
from trios.saber_utils import format_results_for_display

# Initialize (will install SABER_fast if needed)
saber = SABERFastIntegrator()

# Process your AWR results  
results = saber.process_awr_methods(
    methods_results,  # From your notebook
    method_name="harmel",
    inversion_type="gradient"
)

# Display results
print(format_results_for_display(results, "harmel"))
```

### 2. Advanced Usage
```python
# Custom parameter configuration
results = saber.invert_gradient(
    rrs_data,
    parameters_to_invert=["chl", "a_g_440", "bb_p_550"],
    fixed_parameters={
        "water_type": 2,
        "theta_sun": 20,  
        "theta_view": 0
    },
    initial_values=[2.0, 0.5, 0.02],
    lower_bounds=[0.1, 0.01, 0.001],
    upper_bounds=[50.0, 5.0, 0.1]
)
```

### 3. MCMC Bayesian Inversion
```python
# For uncertainty quantification
mcmc_results = saber.invert_mcmc(
    rrs_data,
    parameters_to_invert=["chl", "a_g_440", "bb_p_550", "sd"],
    iterations=10000,
    burnin=2000
)
```

## Integration with Existing Workflow

Add this cell to your `testrun_trios.ipynb` notebook after AWR processing:

```python
# SABER_fast Bio-optical Parameter Inversion
try:
    from trios.saber_integration import SABERFastIntegrator
    from trios.saber_utils import format_results_for_display, create_saber_summary_plot
    
    print("Initializing SABER_fast integration...")
    saber = SABERFastIntegrator()
    
    # Use Harmel method (or any other method in methods_results)
    method_name = "harmel"
    
    if method_name in methods_results:
        print(f"Processing {method_name} method for bio-optical inversion...")
        
        # Gradient-based inversion
        results = saber.process_awr_methods(
            methods_results,
            method_name=method_name,
            inversion_type="gradient",
            verbose=True
        )
        
        # Display formatted results
        print(format_results_for_display(results, method_name))
        
        # Store results for further analysis
        gradient_results = results
        
        # Optional: Create summary plots
        exec(create_saber_summary_plot())
        
    else:
        print(f"Method '{method_name}' not found in methods_results")
        
except ImportError:
    print("SABER integration not available. Install requirements_saber.txt")
except Exception as e:
    print(f"SABER integration error: {e}")
```

## Troubleshooting

### Common Issues

1. **rpy2 Installation Problems**
   ```bash
   # Make sure R development headers are installed
   sudo apt-get install r-base-dev
   
   # Reinstall rpy2
   pip uninstall rpy2
   pip install rpy2
   ```

2. **R Package Installation Failures**
   ```bash
   # First install system dependencies (Ubuntu/Debian):
   sudo apt-get install -y \
       libcurl4-openssl-dev \
       libssl-dev \
       libfontconfig1-dev \
       libfreetype6-dev \
       libxml2-dev \
       libwebp-dev \
       pkg-config
   
   # If libwebpmux-dev is unavailable, skip it - not essential for SABER
   ```
   ```r
   # Option 1: Try full devtools installation
   install.packages("devtools", repos="https://cran.rstudio.com/")
   
   # Option 2: If ragg fails, install devtools without heavy dependencies
   install.packages("devtools", dependencies=c("Depends", "Imports"))
   
   # Option 3: Use lightweight remotes package instead
   install.packages("remotes")
   remotes::install_github("homas01123/SABER_fast")
   ```

3. **Memory Issues with Large Datasets**
   - Filter Rrs data to remove outliers before inversion
   - Use gradient inversion instead of MCMC for large datasets
   - Reduce MCMC iterations if needed

### Performance Tips

1. **Data Preprocessing**
   - Remove negative Rrs values before inversion
   - Filter spectral range to 400-800nm for best performance
   - Use common wavelengths across methods

2. **Parameter Bounds**
   - Set realistic bounds based on water type
   - Use initial estimates close to expected values
   - Consider fixing some parameters if known

3. **Method Selection**
   - Use gradient inversion for quick parameter estimates
   - Use MCMC for uncertainty quantification
   - Harmel method generally provides good results for inversion

## Expected Outputs

The integration will provide:

- **Chlorophyll concentration** (chl) in mg/m³
- **CDOM absorption** (a_g_440) at 440nm in m⁻¹  
- **Particulate backscatter** (bb_p_550) at 550nm in m⁻¹
- **Parameter uncertainties** (standard errors)
- **Inversion diagnostics** (convergence, data points used)

## Next Steps

After successful integration:

1. Compare parameter estimates across different AWR methods
2. Validate results against in-situ measurements if available  
3. Use MCMC for full uncertainty analysis
4. Export results for further oceanographic analysis

For questions or issues, refer to:
- SABER_fast documentation: https://github.com/homas01123/SABER_fast
- rpy2 documentation: https://rpy2.github.io/