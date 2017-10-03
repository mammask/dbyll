---
layout: post
title: Correlation HeatMap
categories: [visualization]
tags: [analytics, visualization, R]
fullview: true
comments: true
---

In this section we provide an example of how to compute a correlation heatmap using **R**:

```{r}
CorrFunction <- function(metricDat,
                         featureVar,
                         interactiveFlag){
  
  # function name: CorrFunction
  #       purpose: To compute an interactive correlation heatmap between a set of features
  #       version: 1a
  #        inputs:
  #             - metricDat       : dataset with feature variables in numeric or integer format
  #             - featureVar      : a vector of the feature variables
  #             - interactiveFlag : if TRUE generate an interactive correlation heatmap
  #       outputs:
  #             - p         : an interactive correlation matrix of the selected features
  #          date: 10/02/2017
  #       contact: mammaskon@gmail.com
  
  library(data.table)
  library(ggplot2)
  library(plotly)
  library(stringr)
  
  corrData <- data.table(cor(metricDat[,featureVar, with = F]), keep.rownames = T)
  setnames(corrData, "rn", "Metric A")
  corrData <- data.table::melt(data        = corrData,
                               id.vars     = "Metric A",
                               value.name  = "Correlation", variable.name = "Metric B")
  
  corrData[, categ:= cut(Correlation, breaks=seq(from = -1, to = 1, by = 0.25))]
  corrData[, `Metric A`:= str_to_title(gsub("_", " ", `Metric A`))]
  corrData[, `Metric B`:= str_to_title(gsub("_", " ", `Metric B`))]
  
  properOrder <- corrData[, list(Min = min(Correlation)), by = categ][order(Min)][,categ]
  corrData[, categ := factor(categ, levels = properOrder)]
  corrData[, Correlation:= round(Correlation,2)]
  
  p <- ggplot(corrData, aes(x = `Metric A`, y = `Metric B`)) +
       geom_tile(aes(fill=categ),colour="black")  +
       scale_fill_brewer(palette = "RdYlGn",name="Correlation") + 
       xlab("") + ylab("") +
       geom_text(aes(label=paste0(format(Correlation, big.mark=",",
       scientific=FALSE))),size = 3, check_overlap = F) + 
       ggtitle("Correlation of feature variables") + 
       theme(panel.background = element_blank(),
             legend.text      = element_text(size=10),
             legend.title     = element_text(size=9),
             axis.text.y      = element_text(size=7),
             axis.text.x      = element_text(size=5),
             axis.line.x      = element_line(color="black"),
             axis.line.y      = element_line(color="black"))
  
  if (interactiveFlag == TRUE){ return(ggplotly(p)) } else { return(p)  }  
}
```
### Example

In the following example we generate a table of 100 variables sampled from the normal distribution and we comute the correlation matrix:

```{r}
library(data.table)
# Set seed
set.seed(42)
# Generate a random dataset
tbl        <- data.table(matrix(rnorm(1000),ncol = 10))
# Define the names of the features
featureVec <- paste0("V",1:10)
# Produce output
corrTbl    <- CorrFunction(metricDat = tbl, featureVar = featureVec, interactiveFlag = FALSE)
```
<img src="https://github.com/mammask/mammask.github.io/blob/master/plots/CorrPlot.png?raw=true">


