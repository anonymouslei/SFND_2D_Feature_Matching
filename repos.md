# Report for Project SFND 2D Feature Tracking
### MP.1 Data Buffer
Lines 70-78 in MidTermProject_Camera_Student.cpp
```c++
DataFrame frame;
frame.cameraImg = imgGray;

if(dataBuffer.size() < dataBufferSize) {
dataBuffer.push_back(frame);
} else {
dataBuffer.erase(dataBuffer.begin());
dataBuffer.push_back(frame);
}
```
### MP.2 Keypoint Detection
Lines 98-105 in MidTermProject_Camera_Student.cpp

Lines 143-263 in matching2D_Student.cpp
```c++

// detect keypoints in image using the traditional Harris detector
void detKeypointsHarris(std::vector<cv::KeyPoint> &keypoints, cv::Mat &img, double& t, bool bVis)
{
    // ...
}
// FAST, BRISK, ORB, AKAZE and SIFT
void detKeypointsModern(std::vector<cv::KeyPoint> &keypoints, cv::Mat &img, double& t, std::string detectorType, bool bVis)
{
    // ...
}
``` 
### MP.3 Keypoint Removal
Line 111-123 in MidTermProject_Camera_Student.cpp
```c++
        // only keep keypoints on the preceding vehicle
        bool bFocusOnVehicle = true;
        cv::Rect vehicleRect(535, 180, 180, 150);
        if (bFocusOnVehicle)
        {
            vector<cv::KeyPoint> newKeypoints;
            for(auto kpt: keypoints)
            {
                if(vehicleRect.contains(kpt.pt))
                    newKeypoints.push_back(kpt);
            }
            keypoints = newKeypoints;
        }
```
### MP.4 Keypoint Descriptors
Lines 151-154 in MidTermProject_Camera_Student.cpp
```c++
        cv::Mat descriptors;
        string descriptorType = "BRISK"; // BRIEF, ORB, FREAK, AKAZE, SIFT
        double descriptorTime;
        descKeypoints((dataBuffer.end() - 1)->keypoints, (dataBuffer.end() - 1)->cameraImg, descriptorTime, descriptors, descriptorType);
```
Lines 65-99 in matching2D_Student.cpp
```c++
void descKeypoints(vector<cv::KeyPoint> &keypoints, cv::Mat &img, double& t, cv::Mat &descriptors, string descriptorType)
{
```
### MP.5 Descriptor Matching
FLANN in lines 27-39 in matching2D_Student.cpp
```c++
    else if (matcherType.compare("MAT_FLANN") == 0)
    {
        if(descSource.type() !=CV_32F)
        {
            descSource.convertTo(descSource, CV_32F);
        }
        if(descRef.type() != CV_32F)
        {
            descRef.convertTo(descRef, CV_32F);
        }

        matcher = cv::DescriptorMatcher::create(cv::DescriptorMatcher::FLANNBASED);
    }
```
KNN in line 47-61 in matching2D_Student.cpp
```c++
    else if (selectorType.compare("SEL_KNN") == 0)
    { // k nearest neighbors (k=2)
        vector<vector<cv::DMatch>> knn_matches;
        matcher->knnMatch(descSource, descRef, knn_matches, 2); // k=2

        const float ratio_thresh = 0.8f;
        for (size_t i = 0; i < knn_matches.size(); i++)
        {
            if(knn_matches[i][0].distance < ratio_thresh * knn_matches[i][1].distance)
            {
                matches.push_back(knn_matches[i][0]);
            }
        }
        cout << "# keypoints removed = " << knn_matches.size() - matches.size() << endl;
    }
```
### MP.6 Descriptor Distance Ratio
```c++
        const float ratio_thresh = 0.8f;
        for (size_t i = 0; i < knn_matches.size(); i++)
        {
            if(knn_matches[i][0].distance < ratio_thresh * knn_matches[i][1].distance)
            {
                matches.push_back(knn_matches[i][0]);
            }
        }
```
### MP.7 Performance Evaluation 1
Following this [Q&A](https://knowledge.udacity.com/questions/105491),
I implemented all the detectors and save all information in the [res_roi.csv](./res/res_roi.csv) file. 
I have summarized the average number of keypoints in ROI in the following table.
The distribution of keypoints' neighborhood size can be seen in the image.

BRISK detect the most amount of keypoint on the preceding vehicle.
HARRIS detect the least. 

Detector  | Average number of detected keypoints on the preceding vehicle | Image
--------  | --------------------| -------------------
FAST      | 149| ![](./res/imgs/FAST_BRISK.png)
SHITOMASI | 117 | ![](./res/imgs/SHITOMASI_BRISK.png)
HARRIS    | 24 | ![](./res/imgs/HARRIS_BRISK.png)
BRISK     | 276| ![](./res/imgs/BRISK_BRISK.png)
ORB       | 116| ![](./res/imgs/ORB_BRISK.png)
AKAZE     | 167| ![](./res/imgs/AKAZE_BRISK.png)
SIFT      | 138| ![](./res/imgs/SIFT_BRISK.png)
### MP.8 Performance Evaluation 2
I implemented all the detectors and save all information in the [res_roi_all.csv](./res/res_roi_all.csv) file. 
**Results: Average number of matched keypoints for detector-descriptor pairs**

| Det\Desc  | BRISK | BRIEF | ORB | FREAK | SIFT | AKAZE | 
|-----------|-------|-------|-----|-------|-------|------|
| FAST      | 99   | 122   | 119 | 98    | 116   |N/A |
| SHITOMASI | 85    | 104   | 100 | 85    |  103  |N/A   |
| HARRIS    | 15    | 19    | 18  | 16    |  18   |N/A   |
| BRISK     | 174   | 189   | 168 | 169   |  182  |N/A   |
| ORB       | 83    | 60    | 85  | 46    | 84   |N/A   |
| AKAZE     | 135   | 140   | 131 | 131   |  141  | 130   |
| SIFT      | 65    | 78    | N/A | 65    |  88   |N/A   |
### MP.9 Performance Evaluation 3

**Results: Average total time (ms) of detection and descriptor extraction for detector-descriptor pairs **

| Det\Desc  | BRISK | BRIEF | ORB | FREAK | SIFT | AKAZE | 
|-----------|-------|-------|-----|-------|-------|------|
| FAST      | 3.28 | 2.98 | 2.09 | 55.53 | 38.07 | N/A
| SHITOMASI | 22.62 | 22.36 | 20.23 | 67.71 | 35.12 | N/A
| HARRIS    | 22.77 | 7.85 | 23.02 | 70.12 | 50.83 | N/A
| BRISK     | 52.62 | 52.56 | 80.41 | 105.39 | 121.24 | N/A
| ORB       | 11.57 | 11.58 | 15.97 | 63.37 | 95.61 | N/A
| AKAZE     | 112.46 | 138.30 | 129.02 | 179.58 | 153.63 | 219.42
| SIFT      |147.33 | 168.08 | N/A | 229.84 | 234.66 | N/A

The TOP3 detector / descriptor combinations are:

Detector+Descriptor     | Average number of matched keypoints | Total time | Keypoints/time ratio
------------ | :----------------------------:| :--------: | :-------------:
FAST+ORB     | 119 keypoints                 | 2.09 ms     | 57
FAST+BRIEF   | 122 keypoints                 | 2.98 ms     | 41
FAST+BRISK   | 99 keypoints                 | 3.28 ms     | 30

