# Building-a-Retail-Data-Pipeline

This project involves creating a data pipeline for Walmart's e-commerce sales analysis with a focus on holidays and public holidays to assess sales patterns. The goal is to clean and analyze data related to grocery sales and complementary factors (e.g., temperature, fuel prices, CPI, unemployment, etc.) for Walmart stores. The primary aim is to analyze how these variables affect weekly sales during public holidays.

The data sources used in this project are:

grocery_sales table from a PostgreSQL database, which contains weekly sales data by store.
extra_data.parquet file containing additional information like temperature, fuel prices, CPI, and more.
After merging the two datasets, the transformed data is used to generate aggregated monthly sales data.

Steps Taken:
Step 1: Data Extraction
Task: Extract data from two sources: the grocery_sales table (PostgreSQL) and the extra_data.parquet file (Parquet format).
Implementation:
The extract() function reads the Parquet file into a DataFrame (extra_df) and merges it with the grocery_sales DataFrame (store_data) using the common index field.
The result is a merged DataFrame (merged_df) that contains both sales data and the complementary data.
python
Kopiera
Redigera
def extract(store_data, extra_data):
    extra_df = pd.read_parquet(extra_data)
    merged_df = store_data.merge(extra_df, on = "index")
    return merged_df
Step 2: Data Transformation
Task: Clean and transform the merged data to prepare it for analysis.
Implementation:
The transform() function fills missing values in numeric columns (CPI, Weekly_Sales, and Unemployment) with their respective mean values.
The Date column is converted to datetime, and the Month is extracted for further analysis.
The dataset is filtered to only include rows where Weekly_Sales is greater than 10,000 to focus on larger sales.
Unnecessary columns (such as Temperature, Fuel_Price, MarkDown1, MarkDown2, etc.) are dropped to simplify the data.
The result is stored in clean_data.
python
Kopiera
Redigera
def transform(raw_data):
    raw_data.fillna({'CPI': raw_data['CPI'].mean(), 'Weekly_Sales': raw_data['Weekly_Sales'].mean(), 'Unemployment': raw_data['Unemployment'].mean()}, inplace=True)
    raw_data["Date"] = pd.to_datetime(raw_data["Date"], format="%Y-%m-%d")
    raw_data["Month"] = raw_data["Date"].dt.month
    raw_data = raw_data.loc[raw_data["Weekly_Sales"] > 10000, :]
    raw_data = raw_data.drop(["index", "Temperature", "Fuel_Price", "MarkDown1", "MarkDown2", "MarkDown3", "MarkDown4", "MarkDown5", "Type", "Size", "Date"], axis=1)
    return raw_data
Step 3: Analysis
Task: Perform an analysis of monthly sales, calculating the average weekly sales for each month.
Implementation:
The avg_weekly_sales_per_month() function groups the data by Month and calculates the mean of weekly sales for each month.
The result is stored in agg_data, which contains the Month and corresponding Avg_Sales for each month.
python
Kopiera
Redigera
def avg_weekly_sales_per_month(clean_data):
    holidays_sales = clean_data[["Month", "Weekly_Sales"]]
    holidays_sales = (holidays_sales.groupby("Month")
                      .agg(Avg_Sales=("Weekly_Sales", "mean"))
                      .reset_index().round(2))
    return holidays_sales
Step 4: Data Loading (Saving to CSV)
Task: Save the cleaned data (clean_data) and the aggregated monthly data (agg_data) as CSV files.
Implementation:
The load() function is defined to take the cleaned and aggregated data and save them to specified file paths.
python
Kopiera
Redigera
def load(full_data, full_data_file_path, agg_data, agg_data_file_path):
    full_data.to_csv(full_data_file_path, index=False)
    agg_data.to_csv(agg_data_file_path, index=False)
Step 5: Validation
Task: Ensure that the CSV files have been correctly saved and exist at the specified file paths.
Implementation:
The validation() function checks if the files exist at the provided paths using the os.path.exists() method.
If the files do not exist, an exception is raised.
python
Kopiera
Redigera
def validation(file_path):
    file_exists = os.path.exists(file_path)
    if not file_exists:
        raise Exception(f"There is no file at the path {file_path}")
Step 6: Putting it all together
The entire pipeline is executed as follows:
Extract data from the database and the Parquet file.
Transform the data by cleaning it and preparing it for analysis.
Analyze the data to calculate average monthly sales.
Load the cleaned and aggregated data into CSV files.
Validate that the CSV files were created successfully.
python
Kopiera
Redigera
# Call the functions in sequence to complete the pipeline
merged_df = extract(grocery_sales, "extra_data.parquet")
clean_data = transform(merged_df)
agg_data = avg_weekly_sales_per_month(clean_data)
load(clean_data, "clean_data.csv", agg_data, "agg_data.csv")
validation("clean_data.csv")
validation("agg_data.csv")
Conclusion:
The project successfully builds a data pipeline that extracts, cleans, transforms, analyzes, and loads Walmart's sales data. The pipeline is designed to analyze how holidays and other factors influence sales patterns in Walmart stores, with the final goal of storing the clean and aggregated data in CSV files for further analysis.
