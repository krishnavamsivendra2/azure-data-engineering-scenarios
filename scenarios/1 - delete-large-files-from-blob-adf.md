# Delete Files Larger Than 1 MB from Azure Blob Storage using Azure Data Factory

## 1. Scenario / Problem Statement
There is a requirement to automatically delete files from an Azure Blob Storage container if the file size exceeds **1 MB**.  
This cleanup process should run in an automated and scalable manner without manual intervention.

---

## 2. Business Requirement
- Identify all files present in a Blob container
- Check the size of each file
- Delete files whose size is greater than **1 MB**
- Solution should be automated and reusable

---

## 3. Assumptions
- The Blob container contains only files (no nested folders)  
- File size comparison is done in **KB**
- 1 MB = **1024 KB**
- Azure Data Factory has required permissions to read and delete blob files

---

## 4. Technology Stack
- **Azure Data Factory**
- **Azure Blob Storage**
- Linked Services (Parameterized & Non-Parameterized)

---

## 5. High-Level Architecture
1. Fetch list of files from Blob container
2. Iterate through each file
3. Fetch metadata (file size)
4. Apply conditional check
5. Delete files greater than 1 MB

---

## 6. Detailed Solution Design

### Step 1: Get List of Files from Blob Container
- Use **Get Metadata** activity
- Linked Service: **Non-parameterized**
- Field selected: `Child Items`
- Output provides a list of all file names present in the container

---

### Step 2: Iterate Over Each File
- Use **ForEach** activity
- Input:
	@activity('GetMetadata_Container').output.childItems
- This allows iteration over each file returned from the container

---

### Step 3: Fetch Metadata for Each File (Inside ForEach)
- Use **Get Metadata** activity
- Linked Service: **Parameterized**
- Parameter passed from ForEach:
	item().name
- Fields selected:
- `Item Name`
- `Size`

This step retrieves the size of each individual file.

---

### Step 4: Apply Conditional Logic
- Use **If Condition** activity
- Condition:
	@greater(activity('GetMetadata_File').output.size, 1024)
- Logic:
- If file size is greater than **1024 KB (1 MB)** → proceed to delete
- Else → no action

---

### Step 5: Delete File
- Use **Delete** activity (inside True condition)
- Pass the file name dynamically:
	item().name
- This deletes only files exceeding the size threshold

---

## 6. Error Handling & Reliability
- If a file cannot be accessed, the pipeline continues for remaining files
- Failure can be logged using:
- Pipeline failure paths
- Azure Monitor alerts (optional)

---

## 7. Performance Considerations
- ForEach can be configured with **batch count** to improve performance
- Suitable for containers with moderate number of files
- For very large containers, segmentation logic can be added

---

## 8. Reusability & Extensibility
- File size threshold can be parameterized
- Can be extended to:
- Archive files instead of deleting
- Apply conditions based on file extension
- Schedule periodic cleanup

---

## 10. Final Outcome
- All files larger than **1 MB** are automatically identified and deleted
- No manual intervention required
- Fully automated and scalable solution using Azure Data Factory
