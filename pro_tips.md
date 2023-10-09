Pro Tips
================
2023-10-09

- [Downloading Data from Google
  Drive](#downloading-data-from-google-drive)
- [Encrypting and Decrypting Data](#encrypting-and-decrypting-data)
- [Collecting Data using Shiny
  Surveys](#collecting-data-using-shiny-surveys)
- [Putting IP info at the bottom of a shiny
  App](#putting-ip-info-at-the-bottom-of-a-shiny-app)

# Downloading Data from Google Drive

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

# Encrypting and Decrypting Data

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

# Collecting Data using Shiny Surveys

``` r
# Load packages
pacman::p_load(shiny, shinysurveys, googledrive, googlesheets4)

options(
  # whenever there is one account token found, use the cached token
  gargle_oauth_email = TRUE,
  # specify auth tokens should be stored in a hidden directory ".secrets"
  gargle_oauth_cache = "your-app-folder-name/.secrets"
)

# Get the ID of the sheet for writing programmatically
# This should be placed at the top of your shiny app
sheet_id <- drive_get("sheet_url")$id

# Define questions in the format of a shinysurvey
survey_questions <- data.frame(
  question = c(
    "Post your feedback here"
  ),
  option = NA,
  input_type = "text",
  input_id = "app_feedback",
  dependence = NA,
  dependence_value = NA,
  required = TRUE
)

# Define shiny UI
ui <- fluidPage(
  surveyOutput(
    survey_questions,
    survey_title = "Hello There!",
    survey_description = "Please give us feedback on how we can improve this application to serve you better"
  )
)
server <- function(input, output, session) {
  renderSurvey()
  
  observeEvent(input$submit, {
    response_data <- getSurveyData()
    
    # Read our sheet
    values <- read_sheet(
      ss = sheet_id,
      sheet = "feedback"
    )
    
    # Check to see if our sheet has any existing data.
    # If not, let's write to it and set up column names.
    # Otherwise, let's append to it.
    if (nrow(values) == 0) {
      sheet_write(
        data = response_data,
        ss = sheet_id,
        sheet = "feedback"
      )
    } else {
      sheet_append(
        data = response_data,
        ss = sheet_id,
        sheet = "feedback"
      )
    }
    
    # Message indication successful submission
    showModal(modalDialog(title = "Submitted Successfully"))
    
    # Clear the survey responses after submission
    clearSurvey()
    
  })
  
  # Function to clear the survey responses
  clearSurvey <- function() {
    for (question_id in survey_questions$input_id) {
      updateTextInput(
        session,
        inputId = question_id,
        value = ""
      )
    }
  }
  
}

# Run the shiny application
shinyApp(ui, server)
```

# Putting IP info at the bottom of a shiny App

``` r
ui <- dashboardPage(
  header, 
  sidebar, 
  dashboardBody(
    body,
    div(
      id = "copyright_terms",
      style = "text-align: center; margin-top: 20px;",
      HTML('&copy; 2023 Company Name Ltd | All Rights Reserved')
    )
  ),
  skin = "red"
)
```
