# BRFSS Diabetes Prediction Project

## Project Overview

This project uses the 2024 Behavioral Risk Factor Surveillance System (BRFSS) survey data to predict the probability of developing different types of diabetes. The BRFSS is an annual telephone survey conducted by the CDC that collects data on U.S. residents regarding health-related risk behaviors, chronic health conditions, and use of preventive services.

## Data Sources

- **Raw Data**: BRFSS 2024 ASCII fixed-width format file
  - URL: https://www.cdc.gov/brfss/annual_data/2024/files/LLCP2024ASC.zip
  - Format: Fixed-width ASCII (2,061 characters per line)
  - Size: ~944 MB (~500,000 rows)
  - File: `LLCP2024.ASC` (excluded from git via .gitignore)

- **Data Dictionary**: HTML codebook with variable definitions
  - URL: https://www.cdc.gov/brfss/annual_data/2024/zip/codebook24_llcp-v2-508.zip
  - Format: HTML table structure with metadata for each variable
  - File: `USCODE24_LLCP_082125.HTML` (excluded from git via .gitignore)

## Project Structure

```
BRFSS-diabetes-2024/
├── parse_raw_data.ipynb    # Data extraction and preprocessing
├── eda_modeling.ipynb      # EDA and modeling (in progress)
├── diabetes_data.pickle    # Cleaned dataset (gitignored)
├── pyproject.toml          # Python project configuration (uv)
├── uv.lock                 # Dependency lock file
├── .python-version         # Python version specification
├── .gitignore             # Excludes large data files
├── README.md              # Project README
└── claude.md              # This file
```

## Data Parsing Architecture

### Hybrid HTML-Driven Approach

The project uses a semi-automated parsing system that combines:

1. **HTML Dictionary Parsing** (BeautifulSoup)
   - Extracts variable metadata from the HTML data dictionary
   - Maps human-readable labels → column positions
   - Returns structured dictionary with: column_range, type, sas_name

2. **Efficient Fixed-Width Reading** (pandas.read_fwf)
   - Reads ASCII file with column specifications
   - Handles 500K+ rows efficiently
   - Low memory footprint, fast performance (~5-10 seconds for 25 columns)

### Key Functions

#### `build_column_lookup(html_file)`
- **Purpose**: Parse HTML data dictionary to extract all variable definitions
- **Input**: Path to HTML codebook file
- **Output**: Dictionary mapping labels to metadata
- **Structure**: `{label: {'column_range': str, 'type': str, 'sas_name': str}}`
- **Regex Patterns**:
  - Label: `r'Label:\s*(.+?)(?:\n|Section)'`
  - Column: `r'Column:\s*(\d+(?:-\d+)?)'`
  - Type: `r'Type of Variable:\s*(Num|Char)'`
  - SAS Name: `r'SAS Variable Name:\s*(\w+)'`

### Adding New Variables

To extract additional columns from the ASCII file:

1. Open `USCODE24_LLCP_082125.HTML` in a browser
2. Search for the variable of interest (e.g., "diabetes", "weight", "age")
3. Copy the exact "Label:" text (e.g., "(Ever told) you had diabetes")
4. Add it to the `columns_to_extract` list in the notebook
5. Run the cells - column positions are extracted automatically

**Example:**
```python
columns_to_extract = [
    "State FIPS Code",
    "Annual Sequence Number",
    "(Ever told) you had diabetes",        # Target variable
    "Reported Weight in Pounds",           # Feature
    "Reported Height in Feet and Inches",  # Feature
    # ... add up to 25 total columns
]
```

## Preprocessing Strategy (eda_modeling.ipynb)

### Missing Value Handling

**Categorical/Ordinal Features:**
- Map NaN and "Refused" → "Unknown"
- Preserves information about non-response patterns
- Applied to 14 categorical + 4 ordinal features

