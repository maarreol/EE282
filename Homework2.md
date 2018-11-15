#### Homework 2 Questions and Answers

1. Write out the commands necessary to: 
   1. Navigate to your home directory, 
   2. create a new directory titled Duh and a file titled okay in your home directory. 
   3. Move the okay file into the Duh directory 
   4. Delete okay from this directory.
   5. Show the files in Duh 

>cd ~ - to move to home directory  
>mkdir Duh - to create a directory titled "Duh"  
>touch okay - to create a file titled "okay"  
>mv okay ./Duh - to move the file "okay" to the "Duh" directory   
>cd ./Duh - to change directory to Duh  
>rm okay - to remove file titled "okay" from this directory  
>ls - to list files and directories in current working directory  

### Question 1 Comments:
Very well done! Great job.

2. Create a matrix and dataframe with numeric indices and compare the differences 

>mymatrix <- matrix (1:9, ncol=3, nrow=3)  
>mydf <- data.frame (x=1:3, y=4:6, z=7:9)  
>mymatrix[,1]  
>mydf[,1]  
>mydf[1]  
>mydf[[1]]

>One of the differences between a matrix and a dataframe is that with a matrix all of the data have to be of the same type.
>In this case all of the data are numeric indices. This is not true for dataframes where every column can be of a different data type (however for the purposes of this example they happen to all be numeric indices).
>mymatrix[,1] and mydf[,1] and mydf[[1]] all give pretty similar results by showing us the information in the first column of our matrix or dataframe as column vectors. When subsetting as mydf[1] however returns the columns as they would be in a dataframe (not vectors). 

### Question 2 Comments:

Good job.

3. Create a file in a directory that can only be read by a colleague in your home directory. 

>cd ~ - to move to home directory  
>ls -l - to see permission currently being granted to directories and files in home directory  
>chmod go+x Duh  - to give group and others permission to execute the directory Duh  
>touch hi - to create file hi  
>chmod go+r hi - to give permission to read the file hi  
>chmod go-wx hi - to remove permissions to write and execute file hi  
>ls -l - to see permissions currently being granted to files
