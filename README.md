# Working-with-web-data-in-R-APIs

Web APIs
There is so much information on the internet about interacting with web APIs it can seem overwhelming. In this post, I am going to keep the explanation and demonstration extremely simple. We’ll use the R package httr for sending HTTP requests to an API server. And then we’ll use jsonlite for parsing the data we get back in the response.
A note about authentication: this tutorial will not touch on authentication. (That will be in part III where I deomonstrate how to pull running and cycling data from the Strava app.). The most common way to authenticate by far is OAuth. I could spend an entire post on OAuth, but for now we are going to just use some API endpoints out on the web that do not require any authentication method to access.

People in space right now

http://api.open-notify.org/ is a simple example of an API server. It has two end points: one that tells you where the International Space Station (ISS) is right now, and one that tells you who is in space at this moment. Let’s use the second endpoint.
First, we’ll use the GET function from httr to send an HTTP request to the server; then we can inspect the object that comes back.
library(tidyverse)
library(httr)
library(jsonlite)

# the name of the end point is called "astros."
req <- GET("http://api.open-notify.org/astros")
str(req)
## List of 10
##  $ url        : chr "http://api.open-notify.org/astros"
##  $ status_code: int 200
##  $ headers    :List of 6
##   ..$ server                     : chr "nginx/1.10.3"
##   ..$ date                       : chr "Sun, 04 Jul 2021 19:50:03 GMT"
##   ..$ content-type               : chr "application/json"
##   ..$ content-length             : chr "494"
##   ..$ connection                 : chr "keep-alive"
##   ..$ access-control-allow-origin: chr "*"
##   ..- attr(*, "class")= chr [1:2] "insensitive" "list"
##  $ all_headers:List of 1
##   ..$ :List of 3
##   .. ..$ status : int 200
##   .. ..$ version: chr "HTTP/1.1"
##   .. ..$ headers:List of 6
##   .. .. ..$ server                     : chr "nginx/1.10.3"
##   .. .. ..$ date                       : chr "Sun, 04 Jul 2021 19:50:03 GMT"
##   .. .. ..$ content-type               : chr "application/json"
##   .. .. ..$ content-length             : chr "494"
##   .. .. ..$ connection                 : chr "keep-alive"
##   .. .. ..$ access-control-allow-origin: chr "*"
##   .. .. ..- attr(*, "class")= chr [1:2] "insensitive" "list"
##  $ cookies    :'data.frame': 0 obs. of  7 variables:
##   ..$ domain    : logi(0) 
##   ..$ flag      : logi(0) 
##   ..$ path      : logi(0) 
##   ..$ secure    : logi(0) 
##   ..$ expiration: 'POSIXct' num(0) 
##   ..$ name      : logi(0) 
##   ..$ value     : logi(0) 
##  $ content    : raw [1:494] 7b 22 70 65 ...
##  $ date       : POSIXct[1:1], format: "2021-07-04 19:50:03"
##  $ times      : Named num [1:6] 0 0.00265 0.04969 0.04989 0.09569 ...
##   ..- attr(*, "names")= chr [1:6] "redirect" "namelookup" "connect" "pretransfer" ...
##  $ request    :List of 7
##   ..$ method    : chr "GET"
##   ..$ url       : chr "http://api.open-notify.org/astros"
##   ..$ headers   : Named chr "application/json, text/xml, application/xml, */*"
##   .. ..- attr(*, "names")= chr "Accept"
##   ..$ fields    : NULL
##   ..$ options   :List of 2
##   .. ..$ useragent: chr "libcurl/7.54.0 r-curl/4.3 httr/1.4.2"
##   .. ..$ httpget  : logi TRUE
##   ..$ auth_token: NULL
##   ..$ output    : list()
##   .. ..- attr(*, "class")= chr [1:2] "write_memory" "write_function"
##   ..- attr(*, "class")= chr "request"
##  $ handle     :Class 'curl_handle' <externalptr> 
##  - attr(*, "class")= chr "response"
We get a large list back with complicated information about the server, the HTTP headers, a status code (which is important), details about the request specifically, and something called content. Content is where the data lives. The status code was 200, so we know it was successful. (Check out a list of status codes and their descriptions here.)
We can wrap our req object around the content() function from the httr package. According to the documentation, the content() function has an as = argument that can take the following values: "raw", "text", or "parsed". Let’s see what each gives with a for loop.
args <- c("raw", "text", "parsed")