**Numeric Features (Missing Indicator Approach):**
- Create binary indicator columns (*_missing)
- Impute original columns with median
- **Critical for _DRNKWK3**: Distinguishes 0 drinks (valid) from missing data
- Prevents conflating valid zero values with missing responses

### Class Imbalance
- Target distribution: 85.1% negative (no diabetes), 14.9% positive (has diabetes)
- Strategy: Stratified train/val/test splitting (60/20/20)
- Future: Consider SMOTE or class weights during modeling

### Function Architecture
All preprocessing in reusable functions to prevent side effects when re-running cells:
- **`clean_target()`** - Binary encoding with validation and class distribution reporting
- **`clean_features()`** - Unified cleaning for all feature types (categorical, ordinal, numeric)
- **`split_train_val_test()`** - Stratified splitting with optional stratification column

## Current Implementation Status

### parse_raw_data.ipynb - Completed ✅
- Data download functionality (wget)
- HTML data dictionary parsing with value label extraction
- Fixed-width ASCII file parsing (31 columns extracted)
- Decimal transformations (WTKG3, HTM4, _BMI5, _DRNKWK3)
- Special code cleaning and outlier handling
- Comprehensive validation pipeline
- Output: diabetes_data.pickle (457,656 rows × 31 columns)

### eda_modeling.ipynb - In Progress ⏳

**Completed:**
- Data loading and parameter configuration
- Preprocessing functions (clean_target, clean_features, split_train_val_test)
- Target encoding: binary (85.1% negative, 14.9% positive)
- Feature cleaning:
  - 14 categorical + 4 ordinal: NaN/"Refused" → "Unknown"
  - 4 numeric + 4 missing indicators: median imputation
- Dataset ready for EDA

**Next Steps:**
- Exploratory Data Analysis (EDA)
- Feature correlation analysis and selection
- Model training and evaluation
- Docker deployment

## Extracted Variables (31 total)

### Identifiers (2)
- **_STATE**: State FIPS Code
- **SEQNO**: Annual Sequence Number

### Target Variables (3)
- **DIABETE4**: (Ever told) you had diabetes - *Primary target*
- **PREDIAB2**: Ever been told you have pre-diabetes or borderline diabetes
- **DIABTYPE**: What type of diabetes do you have

### Demographic Features (6)
- **_URBSTAT**: Urban/Rural Status
- **_AGEG5YR**: Reported age in five-year age categories (ordinal)
- **SEXVAR**: Sex of Respondent
- **_RACE**: Computed Race-Ethnicity grouping
- **EDUCA**: Education Level (ordinal)
- **INCOME3**: Income Level (ordinal)

### Personal Health Features (13)
- **PERSDOC3**: Have Personal Health Care Provider
- **MEDCOST1**: Could Not Afford To See Doctor
- **WTKG3**: Computed Weight in Kilograms (numeric)
- **HTM4**: Computed Height in Meters (numeric)
- **_BMI5**: Computed body mass index (numeric)
- **EXERANY2**: Exercise in Past 30 Days
- **SSBSUGR2**: How often drink regular soda with sugar (numeric)
- **SSBFRUT3**: How often drink sugar-sweetened drinks (numeric)
- **_SMOKER3**: Computed Smoking Status
- **_DRNKWK3**: Computed number of drinks per week (numeric)
- **DRNKANY6**: Drink any alcoholic beverages in past 30 days
- **_RFDRHV9**: Heavy Alcohol Consumption Calculated Variable
- **GENHLTH**: General Health (ordinal)

### Comorbidity Features (7)
- **CVDINFR4**: Ever Diagnosed with Heart Attack
- **CVDCRHD4**: Ever Diagnosed with Angina or Coronary Heart Disease
- **CVDSTRK3**: Ever Diagnosed with a Stroke
- **CHCKDNY2**: Ever told you have kidney disease
- **ASTHMA3**: Ever Told Had Asthma
- **ADDEPEV3**: (Ever told) you had a depressive disorder
- **HAVARTH4**: Told Had Arthritis

