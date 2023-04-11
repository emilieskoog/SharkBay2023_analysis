# Inferring community interaction through horizontal gene transfer (HGT) analysis

> This is a tutorial demonstrating how I used metaCHIP and iGraph to look at gene transfer events specifically between my Sumerlaeota and Hydrogenedentota MAGs and the rest of my microbial community from a pustular mat from Shark Bay, Australia.  

### Step 1: MetaCHIP

You will need to give the program the location of your MAGs and a GTDBtk classification file (GTDB_classifications.tsv). See [here](https://github.com/songweizhi/MetaCHIP) for more information about installing and using MetaCHIP.
```
MetaCHIP PI -p SharkBay -r pc -t 12 -i /Users/location/of/my/MAGs -x fa -taxon /Users/location/of/GTDB_classifications.tsv

MetaCHIP BP -p SharkBay -r pc -pfr -t 12 
```

### Step 2: Analyze and visualize in R 

Now we take our output file of `SharkBay_c14_HGTs_PG.txt` and import it into R.
```
install.packages("igraph")
install.packages("network")
install.packages("sna")
install.packages("ndtv")
install.packages("GGally")

library(reshape)
library(ggplot2)
library(readr)
library(tidyselect)
library(tidyverse)
library(dplyr)
library(tidyr)
library(igraph)
library(tidygraph)
library(ggraph)
library(GGally)
library(network)
library(sna)
library(ggplot2)

#set your working directory
setwd("/Users/emilieskoog/Desktop/")

#import files 

transfer_class <- read.delim("SharkBay_c14_HGTs_PG.txt") 

View(transfer_class)


####################CLASS##########################


Determining_transfer_agent_1_class <- transfer_class %>%
  mutate(Transfer_agent_1_class = case_when(Gene_1_group == 'A' ~ 'Alphaproteobacteria',
                                            Gene_1_group == 'B' ~ 'Chloroflexi',
                                            Gene_1_group == 'C' ~ 'Cyanobacteria',
                                            Gene_1_group == 'D' ~ 'BRC1',
                                            Gene_1_group == 'E' ~ 'Bacteroidetes',
                                            Gene_1_group == 'F' ~ 'Hydrogenedentes',
                                            Gene_1_group == 'G' ~ 'Planctomycetes',
                                            Gene_1_group == 'H' ~ 'Planctomycetes',
                                            Gene_1_group == 'I' ~ 'Planctomycetes',
                                            Gene_1_group == 'K' ~ 'Verrucomicrobia',
                                            Gene_1_group == 'L' ~ 'Myxococcota',
                                            Gene_1_group == 'M' ~ 'Myxococcota',
                                            Gene_1_group == 'N' ~ 'Gammaproteobacteria'))


View(Determining_transfer_agent_1_class)

Determining_transfer_agent_2_class <- Determining_transfer_agent_1_class %>%
  mutate(Transfer_agent_2_class = case_when(Gene_2_group == 'A' ~ 'Alphaproteobacteria',
                                             Gene_2_group == 'B' ~ 'Chloroflexi',
                                             Gene_2_group == 'C' ~ 'Cyanobacteria',
                                             Gene_2_group == 'D' ~ 'BRC1',
                                             Gene_2_group == 'E' ~ 'Bacteroidetes',
                                             Gene_2_group == 'F' ~ 'Hydrogenedentes',
                                             Gene_2_group == 'G' ~ 'Planctomycetes',
                                             Gene_2_group == 'H' ~ 'Planctomycetes',
                                             Gene_2_group == 'I' ~ 'Planctomycetes',
                                             Gene_2_group == 'K' ~ 'Verrucomicrobia',
                                             Gene_2_group == 'L' ~ 'Myxococcota',
                                             Gene_2_group == 'M' ~ 'Myxococcota',
                                             Gene_2_group == 'N' ~ 'Gammaproteobacteria'))

View(Determining_transfer_agent_2_class)

Determining_transfer_agent_2_class$total <- 1

Determining_transfer_agent_2_class <- Determining_transfer_agent_2_class[,-c(1:8)]

Summarized_df_for_HGT_between_transfers_class <- Determining_transfer_agent_2_class %>%
  group_by(Transfer_agent_1_class, Transfer_agent_2_class) %>%
  summarize(Total_HGT_events = sum(total))

View(Summarized_df_for_HGT_between_transfers_class)

#add in vertex sizes
Summarized_df_for_HGT_between_transfers_class_added <- Summarized_df_for_HGT_between_transfers_class %>%
  mutate(MAG_number = case_when(
    grepl(pattern = "Bacteroidetes", x = Transfer_agent_2_class) ~ 20,
    grepl(pattern = "BRC1", x = Transfer_agent_2_class) ~ 1,
    grepl(pattern = "Chloroflexi", x = Transfer_agent_2_class) ~ 2,
    grepl(pattern = "Cyanobacteria", x = Transfer_agent_2_class) ~ 3,
    grepl(pattern = "Hydrogenedentes", x = Transfer_agent_2_class) ~ 1,
    grepl(pattern = "Myxococcota", x = Transfer_agent_2_class) ~ 4,
    grepl(pattern = "Planctomycetes", x = Transfer_agent_2_class) ~ 11,
    grepl(pattern = "Gammaproteobacteria", x = Transfer_agent_2_class) ~ 7,
    grepl(pattern = "Alphaproteobacteria", x = Transfer_agent_2_class) ~ 27,
    grepl(pattern = "Verrucomicrobia", x = Transfer_agent_2_class) ~ 8,
  ))
View(Summarized_df_for_HGT_between_transfers_class_added)

#tangent for just BRC1 and hydrogenedentes

BRC1_Hydrogenedentes <- Summarized_df_for_HGT_between_transfers_class_added[grepl("Hydrogenedentes|BRC1", Summarized_df_for_HGT_between_transfers_class_added$Transfer_agent_2_class),]
View(BRC1_Hydrogenedentes)
BRC1_Hydrogenedentes_added <- BRC1_Hydrogenedentes %>%
  mutate(MAG_number = case_when(
    grepl(pattern = "Bacteroidetes", x = Transfer_agent_1_class) ~ 20,
    grepl(pattern = "BRC1", x = Transfer_agent_1_class) ~ 1,
    grepl(pattern = "Chloroflexi", x = Transfer_agent_1_class) ~ 2,
    grepl(pattern = "Cyanobacteria", x = Transfer_agent_1_class) ~ 3,
    grepl(pattern = "Hydrogenedentes", x = Transfer_agent_1_class) ~ 1,
    grepl(pattern = "Myxococcota", x = Transfer_agent_1_class) ~ 4,
    grepl(pattern = "Planctomycetes", x = Transfer_agent_1_class) ~ 11,
    grepl(pattern = "Gammaproteobacteria", x = Transfer_agent_1_class) ~ 7,
    grepl(pattern = "Alphaproteobacteria", x = Transfer_agent_1_class) ~ 27,
    grepl(pattern = "Verrucomicrobia", x = Transfer_agent_1_class) ~ 8,
  ))

BRC1_Hydrogenedentes_added[nrow(BRC1_Hydrogenedentes_added) + 1,] = c(list("Hydrogenedentes", "BRC1", 0, 1))
BRC1_Hydrogenedentes_added[nrow(BRC1_Hydrogenedentes_added) + 1,] = c(list("BRC1", "Hydrogenedentes", 0, 1))

View(BRC1_Hydrogenedentes_added)
```
![](https://i.imgur.com/rVNtCGk.png)

Now lets make a pretty plot and visualize the data!
```

links <- BRC1_Hydrogenedentes_added[,-c(4)]
View(links)
nodes <- BRC1_Hydrogenedentes_added[,-c(2,3)]
View(nodes)

names(nodes)[names(nodes) == "Transfer_agent_1_class"] <- "taxa"
nodes2 <- nodes[-c(1,3,5,8),]

net <- graph_from_data_frame(d=links, vertices=nodes2, directed=F) 
class(net)

plot(net, vertex.label=NA)
net <- simplify(net, remove.multiple = F, remove.loops = T) 
net <- simplify(net, edge.attr.comb=list(Total_HGT_events="sum","ignore"))

View(nodes2)
View(links)

E(net)$width <- E(net)$Total_HGT_events/10
V(net)$size <- V(net)$MAG_number
V(net)$label.color <- "black"
colrs <- c("orange", "tomato", "green", "cornflowerblue", "red", "gold", "steelblue", "aquamarine3", "pink", "blue")
plot(net, edge.color="black", vertex.color= colrs) 

```

![](https://i.imgur.com/vzSYM7B.png)

