# Visualizing horizontal gene transfer (HGT) events with an alluvial plot

> This is a tutorial demonstrating how I created an alluvial plot to visualize metaCHIP results.  

### Step 1: Run MetaCHIP

See [this link](https://github.com/emilieskoog/SharkBay2023_analysis/tree/main/Community_interactions_through_HGT_analysis) for running metaCHIP.



### Step 2: Analyze and visualize in R 

Now we take the following files and import them into R for analyses:

Output file from metaCHIP: `SharkBay_c14_HGTs_PG.txt`
Gene annotation file: `metachip_class_HGTs_prokka_bacterial_results_PROKKA_08032022.csv`

#### Set working directory and import into R

```
library(reshape)
library(ggplot2)
library(readr)
library(tidyselect)
library(tidyverse)
library(dplyr)
library(ggalluvial)
library(viridis)
library(writexl)
library(cowplot)
setwd("/your/working/directory/")

metachip_prokka_class<- read.csv("metachip_class_HGTs_prokka_bacterial_results_PROKKA_08032022.csv", header = FALSE) 
transfer_class <- read.delim("SharkBay_c14_HGTs_PG.txt") 
```
#### Split headers of annotation file
```
metachip_prokka_class_split_headers <- metachip_prokka_class %>% 
  separate(V9, sep=";", into = c("locus_tag", "Name", "Protein",
                                 "Product", "Function"), remove=TRUE, convert = FALSE)%>%
  
  data.frame
```
#### Clean file up
```
metachip_prokka_class_simplified <- metachip_prokka_class_split_headers[,-c(2:8, 10:11,13) ]

metachip_prokka_class_simplified_cleaned <- metachip_prokka_class_simplified %>% 
  mutate(locus_tag = gsub('ID=', '', locus_tag)) %>% 
  mutate(Product = gsub('product=', '', Product)) %>% 
  data.frame
  
colnames(metachip_prokka_class_simplified_cleaned)[1] <- "Gene_1" 
  
```


Now,  your `metachip_prokka_class_simplified_cleaned` file should look like:

![](https://i.imgur.com/8K2K4AB.png)

Your `transfer_class` file should look like:

![](https://i.imgur.com/dA9JtFU.png)


#### Merge files

Merge these files by the `Gene_1` column
```
merged_files <- merge(metachip_prokka_class_simplified_cleaned, transfer_class, by= "Gene_1")

```
#### Define taxa groups
```
Determining_transfer_agent_1_class <- merged_files %>%
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


```
And remove a quaotation that remaied at the end of each product name:
```
Determining_transfer_agent_2_class <- Determining_transfer_agent_2_class %>% 
  mutate(Product = gsub('"', '', Product)) %>% 
  data.frame
```
Your file should now look like:

![](https://i.imgur.com/UVKKYTN.png)

#### Assign certain gene products as stress-related or not ("other")

```
Stress_diagnosis <- Determining_transfer_agent_2_class %>%
  mutate(Environmental_stressor = case_when(
    grepl(pattern = "Catalase|Catalase-peroxidase|Superoxide dismutase|Thioredoxin|Peroxiredoxin|Glutaredoxin|Rubrerythrin|Glutathione", x = Product) ~ "Oxidative Stress",
    grepl(pattern = "Arsenate reductase|Manganese transport system ATP-binding protein MntB|Manganese transport system ATP-binding protein|Manganese-binding lipoprotein MntA|Tungstate|Manganese|Arsenical-resistance", x = Product) ~ "Heavy Metal Toxicity",
    grepl(pattern = "Chaperone protein ClpB|60 kDa chaperonin", x = Product) ~ "Heat Shock",
    grepl(pattern = "DNA protection during starvation protei|Polyphosphate|Phosphate import|Bacterioferritin", x = Product) ~ "Nutrient limitation",
    grepl(pattern = "Nitrogen regulatory protein P-II", x = Product) ~ "Nitrogen sensing",
    grepl(pattern = "Glycine betaine-binding periplasmic protein|Glycine cleavage system H protein|Glycine betaine-binding periplasmic protein|Glycine/sarcosine N-methyltransferase|Osmoregulated proline transporter OpuE|translocating NADH-quinone reductase|Glycine betaine/proline/choline transporter|Osmoprotectant", x = Product) ~ "Osmotic Stress",
    grepl(pattern = "Multidrug resistance protein MdtB|Efflux pump membrane|multidrug|Virginiamycin A acetyltransferase|N-ethylmaleimide|Lactoylglutathione lyase|Bacitracin", x = Product) ~ "Antitoxicity/ Antibiotic resistance",
    grepl(pattern = "N-acetylgalactosamine-6-O-sulfatase|Trehalase|Trehalose", x = Product) ~ "EPS-related",
    grepl(pattern = "UvrABC system protein B", x = Product) ~ "DNA repair",
  ))

Stress_diagnosis[is.na(Stress_diagnosis)] <- "Other"
```
#### Summarize number of each gene in each category 
```
Stress_diagnosis$number <- 1

summed_stressors<- Stress_diagnosis %>%
  group_by(Environmental_stressor, Transfer_agent_1_class, Transfer_agent_2_class) %>%
  summarize(sum_number = sum(number))
 
reduced_summed_stressors_df <- summed_stressors[!grepl("Other", summed_stressors$Environmental_stressor),]

###remove any column where the transfer is within the same phylum  

reduced_summed_stressors_df <- reduced_summed_stressors_df[reduced_summed_stressors_df$Transfer_agent_1_class != reduced_summed_stressors_df$Transfer_agent_2_class,]

```

#### More broad stress "diagnosis"
```
Stress_diagnosis_2 <- Stress_diagnosis %>%
  mutate(Stress_or_not = case_when(
    grepl(pattern = "Oxidative Stress|Heavy Metal Toxicity|Heat Shock|Nutrient limitation|Osmotic Stress|Nitrogen|Antibiotic|EPS|DNA", x = Environmental_stressor) ~ "Environmental stress-related gene",
    grepl(pattern = "Other", x = Environmental_stressor) ~ "Other",  
  ))

Stress_diagnosis_2$number2 <- 1

summed_stressors2<- Stress_diagnosis_2 %>%
  group_by(Stress_or_not, Transfer_agent_1_class, Transfer_agent_2_class) %>%
  summarize(sum_number2 = sum(number2))

### remove any column where the transfer is within the same phylum
summed_stressors2 <- summed_stressors2[summed_stressors2$Transfer_agent_1_class != summed_stressors2$Transfer_agent_2_class,]
```

#### Create alluvial plots
```
alluvial_plot1 <- ggplot(as.data.frame(summed_stressors2),
                         aes(y = sum_number2, axis1 = Transfer_agent_1_class, axis2 = Transfer_agent_2_class)) + 
  geom_alluvium(aes(fill = Stress_or_not), width = 1/12) +
  geom_stratum(width = 1/12, alpha = 1, fill = "black", color = "grey") +
  geom_label(stat = "stratum", aes(label = after_stat(stratum))) +
  scale_x_discrete(limits = c("Transfer_agent_1_class", "Transfer_agent_2_class"), expand = c(.05, .05)) + 
  scale_fill_manual(values=c("red", "gray50")) +
  ylab("All HGT Counts") 


alluvial_plot2 <- ggplot(as.data.frame(reduced_summed_stressors_df),
                         aes(y = sum_number, axis1 = Transfer_agent_1_class, axis2 = Transfer_agent_2_class)) + 
  geom_alluvium(aes(fill = Environmental_stressor), width = 1/12) +
  geom_stratum(width = 1/12, alpha = 1, fill = "black", color = "grey") +
  geom_label(stat = "stratum", aes(label = after_stat(stratum))) + geom_text(stat = "stratum", color="white",label.strata = TRUE,
                                                                             angle=c(90,90,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0), size=10) +
  scale_x_discrete(limits = c("Transfer_agent_1_class", "Transfer_agent_2_class"), expand = c(.05, .05)) + 
  scale_fill_manual(values=c("#EF6A3F", "#A12531", "#00B7BD", "#9CA0D6", "#FFD100", "#B7BF10", "#476AA7", "#145178")) +
  ylab("Counts of stress-related HGT genes")
```
#### Plot alluvial plots 
```  
plot_grid(alluvial_plot1, alluvial_plot2, labels = c('A', 'B'))
```
![](https://i.imgur.com/X5GGWQh.jpg)
