
# DV360 MTM Data Processing Architecture

```mermaid
graph TD
    A["📁 DV360 Raw CSV Files<br/>(12 Monthly Files)<br/>MX_CCTM_MTM_Pull_*.csv"]
    
    B["⚙️ Configuration & Setup<br/>- Column definitions<br/>- Groupby keys<br/>- Aggregation specs"]
    
    C["🔍 Discover Files<br/>& Peek Header"]
    
    D["📋 Define Column Sets<br/>- CATEGORICAL_COLS<br/>- AVG_COLS SUM_COLS<br/>Build agg_spec"]
    
    E["📥 Per-File Processing Loop"]
    E1["✅ Checkpoint Exists?"]
    E2["💾 Load from Cache"]
    E3["📖 Fresh Load"]
    E4["🧹 Clean & Parse Dates<br/>Strip footer rows<br/>Convert to numeric"]
    E5["📊 Aggregate by<br/>[Date, IO, Creative]<br/>SUM + MEAN"]
    E6["💾 Save Checkpoint<br/>CSV per file"]
    
    F["➕ Concatenate<br/>Aggregated Frames"]
    
    G["🔧 Post-Concat Cleanup<br/>- Drop all-zero columns<br/>- Parse dates<br/>- Rename columns"]
    
    H["📊 TCC Attention Data<br/>Load from Excel<br/>TCCC Metrics.xlsx"]
    
    I["🔎 TCC Filter & Process<br/>- Year = 2025<br/>- dv360 + Display only<br/>- Select key columns"]
    
    J["📈 TCC Aggregation<br/>Group by<br/>[Date, IO, Placement]<br/>Calculate Average AU"]
    
    K["🔗 Merge KPI ⨝ Attention<br/>Inner join on<br/>[Date, IO, Placement]"]
    
    L["❌ Post-Merge Filter<br/>Remove zero-AU<br/>placement groups"]
    
    M["📤 Export Results"]
    M1["📄 Merged CSV<br/>df_merged_raw_file_ex_dt.csv"]
    M2["📊 MTM Summary<br/>mtm_raw_file_dv360.csv<br/>mtm_raw_file_dv360.xlsx"]
    M3["✨ Final Output Excel"]
    
    Z["✅ MTM Input Ready"]
    
    A --> C
    B --> D
    C --> D
    D --> E
    
    E --> E1
    E1 -->|Yes| E2
    E1 -->|No| E3
    E2 --> F
    E3 --> E4
    E4 --> E5
    E5 --> E6
    E6 --> F
    
    F --> G
    
    H --> I
    I --> J
    
    G --> K
    J --> K
    
    K --> L
    L --> M
    
    M --> M1
    M --> M2
    M --> M3
    
    M1 --> Z
    M2 --> Z
    M3 --> Z
    
    style A fill:#e1f5ff
    style H fill:#e1f5ff
    style E fill:#fff3e0
    style F fill:#fff3e0
    style K fill:#f3e5f5
    style L fill:#f3e5f5
    style M fill:#e8f5e9
    style Z fill:#c8e6c9
```

## Pipeline Stages

### 1. **Data Discovery** (Blue)
- Load 12 monthly DV360 CSV files
- Peek at schema to identify available columns

### 2. **Per-File Processing** (Orange)
- **Checkpoint optimization**: Cache each aggregated file to enable restarts
- **Memory-efficient loading**: Parse only required columns (`usecols`)
- **Data cleaning**: Strip footers, parse mixed date formats, normalize device types
- **Aggregation**: Group by `[Date, IO ID, Creative]` and apply SUM/MEAN

### 3. **TCC Attention Data** (Blue)
- Load Excel workbook with AU (Attention Units) metrics
- Filter for 2025, dv360, Display channel
- Aggregate and calculate Average AU

### 4. **Merge & Filter** (Purple)
- Inner join on `[Date, IO ID, Placement]`
- Remove placements with zero attention signal
- Drop all-zero columns

### 5. **Export** (Green)
- Generate merged dataset CSV
- Create MTM input summary (CSV + XLSX)
- Ready for downstream model consumption

## Key Optimizations

- **Per-file aggregation** → Peak memory bounded to single file
- **Checkpoint resumption** → Skip already-processed files on reruns
- **Column filtering** → Only parse needed columns from disk
- **Pre-aggregation concat** → Concatenate small pre-aggregated frames instead of massive raw dataset
