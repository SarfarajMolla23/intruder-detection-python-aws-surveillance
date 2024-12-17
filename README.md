# intruder-detection-python-aws-surveillance
![Screenshot 2024-12-17 204238](https://github.com/user-attachments/assets/0a24b07a-318c-48fa-b610-a5e8585cabd7)

### pipeline
![Screenshot 2024-12-17 204250](https://github.com/user-attachments/assets/bff3a2f8-f205-47c0-aef7-ccdefaad3be0)


# Amazon Kinesis Video Streams Setup Guide

## Producer Setup

### Setting up on Ubuntu

1. Update packages:
   ```bash
   sudo apt update
   ```

2. Clone the repository:
   ```bash
   git clone https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
   ```

3. Create a build directory:
   ```bash
   mkdir -p amazon-kinesis-video-streams-producer-sdk-cpp/build
   cd amazon-kinesis-video-streams-producer-sdk-cpp/build
   ```

4. Install required dependencies:
   ```bash
   sudo apt-get install libssl-dev libcurl4-openssl-dev liblog4cplus-dev libgstreamer1.0-dev \
   libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-base-apps gstreamer1.0-plugins-bad \
   gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-tools

   sudo apt install cmake
   sudo apt-get install g++
   sudo apt-get install build-essential
   ```

5. Build and install the SDK:
   ```bash
   cmake .. -DBUILD_DEPENDENCIES=OFF -DBUILD_GSTREAMER_PLUGIN=ON
   make
   sudo make install
   ```

6. Set environment variables:
   ```bash
   cd ..
   export GST_PLUGIN_PATH=`pwd`/build
   export LD_LIBRARY_PATH=`pwd`/open-source/local/lib
   ```

7. Create a video stream in [Kinesis Video Streams](https://console.aws.amazon.com/kinesisvideo/).

8. Create an IAM user with `AmazonKinesisVideoStreamsFullAccess` permissions and generate access keys.

9. Stream video using the following command:
   ```bash
   gst-launch-1.0 v4l2src do-timestamp=TRUE device=/dev/video0 ! videoconvert ! \
   video/x-raw,format=I420,width=640,height=480,framerate=30/1 ! x264enc bframes=0 \
   key-int-max=45 bitrate=500 ! video/x-h264,stream-format=avc,alignment=au,profile=baseline ! \
   kvssink stream-name=STREAM-NAME storage-size=512 access-key=ACCESS_KEY secret-key=SECRET_KEY aws-region=REGION_NAME
   ```

---

## Consumer Setup

### Consumer #1: Backup

1. Create an S3 bucket.

2. Launch an EC2 instance (t2.medium) with 8GB storage.

3. Create an IAM role with `AmazonKinesisVideoStreamsFullAccess` and `AmazonS3FullAccess` policies, and attach it to the EC2 instance.

4. SSH into the EC2 instance and execute the following commands:
   ```bash
   sudo apt update
   sudo apt install python3-virtualenv

   virtualenv venv --python=python3
   source venv/bin/activate

   git clone https://github.com/aws-samples/amazon-kinesis-video-streams-consumer-library-for-python.git
   cd amazon-kinesis-video-streams-consumer-library-for-python
   pip install -r requirements.txt
   ```

5. Edit the `region` and `stream name` in the script.

6. Add backup functionality.

7. Run the consumer:
   ```bash
   python kvs_consumer_library_example.py
   ```

---

### Consumer #2: Intruder Detection

1. Create the following resources:
   - An S3 bucket
   - A DynamoDB table
   - An SNS topic
   - A subscription to the SNS topic

2. Set up event notifications for the S3 bucket.

3. Launch an EC2 instance (t2.medium) with 8GB storage.

4. Create an IAM role with the following policies:
   - `AmazonKinesisVideoStreamsFullAccess`
   - `AmazonDynamoDBFullAccess`
   - `AmazonS3FullAccess`
   - `AmazonRekognitionFullAccess`

   Attach the role to the EC2 instance.

5. SSH into the EC2 instance and execute the following commands:
   ```bash
   sudo apt update
   sudo apt install python3-virtualenv

   virtualenv venv --python=python3
   source venv/bin/activate

   git clone https://github.com/aws-samples/amazon-kinesis-video-streams-consumer-library-for-python.git
   cd amazon-kinesis-video-streams-consumer-library-for-python
   pip install -r requirements.txt

   sudo apt-get update && sudo apt-get install ffmpeg libsm6 libxext6 -y
   ```

6. Edit the `region` and `stream name` in the script.

7. Add intruder detection functionality.

8. Run the consumer:
   ```bash
   python kvs_consumer_library_example.py
   
