# Console and Cloud Shell

## Overview

In this lab, you become familiar with the Google Cloud web-based interface. There are two integrated environments: a GUI (graphical user interface) environment called the Cloud Console, and a CLI (command-line interface) called Cloud Shell. In this class you use both environments.

Here are a few things you need to know about the Cloud Console:

- The Cloud Console is under continuous development, so occasionally the graphical layout changes. This is most often to accommodate new Google Cloud features or changes in the technology, resulting in a slightly different workflow.
- You can perform most common Google Cloud actions in the Cloud Console, but not all actions. In particular, very new technologies or sometimes detailed API or command options are not implemented (or not yet implemented) in the Cloud Console. In these cases, the command line or the API is the best alternative.
- The Cloud Console is extremely fast for some activities. The Cloud Console can perform multiple actions on your behalf that might require many CLI commands. It can also perform repetitive actions. In a few clicks you can accomplish activities that would require a great deal of typing and would be prone to typing errors.
- The Cloud Console is able to reduce errors by only offering up through its menus valid options. It is able to leverage access to the platform "behind the scenes" through the SDK to validate configuration before submitting changes. A command line can't do this kind of dynamic validation.

## Objectives

In this lab, you learn how to perform the following tasks:

- Get access to Google Cloud.
- Create a Cloud Storage bucket using the Cloud Console.
- Create a Cloud Storage bucket using Cloud Shell.
- Become familiar with Cloud Shell features.

## Task 1: Create a bucket using the Cloud Console

1. To login on the gcloud sdk, type in the following commands and follow the prompts:

   ```
   gcloud init
   ```

2. create a bucket with a globally unique name using the following command:

   ```
   gsutil mb gs://Bucket_Name
   ```

## Task 2: Access Cloud Shell

In this section, you explore Cloud Shell and some of its features.

You can use the Cloud Shell to manage projects and resources via command line, without having to install the Cloud SDK and other tools on your computer.

1. Start a SSH session:

   ```
   gcloud alpha cloud-shell ssh
   ```

2. Copy a file, 'data.txt', from Cloud Shell to your local machine:

   ```
   gcloud alpha cloud-shell scp cloudshell:~/data.txt localhost:~data.txt
   
   ```

3. Mount your cloud shell directory onto your local file system:

   ```
   gcloud alpha cloud-shell get-mount-command ~/my-cloud-shell
   ```

4. Set your desired configuration using:

   ```
   gcloud init
   ```

5. Close cloud shell:

   ```
   exit
   ```

## Task 3: Create a bucket using Cloud Shell

### Create a second bucket and verify in the Cloud Console

1. open cloud shell:

   ```
   gcloud alpha cloud-shell ssh
   ```

2. Use the gsutil command to create another bucket. Replace <BUCKET_NAME> with a globally unique name (you can append a 2 to the globally unique bucket name you used previously):

   ```
   gsutil mb gs://<BUCKET_NAME>
   ```

## Task 4: Explore more Cloud Shell features

Upload a File to the created bucket from your local machine with the following command:

```
gsutil cp OBJECT_LOCATION gs://<BUCKET_NAME>
```

Where OBJECT_LOCATION refers to the local path to your object.

## Task 5: Create a persistent state in Cloud Shell

In this section you will learn a best practice for using Cloud Shell. The gcloud command often requires specifying values such as a **Region** or **Zone** or **Project ID**. Entering them repeatedly increases the chances of making typing errors. If you use Cloud Shell a lot, you may want to set common values in environment variables and use them instead of typing the actual values.

**Identify available regions**

To list available regions, execute the following command:

```
gcloud compute regions list
```

Select a region from the list and note the value in any text editor. This region will now be referred to as [YOUR_REGION] in the remainder of the lab.

### Create and verify an environment variable

1. Create an environment variable and replace [YOUR_REGION] with the region you selected in the previous step:

   ```
   INFRACLASS_REGION=[YOUR_REGION]
   ```

2. Verify it with echo:

   ```
   echo $INFRACLASS_REGION
   ```

   You can use environment variables like this in gcloud commands to reduce the opportunities for typos, and so that you won't have to remember a lot of detailed information.

### Append the environment variable to a file

1. Create a subdirectory for materials used in this class:

   ```
   mkdir infraclass
   ```

2. Create a file called "config" in the infraclass directory:

   ```
   touch infraclass/config
   ```

3. Append the value of your Region environment variable to the config file:

   ```
   echo INFRACLASS_REGION=$INFRACLASS_REGION >> ~/infraclass/config
   ```

4. Create a second environment variable for your Project ID, replacing [YOUR_PROJECT_ID] with your Project ID. You can find the project ID on the Cloud Console Home page:

   ```
   INFRACLASS_PROJECT_ID=[YOUR_PROJECT_ID]
   ```

5. Append the value of your Project ID environment variable to the config file:

   ```
   echo INFRACLASS_PROJECT_ID=$INFRACLASS_PROJECT_ID >> ~/infraclass/config
   ```

6. Use the source command to set the environment variables, and use the echo command to verify that the project variable was set:

   ```
   source infraclass/config
   echo $INFRACLASS_PROJECT_ID
   ```

   This gives you a method to create environment variables and to easily recreate them if the Cloud Shell is cycled. However, you will still need to remember to issue the source command each time Cloud Shell is opened.

   In the next step you will modify the .profile file so that the source command is issued automatically any time a terminal to Cloud Shell is opened.

7. Close and re-open Cloud Shell. Then issue the echo command again:

   ```
   exit /
   gcloud alpha cloud-shell ssh /
   echo $INFRACLASS_PROJECT_ID
   ```

   There will be no output because the environment variable no longer exists.

### Modify the bash profile and create persistence

1. Edit the shell profile with the following command:

   ```
   nano .profile
   ```

2. Add the following line to the end of the file:

   ```
   source infraclass/config
   ```

3. Press **Ctrl+O**, **ENTER** to save the file, and then press **Ctrl+X** to exit nano.

4. Close and then re-open Cloud Shell to cycle the VM.

   ```
   exit/
   gcloud alpha cloud-shell ssh
   ```

5. Use the echo command to verify that the variable is still set:

   ```
   echo $INFRACLASS_PROJECT_ID
   ```

   You should now see the expected value that you set in the config file.

## Task 6: Review the Google Cloud interface

Cloud Shell is an excellent interactive environment for exploring Google Cloud using Google Cloud SDK commands like `gcloud` and `gsutil`.

You can install the Google Cloud SDK on a computer or on a VM instance in Google Cloud. The gcloud and gsutil commands can be automated using a scripting language like bash (Linux) or Powershell (Windows). You can also explore using the command-line tools in Cloud Shell, and then use the parameters as a guide for re-implementing in the SDK using one of the supported languages.

The Google Cloud interface consists of two parts: the Cloud Console and Cloud Shell.

The Console:

- Provides a fast way to get things done
- Presents options to you, instead of requiring you to know them
- Performs behind-the-scenes validation before submitting the commands

Cloud Shell provides:

- Detailed control
- Complete range of options and features
- A path to automation through scripting