for (arg in args) {
  req_content <- content(req, as = arg)
  print(paste0("This is the ", arg, " content..."))
  if (typeof(req_content) == "list") {
    print(req_content[[1]][1:5]) # just cutting down the output by subsetting the list.
  } else {
    print(head(req_content, 5)) # just cutting down the output with head().
  }
}
## [1] "This is the raw content..."
## [1] 7b 22 70 65 6f
## [1] "This is the text content..."
## [1] "{\"people\": [{\"name\": \"Mark Vande Hei\", \"craft\": \"ISS\"}, {\"name\": \"Oleg Novitskiy\", \"craft\": \"ISS\"}, {\"name\": \"Pyotr Dubrov\", \"craft\": \"ISS\"}, {\"name\": \"Thomas Pesquet\", \"craft\": \"ISS\"}, {\"name\": \"Megan McArthur\", \"craft\": \"ISS\"}, {\"name\": \"Shane Kimbrough\", \"craft\": \"ISS\"}, {\"name\": \"Akihiko Hoshide\", \"craft\": \"ISS\"}, {\"name\": \"Nie Haisheng\", \"craft\": \"Tiangong\"}, {\"name\": \"Liu Boming\", \"craft\": \"Tiangong\"}, {\"name\": \"Tang Hongbo\", \"craft\": \"Tiangong\"}], \"number\": 10, \"message\": \"success\"}"
## [1] "This is the parsed content..."
## [[1]]
## [[1]]$name
## [1] "Mark Vande Hei"
## 
## [[1]]$craft
## [1] "ISS"
## 
## 
## [[2]]
## [[2]]$name
## [1] "Oleg Novitskiy"
## 
## [[2]]$craft
## [1] "ISS"
## 
## 
## [[3]]
## [[3]]$name
## [1] "Pyotr Dubrov"
## 
## [[3]]$craft
## [1] "ISS"
## 
## 
## [[4]]
## [[4]]$name
## [1] "Thomas Pesquet"
## 
## [[4]]$craft
## [1] "ISS"
## 
## 
## [[5]]
## [[5]]$name
## [1] "Megan McArthur"
## 
## [[5]]$craft
## [1] "ISS"
Depending on the data returned, you may want to use as = "text" or as = "parsed"; I don’t think you would ever want to use as = "raw" unless you were sending this to another process for encoding.
Let’s use as = "text" to demonstrate how jsonlite is used.
req_content <- content(req, as = "text")
people_list <- fromJSON(req_content, flatten = TRUE)
str(people_list)
## List of 3
##  $ people :'data.frame': 10 obs. of  2 variables:
##   ..$ name : chr [1:10] "Mark Vande Hei" "Oleg Novitskiy" "Pyotr Dubrov" "Thomas Pesquet" ...
##   ..$ craft: chr [1:10] "ISS" "ISS" "ISS" "ISS" ...
##  $ number : int 10
##  $ message: chr "success"
This looks like the same list that we got with as = "parsed" but this option does not work with all API data. It’s best in most instances to have the function return text and then parse that text with the fromJSON function from jsonlite.
Let’s subset this list and save it as a tibble. From there, we can begin working with the data!
people <- as_tibble(people_list$people)
people
## # A tibble: 10 x 2
##    name            craft   
##    <chr>           <chr>   
##  1 Mark Vande Hei  ISS     
##  2 Oleg Novitskiy  ISS     
##  3 Pyotr Dubrov    ISS     
##  4 Thomas Pesquet  ISS     
##  5 Megan McArthur  ISS     
##  6 Shane Kimbrough ISS     
##  7 Akihiko Hoshide ISS     
##  8 Nie Haisheng    Tiangong
##  9 Liu Boming      Tiangong
## 10 Tang Hongbo     Tiangong
theme_set(theme_minimal())

people %>% 
  count(craft) %>% 
  ggplot(aes(x = craft, y = n, fill = craft)) +
  geom_col() +
  scale_fill_viridis_d(option = "magma", begin = 0.4, end = 0.8) +
  coord_flip() +
  theme(panel.grid.minor = element_blank(),
        panel.grid.major.y = element_blank(),
        plot.title = element_text(face = "bold"), 
        legend.position = "none") +
  labs(
    title = "Number of people in space right now",
    subtitle = "By spacecraft",
    x = element_blank(),
    y = element_blank()
  )

Tune in next time for how to get data from web APIs that require authentication!

