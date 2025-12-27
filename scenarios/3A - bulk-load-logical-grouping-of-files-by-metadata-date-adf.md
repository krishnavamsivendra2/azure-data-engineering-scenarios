# Bulk Load: Logical Grouping of Files Based on Metadata Date using Azure Data Factory

## 1. Scenario / Problem Statement
At the time of initial pipeline setup, the source **Azure Blob Storage** container already contains a large number of historical files stored in an unorganized manner.  
The requirement is to perform a **one-time bulk load** to logically group these files into **date-based folders (`yyyy/MM/dd`)** in **ADLS Gen2**, using the **file metadata date**.

---

## 2. Business Requirement
- Handle initial historical data load
- Read multiple files present in the source container
- Extract date information from file metadata
- Organize files into date-based folders in ADLS Gen2
- Move files after successful copy

---

## 3. Assumptions
- This is a **one-time or limited-run** bulk load
- Date is derived from **file metadata** (LastModified)
- Source storage is **Azure Blob Storage**
- Destination storage is **ADLS Gen2**
- Azure Data Factory has required read/write/delete permissions
- Folder structure follows `yyyy/MM/dd`

---

## 4. Technology Stack
- **Azure Data Factory**
- **Azure Blob Storage**
- **Azure Data Lake Storage Gen2**
- Schedule or manual trigger
- Parameterized datasets

---

## 5. High-Level Architecture
1. Pipeline is triggered manually or via schedule
2. List of files is fetched from source container
3. Files are iterated using ForEach
4. Metadata date is extracted for each file
5. Destination folder path is generated dynamically
6. Files are copied to ADLS Gen2
7. Source files are deleted after copy

---

## 6. Detailed Solution Design

### Step 1: Create Pipeline Variables

Create the following pipeline-level variables to support metadata-based date extraction and dynamic folder creation:

| Variable Name 		| Data Type | 					Description			   		  |
|-------------------------------|-----------|-------------------------------------------------------------------------------------|
| `year` 			| String    | Stores the year extracted from file metadata (`LastModified`) 			  |
| `month` 			| String    | Stores the month extracted from file metadata 					  |
| `day` 			| String    | Stores the day extracted from file metadata 					  |
| `destination_folder_name`     | String    | Stores the dynamically generated destination folder path in the format `yyyy/MM/dd` |

> ℹ️ **Note:**  
> All variables are initialized without default values and are populated dynamically during pipeline execution using **Set Variable** activities.


### Step 2: Pipeline Trigger
- Use **Manual** or **Schedule Trigger**
- Suitable for bulk historical processing
- Pipeline processes multiple files in a single run

---

### Step 3: Fetch List of Files
- Use **Get Metadata** activity
- Linked Service: Non-parameterized (source container)
- Field selected:
  - `Child Items`
- Output contains list of all files in the container

---

### Step 4: Iterate Through Files
- Use **ForEach** activity
- Input:
	`@activity('GetMetadata_Container').output.childItems`
- Each iteration processes one file

---

### Step 5: Fetch Metadata for Each File
	#### (Inside ForEach)

- Use **Get Metadata** activity
- Dataset is parameterized using: `item().name`
- Retrieve:
	- `LastModified`
	- `ItemName`

---

### Step 6: Extract Date from Metadata 
	#### (Inside ForEach)

- Use **Set Variable** activities to extract:
  - `year`
  - `month`
  - `day`
- Date is derived from `LastModified` metadata

ADF expressions used:

```text
@formatDateTime(activity('GetMetadata_File').output.lastModified, 'yyyy')
@formatDateTime(activity('GetMetadata_File').output.lastModified, 'MM')
@formatDateTime(activity('GetMetadata_File').output.lastModified, 'dd')
```
---

### Step 7: Construct Destination Folder Path
	#### (Inside ForEach)

- Build folder path dynamically using extracted date variables

ADF expression used:

```text
@concat(variables('year'),'/',variables('month'),'/',variables('day'))
```
- The generated value is assigned using a **Set Variable** activity.

---

### Step 8: Copy File to ADLS Gen2
	#### (Inside ForEach)
- Use **Copy Data** activity
- Source:
  - Azure Blob Storage (current file)
- Sink:
  - ADLS Gen2
  - Folder path set dynamically using `destination_folder_name`

📌 Note:
- ADLS Gen2 automatically creates folders if they do not exist
- Existing folders are reused

---

### Step 9: Delete Source File
	#### (Inside ForEach)
- Use **Delete** activity
- Deletes the processed file from source Blob container
- Completes **move semantics**

---

## 8. Error Handling & Reliability
- Failure of one file does not impact others
- Retry policies can be configured
- Error handling can be extended with logging or alerting

---

## 9. Performance Considerations
- ForEach batch count can be tuned for parallel processing
- Suitable for large historical datasets
- Recommended to run during off-peak hours

---

## 10. Reusability & Extensibility
- Folder format can be changed easily
- Can be extended to:
  	- Validate file formats
	- Route invalid files to error folders
	- Apply size or extension-based filtering

---

## 11. Final Outcome
- Historical files are logically grouped by date
- Data lake is well-organized and analytics-ready
- One-time bulk ingestion completed using Azure Data Factory
