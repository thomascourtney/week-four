# Week Four

## Excel & R

- Make sure to open the "csv" file with Excel instead of the Mac's "Numbers"
  - Easier and more familiar interface
  
- Go to the "Data" tab on the top of Excel
  - Hit the 'Filter' button at the top
  
- By hitting the arrows in the first rows, we can categorize these inputs based on what colomun we choose
  - In this instance, we chose by location (GEO_NAME)
  - Click 'Select All' to choose just one location
  - We chose 'Ottawa - Gatineau (Ontario part)'
  
- When selecting multiple cells, the Average, Count and Sum appears on the bottom of Excel
  - We can also use the command '=sum()' to get the sum of selected cells
  
- To create a graph, select for example 2 columns to be implemented in this graph
  - For this example we selected 4 rows out of those 2 coloumns
  - Hit the 'Inset' tab at the top, then select 'Recommended Charts' and chose whatever type of graph you would like
  

## Basic Counting and Plotting in R

- Import data from the web using the "RCurl" package

- Once we have the data in R, we can created some charts and execute come commands
 
   - Counting the number of documents by the published city 'table(documents$Newspaper.City)'
 
   - Plot that count 'barplot(counts, main="Cities", xlab="Number of Articles")'
  
   - You can make a new barplot based on other factors in this data set ( Year, Publication Year, Number of Articles)
   
- If you would like to see this barplot you can expand it in a pop-up window, from there this can be exported as a PNG or PDF file


## Voyant

- First attempt did not work "Failed attempt to create a corpus"

- I had to download the Excel file, the upload it into Voyant Tools

- Make sure before uploading the document, you select 'from cells in each row ' and choose the Content, Author and Title in the 'Options' section

- Your corpys id will be in the URL after '?corpus='
  - You can always share or return to your original corpus that you've created with this corpus id

- I find it very interesting when clicking on different words, the graphs change immensely and you can see the trends of the words by years

- A fully interactive widget can be created from Voyant Tools by using the 'embedded tool'
  - You have to save the html and implement it into your file
  

## AntConc

- Make sure to unzip files in folder provided
  - Make sure to load data into AntConc correctly
  - Tutorial: (https://programminghistorian.org/en/lessons/corpus-analysis-with-antconc)
  
- Load files into the application

- Search for items using the search bottom and press 'Start'

- Sometimes the app does take a while for things to load
  - Play around with the different options for different results

## Topic Models

- Had a tough time in the beginning downloading TMT
  - Had to download it via the link provided in the discord (https://github.com/shawngraham/topic-modeling-tool-1/blob/master/TopicModelingTool.zip)

- Also had a tough time extracting the files 
  - Hit 'Input Dir...' only one time

- Select output directory by hitting 'output'

- Make sure to install these two packages
  - 'Tiddyverse' & 'Tiddytext'

- Import csv file, put the data into a tribble, tiddy the data, then in this instance delete the stopwords from our data

- To look at how the data was transformed
  - Hit 'cb', then enter to see
  - To see the data with the digets removed, hit 'cb_df'

- We have to then create it into a matrix, to the build the topic model
  - cb_words <- tidy_cb %>%
  count(id, word, sort = TRUE)
  
  - head(cb_words)
  
  - dtm <- cb_words %>%
  cast_dtm(id, word, n)
  
  - require(topicmodels)
  
  - K <- 15
  
  - set.seed(9161)
  
  - topicModel <- LDA(dtm, K, method="Gibbs", control=list(iter = 500, verbose = 25))




- In this example, every document will have 15 topics, we will rearrange to get the top 5 terms per topic (the 5 words will give an idea of the topic)
have a look a some of the results (posterior distributions)
tmResult <- posterior(topicModel)

format of the resulting object
attributes(tmResult)

lengthOfVocab
ncol(dtm)

topics are probability distributions over the entire vocabulary
beta <- tmResult$terms   # get beta from results
dim(beta)

for every document we have a probability distribution of its contained topics
theta <- tmResult$topics
dim(theta)        

top5termsPerTopic <- terms(topicModel, 5)
topicNames <- apply(top5termsPerTopic, 2, paste, collapse=" ")
topicNames




- Now visualizing the patterns

load libraries for visualization
library("reshape2")
library("ggplot2")

select some documents for the purposes of
sample visualizations
here, the 2nd, 100th, and 200th document
in our corpus

exampleIds <- c(2, 100, 200)

N <- length(exampleIds)
get topic proportions form example documents
topicProportionExamples <- theta[exampleIds,]
colnames(topicProportionExamples) <- topicNames

put the data into a dataframe just for our visualization
vizDataFrame <- melt(cbind(data.frame(topicProportionExamples), document = factor(1:N)), variable.name = "topic", id.vars = "document")  

specify the geometry, aesthetics, and data for a plot
ggplot(data = vizDataFrame, aes(topic, value, fill = document), ylab = "proportion") +
  geom_bar(stat="identity") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +  
  coord_flip() +
  facet_wrap(~ document, ncol = N)



- We can now see our topic model (topic on Y-axis, value on X-axis)

topics over time
append decade information for aggregation
cb$decade <- paste0(substr(cb$date, 0, 3), "0")
get mean topic proportions per decade
topic_proportion_per_decade <- aggregate(theta, by = list(decade = cb$decade), mean)
set topic names to aggregated columns
colnames(topic_proportion_per_decade)[2:(K+1)] <- topicNames

reshape data frame, for when I get the topics over time thing sorted
vizDataFrame <- melt(topic_proportion_per_decade, id.vars = "decade")

plot topic proportions per deacde as bar plot
require(pals)
ggplot(vizDataFrame, aes(x=decade, y=value, fill=variable)) +
  geom_bar(stat = "identity") + ylab("proportion") +
  scale_fill_manual(values = paste0(alphabet(20), "FF"), name = "decade") +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))

- When we chose 15 topic models in a previous step, lets go back a change it to more and also less topic models and see how the model changes with different inputs
