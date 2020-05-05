# Power BI: dataset refresh from multiple similar files
Example of how to refresh a Power BI dataset from multiple similar files

## Example Scenario

I have a folder on my local machine where every day a new file is added, containing the aggregated sales numbers that took place during the day. All files have a similar structure, but obviously contain different data. 

![file.png](images/file-structure.png)

All the files adhere to the same naming convention, as seen below in the screenshot below:

![file-names.png](images/file-names.png)

It should be noted that the date of the sales is only present in the file names, and not in the file contents. I need to build a report that:
  1) refreshes daily
  2) will include the latest data available at the time of the refresh
  3) will only load the last 7 days worth of data OR the last 7 files available in the folder 
  
  ## Solution
  
  I create a new report, and choose Folder as my data source.

![crop.png](images/crop.png)

  I enter the path of folder containing all my Sales files and select Transform Data as soon as the option becomes available. I now have a view of all the files present in the folder, or a sample subset if the list is too big. If I continue without any filtering, I will load all the files into my Power BI dataset. 
  
![all-files.png](images/all-files.png)

### Only load the last 7 days
  In order to load only the Sales files for the last 7 days, I first need to extract the Sales Date from the file name. I can do this by using the Add Columns From Examples functionality, or by using traditional string operations. Once done, my dataset looks like this:
  
  ![add-sales-date.png](images/add-sales-date.png)
  
  I also need to convert the new SalesDate column into a DateTime data type before proceeding
  
  ![add-sales-date.png](images/add-sales-date-type.png)
  
  I now have a Sales Date column I can use to filter only the last 7 days. I apply a filter, using the user interface, to only show the last 7 days (for this example, the last 7 days means the Sales Date is after .
  
  ![add-sales-date.png](images/filter-date.png)
  
  While this works, I need to manually change the date filter every day, in order to make sure I still get the last 7 days tomorrow. I can do this by aplying a dynamic filter. 
  
  I need to go to the Advanced Editor option and change the line applying the SalesDate filter just created as follows:
  
  ![adv-editor.png](images/adv-editor.png)
  
  `#"Filtered Rows" = Table.SelectRows(#"Changed Type", each [SalesDate] > Date.AddDays ( DateTime.LocalNow(), -7 ))` 
  This change now calculates the last 7 days based on the current date (`DateTime.LocalNow()`), making sure the refresh will also work tomorrow. 
  
  If the number of days loaded is likely to change often, I can also use a parameter (I named the parameter noDays in this example):
  
  `#"Filtered Rows" = Table.SelectRows(#"Changed Type", each [SalesDate] > Date.AddDays ( DateTime.LocalNow(), -1 * noDays ))`
  
### Only load the last 7 files 
  There is a chance I might not get a file every single day. To make sure I still load 7 files in that situation, I will change the filter to load the last 7 files based on sales date (as opposed to the last 7 days loaded above).
    
  To implement this, I still have to create my SalesDate column and convert it to DateTIme, just like before. I then sort by the SalesDate column descending    
  
  ![sort-desc.png](images/sort-desc.png)
  
  The next step is to add an index, starting from 0
  
  ![add-index.png](images/add-index.png)    
  
  I now add a filter to only include files with the index less than 7, using the user interface
  
  ![filter-index.png](images/filter-index.png)    
    
  If the number of files I wish to load is likely to change often, I can also use a parameter (I named the parameter noFiles in this example). Once I create the parameter, I just need to change the filter in the Advanced Editor, as follows:
    
    `#"Filtered Rows" = Table.SelectRows(#"Added Index", each [Index] < noFiles)`
  
### Merge files
  I have now made sure I am only loading the files I need to, as opposed to everything that's in the folder. Next step is to merge the content of these files into one Power BI semantic table. In order to do that, I need to click on the Combine Files option in the Content column.
  
   ![combine-option.png](images/combine-option.png)  
    
  In the following Combine Files pop-up, I make sure to select the correct tab in my Excel files and check the preview on the right. 
  
   ![combine-files.png](images/combine-files.png)  
   
  The resulting table should contain data from all the files loaded
  
   ![combined-table.png](images/combined-table.png)       
    
  
### Add Date
  As you can see from the image above, my resulting table does not contain the sales date column. That is because the source Excel files I am loading did not have the sales date as part of their content. The only place I can extract the sales date is from the file name.
  
  Since I already extracted the SalesDate at the begining of this post to help me filter out files I did not want to use, I can just include that SalesDate in my resulting table. Here's how I do it:
  
  When I clicked Comnbine Files, Power BI automatically generated a number of steps in the Applied Steps section of my workbook.
  
 ![expanded-steps.png](images/expanded-steps.png)       
 
  Next, I click on the little cog icon next to the last Removed Other Columns step, in order to modify the columns being removed
  
  ![remove-cols.png](images/remove-cols.png)       
  
  And I select the SalesDate column.
  
  ![select-sales-date.png](images/select-sales-date.png)       
  
  The resulting table now included the SalesDate, as well as the data from all the relevant files.
  
  ![all-data.png](images/all-data.png)       
    
  ### Summary 
