<h1>Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab</h1> 

> Lab Link : https://www.cloudskillsboost.google/catalog_lab/2467
<h3>Solution:</h3>
<hr>

### Task 1: Create a bucket

* 1. Copy the 'Bucket Name' available under task 1 and on the left pane. (e.g. `memories-bucket-16601`)
* 2. Login to Google Cloud Platform using the credentials available on the left pane. (`Username1` and `Password`).
* 2. Go to **Navigation Menu (☰)** from the top-left corner > Select **'Cloud Storage'** > Select **'Bucket'**.
* 3. Select **'Create a Bucket'**
* 4. 1. Paste the task 1 name in **Name Your Bucket**. 
     2. Scroll down to the end and select `CREATE`

### Task 2: Create a Pub/Sub topic

* 1. Copy the 'Topic Name' available under task 2 and on the left pane. (e.g. `memories-topic-734`)
* 2. Go to **Navigation Menu (☰)** from the top-left corner > Scroll down to bottom, and select `more options` dropdown > Select **'Pub/Sub'** > Select **'Topics'**
* 3. Select **'Create Topic'**.
* 4. Paste the 'Topic Name' in **'Topic ID'**.
* 5. Select `CREATE TOPIC`.

### Task 3. Create the thumbnail Cloud Function

* 1. Copy the 'Cloud Function Name' available under task 2 and on the left pane. (e.g. `memories-thumbnail-creator` )
* 2. Go to GCP(Google Cloud Platform) and click 'Search' > type 'Cloud Function' and select 'Cloud Function'. 
* 3. Select `CREATE FUNCTION`.
* 4. 1. Paste 'Cloud Function Name' (copied at the step i) in **'Function Name'** 
     2. Select 'us-east1' from **'Region'** dropdown
     3. Scroll-down and go to **'Trigger Type'** > Select 'Cloud Storage'.
     4. Go to **'Event type'** > Select 'Finalize/Create'.
     5. Go to **'Bucket'** > Select `BROWSE` > Select the bucket you previously created.
     6. Click `SAVE`.
     7. Tap 'Next'.
* 5. *To be continued*

