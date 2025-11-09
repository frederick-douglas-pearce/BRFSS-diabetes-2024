# BRFSS Diabetes Risk Prediction

This project applies machine learning to predict diabetes risk based on the Behavioral Risk Factor Surveillance System (BRFSS) survey data for 2024, and serves as the midterm project for the DataTalksClub Machine-Learning-Zoomcamp course.

## Project Overview

The BRFSS is an annual telephone survey conducted by the CDC that collects data on U.S. residents regarding health-related risk behaviors, chronic health conditions, and use of preventive services. This project uses the 2024 survey data to build predictive models for diabetes risk assessment.

## Data Sources

- **Raw Data**: BRFSS 2024 ASCII fixed-width format file
  - Source: [CDC BRFSS 2024 Data](https://www.cdc.gov/brfss/annual_data/2024/files/LLCP2024ASC.zip)
  - Format: Fixed-width ASCII (~944 MB, ~458,000 rows, 2,061 characters per line)

- **Data Dictionary**: HTML codebook with variable definitions
  - Source: [BRFSS 2024 Codebook](https://www.cdc.gov/brfss/annual_data/2024/zip/codebook24_llcp-v2-508.zip)
  - Format: HTML table structure with metadata and value labels

## Project Structure

```
BRFSS-diabetes-2024/
├── parse_raw_data.ipynb    # Data extraction and preprocessing
├── eda_modeling.ipynb      # EDA and modeling (in progress)
├── README.md               # This file
├── claude.md              # Detailed technical documentation
├── .gitignore             # Excludes large data files
└── diabetes_data.pickle   # Cleaned dataset output (gitignored)
```

## Notebooks

### 1. parse_raw_data.ipynb (Completed)

**Purpose**: Extract, transform, and clean raw BRFSS data for analysis.

**Key Features**:
- Automated data download from CDC sources
- HTML data dictionary parsing using BeautifulSoup
- Fixed-width ASCII file parsing (31 selected variables)
- Decimal place transformations for numeric variables
- Special code handling and outlier capping
- Data validation (grain checks, missing values, data types, range checks)
- Out-of-range record removal
- Export to pickle format

**Variables Extracted**:
- **Identifiers**: State FIPS Code, Annual Sequence Number
- **Target Variables**: Diabetes status, pre-diabetes, diabetes type
- **Demographics**: Age, sex, race, education, income, urban/rural status
- **Health Metrics**: Weight, height, BMI, general health status
- **Behavioral Factors**: Exercise, smoking status, alcohol consumption, sugar-sweetened beverage intake
- **Comorbidities**: Heart attack, heart disease, stroke, kidney disease, asthma, depression, arthritis
- **Healthcare Access**: Has healthcare provider, affordability of care

**Output**: `diabetes_data.pickle` - cleaned dataset with 457,656 rows and 31 columns

**Technical Highlights**:
- Well-documented functions with comprehensive docstrings
- Reusable code structure for easy modification
- Validation pipeline ensuring data quality
- Minimal cell output for clean notebook flow

### 2. eda_modeling.ipynb (In Progress)

**Purpose**: Perform exploratory data analysis and develop predictive models for diabetes risk.

**Completed**:
- Data loading from pickle file
- Parameter configuration (target, features categorized as categorical/ordinal/numeric)
- Preprocessing functions:
  - `clean_target()` - Binary encoding with class distribution (85.1% negative, 14.9% positive)
  - `clean_features()` - Unified cleaning for all feature types:
    - Categorical/ordinal: NaN and "Refused" → "Unknown"
    - Numeric: Missing indicators + median imputation (preserves distinction between 0 and missing)
  - `split_train_val_test()` - Stratified data splitting (60/20/20)
- Feature preparation:
  - 14 categorical features (disease indicators, demographics, behaviors)
  - 4 ordinal features (age, education, income, general health)
  - 4 numeric features + 4 missing indicators (WTKG3, HTM4, _BMI5, _DRNKWK3)
- Ready for EDA phase with cleaned dataset

**In Progress**:
- Exploratory Data Analysis (EDA)
- Feature correlation analysis (especially WTKG3/HTM4/_BMI5)

**Planned**:
- Feature Engineering and Selection
- Model Development (logistic regression, random forest, XGBoost)
- Hyperparameter tuning and cross-validation
- Model Evaluation and selection
- Docker deployment of final model

## Getting Started

### Prerequisites

- Python 3.x
- Jupyter Notebook
- Required packages: pandas, numpy, beautifulsoup4, pickle

### Installation

1. Clone the repository:
```bash
git clone https://github.com/frederick-douglas-pearce/BRFSS-diabetes-2024.git
cd BRFSS-diabetes-2024
```

2. Install dependencies (using uv or pip):
```bash
uv sync
# or
pip install pandas numpy beautifulsoup4
```

3. Run the data parsing notebook:
```bash
jupyter notebook parse_raw_data.ipynb
```

The notebook will automatically download the required data files (~945 MB total) on first run.

## Data Files (Excluded from Git)

The following files are automatically downloaded but gitignored due to size:
- `LLCP2024.ASC` - Raw ASCII data file
- `USCODE24_LLCP_*.HTML` - Data dictionary
- `*.zip` - Downloaded archives
- `diabetes_data.pickle` - Processed output dataset

## Documentation

See [claude.md](claude.md) for detailed technical documentation including:
- Data format specifications
- Parsing methodology
- Transformation details
- Variable definitions

## References

- [BRFSS 2024 Data](https://www.cdc.gov/brfss/annual_data/annual_2024.html)
- [CDC BRFSS Overview](https://www.cdc.gov/brfss/)
- [DataTalksClub ML Zoomcamp](https://github.com/DataTalksClub/machine-learning-zoomcamp)

## License

This project is for educational purposes as part of the DataTalksClub Machine Learning Zoomcamp.

## Acknowledgments

- CDC BRFSS for providing comprehensive public health surveillance data
- DataTalksClub for the Machine Learning Zoomcamp course
- Claude Code for development assistance
