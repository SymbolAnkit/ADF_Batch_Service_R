
# Install the rAzureBatch library
Sys.setenv(TZ='GMT')
install.packages("devtools",lib=Sys.getenv("AZ_BATCH_TASK_WORKING_DIR"))
library(devtools,lib=Sys.getenv("AZ_BATCH_TASK_WORKING_DIR"))

#install_url("https://mlpocstorage.blob.core.windows.net/scripts/Azure-rAzureBatch-v0.6.2-2-g1ab39ca.zip?sp=r&st=2019-05-10T08:07:20Z&se=2019-05-10T16:07:20Z&spr=https&sv=2018-03-28&sig=SDXU4LcChSRtJajSBe3We5RmJ89TRQcuM4gUBtfYAmw%3D&sr=b")
library(rAzureBatch)
print("lib done")
# Define parameters of the Blob Storage Account
storageName   <- "mlpocstorage"
storageKey    <- "LQX7yzaLtb1A/d11XoA2gTVwa8ZZriAkz3Oxh6le+4xkFQmDgw+oOCPv5DBOqb4IovKf3PjQJi5kDNTIRLvPSg=="
containerName <- "sample"
print("part0")
# Submit the shared key credentials
storageCredentials <- rAzureBatch::SharedKeyCredentials[["new"]](
  name = storageName,
  key  = storageKey
)
print("part1")
# Define storage client
storageClient <- rAzureBatch::StorageServiceClient[["new"]](
  authentication = storageCredentials,
  url            = sprintf("https://%s.blob.%s",
                           storageName,
                           "core.windows.net"
  )
)
print("part 2")
# Generate read SaS token
readSasToken <- storageClient[["generateSasToken"]](permission = "r", "c", path = containerName)

print("part 3")
# Get the blob url for the  dataset
blobUrl <- rAzureBatch::createBlobUrl(storageAccount = storageName,
                                      containerName  = containerName,
                                      sasToken       = readSasToken,
                                      fileName       = "input/openoptys_2018.csv")
# Read file using the blob url
print("part 4")
Dataset <- read.csv(blobUrl)

# Create a local temporary folder and save the result of the analysis
dir.create("output")
print("part 5")
write.csv(Dataset, "output/openoptydata.csv", row.names = F)
# Generate write sas token
writeSasToken <- storageClient[["generateSasToken"]](permission = "w", "c", path = containerName)

print("part 6")

# Upload dataset to the blob storage container

storageClient[["blobOperations"]][["uploadBlob"]](
  sasToken      = writeSasToken,
  containerName = paste0(containerName,"/output"),
  fileDirectory = "output/openoptydata.csv",
  accountName   = storageName
)
print("part 7")
