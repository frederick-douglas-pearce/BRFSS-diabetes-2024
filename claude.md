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
├── parse_raw_data.ipynb    # Main notebook for data parsing
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

## Current Implementation Status

### Completed
- ✅ Data download functionality (wget)
- ✅ HTML data dictionary parsing
- ✅ Fixed-width ASCII file parsing
- ✅ Label-based column extraction
- ✅ Validation and preview output
- ✅ Initial 2 columns: State FIPS Code, Annual Sequence Number

### Planned
- ⏳ Identify and extract ~25 relevant features
- ⏳ Identify diabetes target variable(s)
- ⏳ Exploratory Data Analysis (EDA)
- ⏳ Feature engineering
- ⏳ Model training and evaluation
- ⏳ Model deployment

## Important Column Information

### Currently Extracted

| Label | SAS Name | Column Range | Type | Description |
|-------|----------|--------------|------|-------------|
| State FIPS Code | _STATE | 1-2 | Num | State identifier |
| Annual Sequence Number | SEQNO | 36-45 | Char | Unique record ID |

### Key Variables to Consider

**Target Variables:**
- `(Ever told) you had diabetes` - Column 149 (DIABETE4)
- `Age When First Told You Had Diabetes` - Columns 150-151

**Demographic Features:**
- Sex of Respondent
- Age categories
- Income level
- Education level
- Race/ethnicity

**Health-Related Features:**
- General Health Status
- Weight and Height (BMI calculation)
- Exercise/physical activity
- Smoking status
- Alcohol consumption

**Health Conditions:**
- High blood pressure
- High cholesterol
- Heart disease
- Stroke history

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
  - pandas (data manipulation)
  - beautifulsoup4 (HTML parsing)
  - jupyter (notebook environment)

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

## References

- BRFSS 2024 Data: https://www.cdc.gov/brfss/annual_data/annual_2024.html
- CDC BRFSS Overview: https://www.cdc.gov/brfss/
- DataTalksClub ML Zoomcamp: https://github.com/DataTalksClub/machine-learning-zoomcamp

---

*Last Updated: 2025-11-05*
