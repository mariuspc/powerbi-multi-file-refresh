# Power BI: refresh from multiple similar files
Example of how to refresh a Power BI dataset from multiple similar files

## Example Scenario
I have a folder on my local machine where every day a new file is added, containing the aggregated sales numbers that took place during the day. All files have a similar structure, but obviously contain different data. 

![file.png](images/file-structure.png)

All the files adhere to the same naming convention, as seen below in the screenshot below:

![file-names.png](images/file-names.png)

It should be noted that the date of the sales is only present in the file names, and not in the file contents. 
