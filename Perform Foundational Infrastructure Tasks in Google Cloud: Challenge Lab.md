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
* 5. 1. On the Entry point, write `thumbnail`.
     2. In the runtime, select **Node.js 14** 
* 6. Go to the **index.js** file > Delete everything by pressing `Ctrl` + `A` then `Backspace`. ( For Mac, `⌘` + `A` then `delete`)
* 7. Then copy the block of code from below and paste it in index.js file. For Example: *(In line 15 of index.js replace the text REPLACE_WITH_YOUR_TOPIC     ID with the Topic Name you created in task 2.)*
```yaml
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "REPLACE_WITH_YOUR_TOPIC ID";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
* 8. Go to the **package.json** file > Delete everything by pressing `Ctrl` + `A` then `Backspace`. ( For Mac, `⌘` + `A` then `delete`)
* 9. Then copy the block of code from below and paste it in package.json file.

```yaml

{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}

```

* 10. Now Click `DEPLOY`.
* 11. Wait until it is completely ready (check when the loading icon stops and green tick appears).
![Screen Recording 2022-11-23 at 9 26 04 PM-2](https://user-images.githubusercontent.com/58916385/203592878-114105cf-d422-40dc-864f-6d14da774d5a.gif)



* 12. Download the image from this site:

https://storage.googleapis.com/cloud-training/gsp315/map.jpg ( Open link > Right-Click > Save / Save image as... )

* 13. Go to Navigation Menu (☰) > Cloud Storage > Browser
* 14. Click on the bucket-name that we've created previously ( The one which starts with `memories-bucket-......` )
* 15. Select `Upload Files` > Choose the image which we've downloaded few steps ago and upload.
* 16. At the far right of the newly uploaded image, choose the three-dot menu icon and click `Rename`.
* 17. Rename it as `Map.jpg`.
* 18. Click RENAME.
* 19. wait for sometime and click REFRESH on the top.
* TASK 3 DONE !!! WOO-HOO

### Task 4. Remove the previous cloud engineer

**Go back to the Cloudskills lab and check the second Username given at the left pane**

* 1. Go to Navigation Menu (☰) > IAM & Admin > IAM .
* 2. Scroll down to the end and locate the second Username > at the far right of the username, select the pencil icon > Edit Permissions page opens at the right side of the screen
* 3. Click on the Dustbin icon right beside the `Viewer` role.
* 4. Click `SAVE`.


## HURRAYYY !!! LAB COMPLETE 🥳




