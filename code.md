   ### Define TimeZone
    Sys.setenv(TZ='GMT')

    install.packages("devtools",lib=Sys.getenv("AZ_BATCH_TASK_WORKING_DIR"))
    library(devtools,lib=Sys.getenv("AZ_BATCH_TASK_WORKING_DIR"))
### Install rAzureBatch package (put rAzureBatch zip file in script folder)
  #### then mention its URL
    install_url("xxxxxxxxxxx")
    library(rAzureBatch)

   ### Define parameters of the Blob Storage Account
    storageName   <- "Storage name"
    storageKey    <- "Storage Key"
    containerName <- "Container Name"

   ### Submit the shared key credentials
     storageCredentials <- rAzureBatch::SharedKeyCredentials[["new"]](
      name = storageName,
      key  = storageKey
        )

   ### Define storage client
    storageClient <- rAzureBatch::StorageServiceClient[["new"]](
          authentication = storageCredentials,
          url            = sprintf("https://%s.blob.%s",
                           storageName,
                           "core.windows.net"
                          )
                  )

   ### Generate read SaS token
    readSasToken <- storageClient[["generateSasToken"]](permission = "r", "c", path = containerName)


   ### Get the blob url for the  dataset
      blobUrl <- rAzureBatch::createBlobUrl(storageAccount = storageName,
                                      containerName  = containerName,
                                      sasToken       = readSasToken,
                                      fileName       = "input/filename.csv")

   ### Read file using the blob url

      Dataset <- read.csv(blobUrl)

__ Perfrom your data transformation and monipulation work __ 

### Create a local temporary folder and save the result of the analysis
    dir.create("output")

    write.csv(Dataset, "output/filename.csv", row.names = F)

   ### Generate write sas token
    
    writeSasToken <- storageClient[["generateSasToken"]](permission = "w", "c", path = containerName)

   ### Upload dataset to the blob storage container

    storageClient[["blobOperations"]][["uploadBlob"]](
      sasToken      = writeSasToken,
      containerName = paste0(containerName,"/output"),
      fileDirectory = "output/filename.csv",
      accountName   = storageName
    )

