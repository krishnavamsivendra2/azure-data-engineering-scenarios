# Incremental Load: Event-Driven Logical Grouping of Files Using System Date in Azure Data Factory

## 1. Scenario / Problem Statement
After the initial bulk load is completed, new files arrive **daily** in the source **Azure Blob Storage** container.  
Each execution processes **only files for the current date**.
The requirement is to implement an **event-driven incremental pipeline** that automatically organizes incoming files into **date-based folders (`yyyy/MM/dd`)** in **ADLS Gen2**, using the **system date at runtime**.

---

## 2. Business Requirement
- Automatically trigger pipeline on file arrival
- Handle incremental daily data ingestion
- Organize files into date-based folders using system date
- Move files from Blob Storage to ADLS Gen2
- Avoid looping and batch processing

---

## 3. Assumptions
- Trigger type is **event-based**
- Source container contains **only files for the current date**
- Date is derived from **ADF system runtime**
- Source storage is **Azure Blob Storage**
- Destination storage is **ADLS Gen2**
- Azure Data Factory has required read/write/delete permissions
- Folder structure follows `yyyy/MM/dd`

---

## 4. Technology Stack
- **Azure Data Factory**
- **Azure Blob Storage**
- **Azure Data Lake Storage Gen2**
- Blob Event Trigger
- Parameterized datasets

---

## 5. High-Level Architecture
1. Event trigger detects new file arrival
2. Pipeline captures file name and path from trigger
3. Destination folder path is generated using system date
4. File is copied to ADLS Gen2
5. Source file is deleted after copy

---

## 6. Detailed Solution Design

### Step 1: Event-Based Trigger
- Configure **Blob Event Trigger**
- Trigger fires on `BlobCreated` event
- Trigger payload automatically provides:
	- File name
	- Folder path

---

### Step 2: Pipeline Parameters
Create the following pipeline parameters populated by the trigger:

| Parameter Name | Data Type |        Description        |
|----------------|-----------|---------------------------|
| `p_fileName`   |   String  | Name of the incoming file |
| `p_folderPath` |   String  | Source folder path        |
 
---

### Step 3: Create Pipeline Variable

|        Variable Name      | Data Type |                 Description                  |
|---------------------------|-----------|----------------------------------------------|
| `destination_folder_name` |  String   | Destination folder path based on system date |

> ℹ️ **Note:**  
> The variable is populated dynamically during pipeline execution using a **Set Variable** activity

---

### Step 4: Generate Destination Folder Path Using System Date
- Use **Set Variable** activity
- Folder structure is derived from current system date

ADF expression used:

```text
@formatDateTime(utcNow(), 'yyyy/MM/dd')
```
---

### Step 5: Copy File to ADLS Gen2
- Use **Copy Data** activity
- Source
	- Azure Blob Storage (file name and path received from trigger payload)
- Sink:
	- Azure Data Lake Storage Gen2
- Folder path dynamically set using `destination_folder_name`
> 📌 Note:
> ADLS Gen2 automatically creates folders if they do not exist
> Existing folders are reused without additional checks

---

### Step 6: Delete Source File
- Use **Delete** activity
- Deletes the processed file from the source Blob container
- Completes move semantics for incremental ingestion

---

## 7. Error Handling & Reliability
- Each file is processed independently
- Failure of one file does not impact others
- Retry policies can be applied to Copy Data activity
- Pipeline failures can be logged or monitored using Azure Monitor

---

## 8. Performance Considerations
- No ForEach or looping logic required
- Event-driven execution ensures low latency
- Scales naturally with incoming file volume
- Suitable for near real-time ingestion scenarios

---

## 9. Reusability & Extensibility
- Folder format can be easily modified
- Can be extended to:
	- Add data validation rules
	- Route files to error folders
	- Apply source-system-based partitioning

---

## 10. Final Outcome
- Incoming files are automatically organized by ingestion date
- Data lake remains clean, structured, and analytics-ready
- Completes a fully automated, event-driven incremental ingestion pattern using Azure Data Factory
