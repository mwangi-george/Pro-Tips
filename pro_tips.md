Pro Tips
================
2023-10-09

# Downloading data from google drive

``` r
library(googledrive)


## Drive Authentication
options(
  # whenever there is one account token found, use the cached token
  gargle_oauth_email = "your_email@gmail.com",
  # specify auth tokens should be stored in a hidden directory ".secrets"
  gargle_oauth_cache = "your-app-folder-name/.secrets"
)

# downloading
drive_download(
  file ="sheet_url",
  path = "data/name_for_the_downloaded_data.xlsx", 
  overwrite = T
)
```

# encrypting data

``` r
library(encryptr)

# The first step for the encryptr workflow is to create a pair of encryption keys. This uses the openssl package. 
# The public key is used to encrypt information and can be shared. The private key allows decryption of the encrypted information. 
# It requires a password to be set. This password cannot be recovered if lost. If the file is lost or overwritten, any data encrypted with the public key cannot be decrypted.

genkeys()

# encrypt and decrypt
diamonds %>%
  head() %>% 
  encrypt(price, clarity) %>% 
  decrypt(price, clarity)
```