---
title: "Season of KDE: Adding MCAP support to Labplot"
author: wirthual
date: 2024-04-05 20:00:00 +0800
categories: [Software Development, Open Source]
tags: [c++, qt, mcap, data analysis, labplot, kde]
render_with_liquid: false
---

This article describes the work done for adding MCAP support to LabPlot as part of the Season of KDE 2024.

Here you can see the result of the project, a MCAP file is recorded in the browser and analyzed using LabPlot!

<iframe width="560" height="315" src="https://www.youtube.com/embed/kK5QbFi90wA?si=OCwYmXHr-iD7F85r" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Adding MCAP Support for LabPlot

For the Summer of KDE project I added the capability to LabPlot to import data stored in the MCAP container format. The MCAP container format is the new default storage format for ROS2. The MCAP format is an container format which allows the storage of any encoded data. Well known encoding formats supported by MCAP include Protobuf, Flatbuf and JSON. MCAP comes with a great set of features like storing of attachments, support for compression, the support for multiple channels and topics as well as libraries in a variety of different programming languages. This versatility of MCAP makes it the ideal format to store timestamped robotics data or any other sequence of data which is logged in a sequential manner. For the scope of this project, I aimed to add support for JSON encoded MCAP files into LabPlot. This allows a direct import for analysis and visualization of your data without the need of exporting your existing MCAP files into another format.


## The implementation

The plan to include MCAP support into LabPlot was to first get the file format supported by the backend and as a second step extend the frontend and UI with the additional components to work with MCAP files. Using the [test driven development](https://martinfowler.com/bliki/TestDrivenDevelopment.html) approach, a first step was to write test cases for the import of MCAP files. The support of MCAP for Python made it super easy to create a Jupyter Notebook to quickly write out an basic MCAP file which can be used for testing. After I had an easy example to build my tests on I could extend the LabPlot Test Suite by an additional test for the MCAP support which made it a fast and efficient way of implementing the required functionality into the LabPlot backend. 

### The backend
Using the official MCAP Documentation I was able to successfully import the first MCAP file in LabPlot. The data transformation in the internal LabPlot format was the majority of the work needed. MCAP treats logTime, publishTime as well as the sequence in the recording as special fields separated from the JSON encoded payload. Additional functionality like extracting topics and reading the file partitially for preview was impleneted as a next step. After successful loading of the basic file, a more complex data structure was used to ensure correct functionality on read world data. For this, the NuScene dataset was transformed into MCAP and the data from the IMU was used to extend the initial capabilites. The support for this nested data structure was implemented in LabPlot which required the flattening of the JSON in order to be able to load it into the row and columns of LabPlot. In order to close the loop, the functionality to export data into the MCAP format was implemented. Similar as for the import, the handling of logTime, publishTime and sequence was introduced together with the JSON encoding of the general data. 

To get a sense of the performance of the import, testcase was extended to measure the execution time of the data import. This test was performed on a simple MCAP file with 100000 rows with 10 values each. This shows the import duration for the different compression methods available to MCAP:

_Import Time_            |  _File Size_
:-------------------------:|:-------------------------:
![Import Time](assets/img/seasonofkde/import_time.png) |![File Size](assets/img/seasonofkde/file_size.png)

_Note:_

These benchmarks were executed on a laptop with Intel® Core™ i7-9750H CPU @ 2.60GHz × 12 and 16GB of RAM. The visualization was of course created using LabPlot.

### The frontend
For the import functionality of the frontend, the two interaction points include the Import and Export Dialog of LabPlot. Those Dialogs needed to be extended with additional settings required for MCAP. This work required a lot of testing and handling multiple cases to make sure the user has a consistent experience working with MCAP as with the other supported formats. A lot of edge cases needed to be considered, in order to clearly communicate problems to the user. Additonal testcases got introduced including the loading of empty files, corrupted files or files which do not contain any topic with the supported JSON encoding.


_MCAP File Import Dialog_             |   _MCAP File Export Dialog_
:-------------------------:|:-------------------------:
![MCAP Import Dialog](assets/img/seasonofkde/import_dialog.png) |![MCAP Export Dialog](assets/img/seasonofkde/export_dialog.png)



## Going Forward
MCAP allows very precice time logging in nanosecond resolution. Currently LabPlot is internally storing this data using DataTime objects from the Qt Framework. A shortcomming of this object type is it only supports time in a millisecond resolution. In oder to preserve the full time accuracy provided by MCAP, the change of the internal datastructure in LabPlot to use an basic data type for storing the time in e.g. nanoseconds would be a great addition to LabPlot since this could also lead to addiational performance gains when large sets of data are imported or exported. Support for additional encodings like ROS2 or Protobuf could be implemented to allow the reading of additional data sources. An improved export functionality could allow inference of the right data schema to allow the export with a detailed schema definition of the data. 