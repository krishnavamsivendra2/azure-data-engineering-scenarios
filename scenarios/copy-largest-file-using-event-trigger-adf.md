# Copy the Largest File to a Secondary Container Using Event Trigger in Azure Data Factory

## 1. Scenario / Problem Statement
Whenever a new file is added to a source Azure Blob Storage container, an **event trigger** should automatically initiate a pipeline that identifies the **largest file for the current date** and copies it to a secondary Blob container.

---

## 2. Business Requirement
- Trigger pipeline automatically on blob arrival
- Identify the largest file present for the current date
- Copy only the largest file to a destination container
- Ensure the solution is automated and scalable

---

## 3. Assumptions
- Files are organized by **date-based folders** (e.g., `yyyy/MM/dd`)
- Each event trigger corresponds to a single day’s data
- File size comparison is done in **KB**
- Minimum valid file size is **0 KB**
- Azure Data Factory has read/write permissions on both containers

---

## 4. Technology Stack
- **Azure Data Factory**
- **Azure Blob Storage**
- Event-based triggers
- Pipeline variables

---

## 5. High-Level Architecture
1. Event trigger detects file arrival
2. Pipeline initializes comparison variables
3. All files for the day are scanned
4. Largest file is identified
5. File is copied to destination container

---

## 6. Detailed Solution Design

### Step 1: Pipeline Trigger
- Configure **Blob Event Trigger**
- Trigger fires whenever a file is added to the source container
- Pipeline execution begins automatically

---

### Step 2: Initialize Variables
Create and initialize the following pipeline variables:

| Variable Name 		|    Type 	|                Purpose 		|
|-------------------------------|---------------|---------------------------------------|
| `currentDate` 		| String 	| Stores current date 			|
| `maxFileSizeKB`		| Int 		| Tracks the largest file size 		|
| `largestFileName` 		| String 	| Stores the name of the largest file 	|

- `currentDate` is set using **Set Variable** activity with system date
- `maxFileSizeKB` is initialized to **0**
- `largestFileName` is initialized as empty

---

### Step 3: Fetch List of Files from Source Container
- Use **Get Metadata** activity
- Linked Service: Non-parameterized (source container)
- Field selected: `Child Items`
- Output contains list of all files for the current date

---

### Step 4: Iterate Through Files
- Use **ForEach** activity
- Input:
	@activity('GetMetadata_Container').output.childItems

#### Inside ForEach:

##### a) Fetch Metadata for Each File
- Use **Get Metadata** activity
- Parameters:
- File name from `item().name`
- Fields selected:
- `Item Name`
- `Size`

##### b) Compare File Size
- Use **If Condition** activity
- Condition:
	@greater(activity('GetMetadata_File').output.size, variables('maxFileSizeKB'))

##### c) Update Variables
If condition is true:
- Update `maxFileSizeKB` with current file size using **Set Variable** activity 
- Update `largestFileName` with current file name using **Set Variable** activity 

This ensures the variables always store the largest file encountered so far.

---

### Step 5: Copy the Largest File
- After **ForEach** completes, `largestFileName` holds the required file
- Use **Copy Data** activity
- Source file name is passed dynamically:
	variables('largestFileName')
- File is copied from source container to destination container

---

## 6. Error Handling & Reliability
- Pipeline continues even if a single file metadata read fails
- Copy activity executes only after full comparison is completed
- Can be extended with failure logging or alerting

---

## 7. Performance Considerations
- ForEach batch count can be optimized
- Suitable for moderate daily file volumes
- Event-based execution avoids unnecessary scheduled runs

---

## 8. Reusability & Extensibility
- Logic can be reused for:
- Smallest file selection
- Multiple destination containers
- File-type based filtering
- Date logic can be parameterized

---

## 9. Final Outcome
- Pipeline automatically reacts to blob arrival
- Identifies the **largest file for the day**
- Copies only the required file
- Fully event-driven and automated using Azure Data Factory