## Data Format Notes

### Column Position Indexing
- **Data Dictionary**: Uses 1-based indexing (columns start at 1)
- **Python**: Uses 0-based indexing (subtract 1 from dictionary values)
- **Conversion**: Handled automatically in the parsing code

### Column Range Formats
- Single column: `149` (just one digit/position)
- Range: `1-2`, `36-45`, `207-210` (hyphen-separated)

### Variable Types
- **Num**: Numeric variables (typically categorical codes or continuous)
- **Char**: Character/string variables (IDs, text responses)

### Line Structure
- Each line is exactly 2,061 characters
- Uses CR LF (Windows-style line endings)
- ASCII encoding
- Fixed positions (no delimiters)

## Development Environment

- **Python Version**: Specified in `.python-version`
- **Package Manager**: uv (see `pyproject.toml`)
- **Key Dependencies**:
  - uv (fast Python package installer and resolver)
  - pandas, numpy (data manipulation)
  - beautifulsoup4 (HTML parsing)
  - scikit-learn (modeling and evaluation)
  - xgboost (gradient boosting)
  - matplotlib, seaborn (visualization)
  - jupyter (notebook environment)

### Running Notebooks

```bash
# Install dependencies with uv
uv sync

# Run Jupyter Lab with uv
uv run --with jupyter jupyter lab
```

## Tips for AI Assistants

1. **Large Files**: Never try to read the entire ASCII file - it's ~944 MB. Use `read_fwf()` with appropriate parameters or read small samples.

2. **HTML Parsing**: The `build_column_lookup()` function already extracts all variables. Don't re-parse unless necessary.

3. **Column Extraction**: Always use the label-based approach. Don't manually calculate column positions.

4. **Performance**: For 500K rows, expect:
   - 2 columns: ~2-3 seconds
   - 25 columns: ~5-10 seconds
   - Manual line-by-line: ~45-60 seconds (avoid!)

5. **Git Workflow**: Data files (.ASC, .HTML, .zip) are gitignored. Only commit code and documentation.

6. **Testing**: When testing parsing code, use `nrows` parameter in `read_fwf()` to limit rows (e.g., `nrows=1000` for testing).

## Useful Commands

### Download data (if not present)
```bash
wget https://www.cdc.gov/brfss/annual_data/2024/files/LLCP2024ASC.zip
unzip LLCP2024ASC.zip
wget https://www.cdc.gov/brfss/annual_data/2024/zip/codebook24_llcp-v2-508.zip
unzip codebook24_llcp-v2-508.zip
```

### Test parsing with small sample
```python
df = pd.read_fwf(
    'LLCP2024.ASC ',
    colspecs=colspecs,
    names=column_names,
    nrows=1000,  # Only read first 1000 rows for testing
    dtype=str,
    encoding='ascii'
)
```

### Find variables in HTML dictionary
```python
# After running build_column_lookup()
for label, info in column_lookup.items():
    if 'diabetes' in label.lower():
        print(f"{label}: {info}")
```

## Project Goals

1. **Primary**: Predict diabetes risk based on BRFSS survey responses
2. **Educational**: Part of Machine Learning Zoomcamp (DataTalksClub)
3. **Technical**: Demonstrate end-to-end ML pipeline from raw data to deployment
4. **Ultimate Goal**: Deploy a working machine learning model using Docker
   - Details of deployment architecture to be determined as project develops
   - Focus on creating a production-ready containerized solution
   - Keep deployment considerations in mind throughout development

## References

- BRFSS 2024 Data: https://www.cdc.gov/brfss/annual_data/annual_2024.html
- CDC BRFSS Overview: https://www.cdc.gov/brfss/
- DataTalksClub ML Zoomcamp: https://github.com/DataTalksClub/machine-learning-zoomcamp

---

*Last Updated: 2025-11-08*
