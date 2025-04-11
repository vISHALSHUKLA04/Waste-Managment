# Waste-Managment
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Load data with proper type handling
try:
    df = pd.read_csv('dataset.csv', 
                    dtype={
                        'Solid Waste Registration Number': str,
                        'WMU Sequence Number': str,
                        'System Type Code': str,
                        'Texas Waste Code': str,
                        'Origin Code': str,
                        'Source Code': str,
                        'Primary NAICS Code': str
                    },
                    low_memory=False)
    
    print(" Dataset loaded successfully")
    print(f" Total records: {len(df):,}")
except Exception as e:
    print(f"Error loading file: {e}")
    exit()

# 1. Dataset Overview
print("\n=== DATASET OVERVIEW ===")
print(df.info())
print("\nMissing values per column:")
print(df.isnull().sum())

# 2. Top Waste Codes Analysis (FIXED palette warning)
plt.figure(figsize=(12, 6))
top_codes = df['Texas Waste Code'].value_counts().head(10)
sns.barplot(x=top_codes.index, 
            y=top_codes.values, 
            hue=top_codes.index,  # Added to fix warning
            palette="viridis",
            legend=False)         # Added to fix warning
plt.title('Top 10 Most Frequent Waste Codes', fontsize=14)
plt.xlabel('Waste Code', fontsize=12)
plt.ylabel('Count', fontsize=12)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# 3. Waste Status Distribution
plt.figure(figsize=(8, 8))
status_counts = df['WMU_REGIS_STATUS_CD'].value_counts()
plt.pie(status_counts, 
       labels=status_counts.index, 
       autopct='%1.1f%%',
       colors=sns.color_palette('pastel'),
       startangle=90,
       wedgeprops={'edgecolor': 'white', 'linewidth': 1})
plt.title('Waste Registration Status', fontsize=14)
plt.tight_layout()
plt.show()

# 4. Temporal Analysis (if date column exists)
if 'Waste Code Status Change Date' in df.columns:
    df['Year'] = pd.to_datetime(
        df['Waste Code Status Change Date'], 
        errors='coerce'
    ).dt.year
    
    plt.figure(figsize=(12, 6))
    yearly_data = df['Year'].value_counts().sort_index()
    sns.lineplot(x=yearly_data.index, y=yearly_data.values, 
                marker='o', color='royalblue', linewidth=2.5)
    plt.title('Waste Records by Year', fontsize=14)
    plt.xlabel('Year', fontsize=12)
    plt.ylabel('Number of Records', fontsize=12)
    plt.grid(True, linestyle='--', alpha=0.7)
    plt.tight_layout()
    plt.show()

# 5. Industry Analysis (NAICS Codes) (FIXED palette warning)
plt.figure(figsize=(12, 8))
naics_counts = df['Primary NAICS Code'].value_counts().head(15)
sns.barplot(y=naics_counts.index.astype(str), 
           x=naics_counts.values,
           hue=naics_counts.index.astype(str),  # Added to fix warning
           palette="rocket_r",
           legend=False)                       # Added to fix warning
plt.title('Top 15 Industries by NAICS Code', fontsize=14)
plt.xlabel('Count', fontsize=12)
plt.ylabel('Industry Code', fontsize=12)
plt.tight_layout()
plt.show()

# 6. Hazardous Waste Analysis
hazardous = df[(df['Radioactive Flag'] == 'true') | 
              (df['New Chemical Substance Flag'] == '1')]
print(f"\n Found {len(hazardous)} hazardous waste records")

# 7. Correlation Heatmap (for numeric columns)
numeric_cols = df.select_dtypes(include=['int64', 'float64']).columns
if len(numeric_cols) > 1:
    plt.figure(figsize=(10, 8))
    sns.heatmap(df[numeric_cols].corr(), 
               annot=True, 
               cmap='coolwarm',
               center=0,
               fmt=".2f",
               linewidths=.5)
    plt.title('Correlation Between Numeric Features', fontsize=14)
    plt.tight_layout()
    plt.show()
else:
    print("\nNot enough numeric columns for correlation heatmap")

print("\n=== ANALYSIS COMPLETE ===")
