# README
Baxter Lowell

<link href="README_files/libs/htmltools-fill-0.5.9/fill.css" rel="stylesheet" />
<script src="README_files/libs/htmlwidgets-1.6.4/htmlwidgets.js"></script>
<script src="README_files/libs/plotly-binding-4.12.0/plotly.js"></script>
<script src="README_files/libs/setprototypeof-0.1/setprototypeof.js"></script>
<script src="README_files/libs/typedarray-0.1/typedarray.min.js"></script>
<script src="README_files/libs/jquery-3.5.1/jquery.min.js"></script>
<link href="README_files/libs/crosstalk-1.2.2/css/crosstalk.min.css" rel="stylesheet" />
<script src="README_files/libs/crosstalk-1.2.2/js/crosstalk.min.js"></script>
<link href="README_files/libs/plotly-htmlwidgets-css-2.25.2/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="README_files/libs/plotly-main-2.25.2/plotly-latest.min.js"></script>

``` r
# read in data (both USA and global oil exports)
#| echo: FALSE

library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.1
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.2
    ✔ purrr     1.2.1     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(readr)
library(dplyr)

global_exports_oil <- read.csv(here::here("data/global_oil_exports.csv"), row.names = NULL)

usa_exports <- read.csv(here::here("data/trade_data.csv"), row.names = NULL)

# cleaning up USA oil exports data

usa_exports_clean = usa_exports %>%
  select(c("reporterISO","reporterDesc", "partnerDesc", "partnerISO", "cmdDesc", "altQty", "fobvalue")) %>%
  rename(
    "Exporter" = "reporterISO",
    "Importer" = "partnerISO",
    "Product" = "cmdDesc",
    "quantity_bbl" = "altQty",
    "value_usd" = "fobvalue"
  )%>%
  mutate(
    partnerDesc = recode(partnerDesc, 
                     "Korea" = "Rep. of Korea"
                     )
  )
  

global_exports_clean = global_exports_oil %>%
   select(c("reporterISO", "partnerISO", "reporterDesc", "partnerDesc", "cmdDesc", "altQty", "fobvalue")) %>%
  rename(
    "Exporter" = "reporterISO",
    "Importer" = "partnerISO",
    "Product" = "cmdDesc",
    "quantity_bbl" = "altQty",
    "value_usd" = "fobvalue"
  )%>%
  mutate(
    partnerDesc = recode(partnerDesc, 
                     "Korea" = "Rep. of Korea"
                     )
  )%>%
   slice(-560)
```

### Data

For this project I am looking at global crude oil trade data from 2023.
My data comes from the United Nations Comtrade Data set, which is a
comprehensive database for global trade, including hundreds of products,
imports and exports, and more. I also am using the Happy Planet Index to
introduce some additional variables.

### Questions

I want to explore both trade networks of crude oil for both the United
States and the entire world. Some basic questions I am looking to answer
are; Who does the US export the most oil to? Does exporting oil help to
lead to a better quality of life? What do trade networks look like for
individual countries?

### Graph on US Oil Exports

``` r
usa_exports_clean %>%
  filter(Importer != "W00") %>%
  filter(partnerDesc != "Other Asia, nes")%>%
  arrange(desc(quantity_bbl))%>%
  slice_head(n = 20)%>%
  mutate(
    partnerDesc = fct_reorder(partnerDesc, quantity_bbl, .desc = TRUE))%>%

ggplot(data = ., aes(x = partnerDesc, y = quantity_bbl))+
  geom_col()+
  labs(title = "Top 20 US Crude Oil Export Destinations (2023)",
       x = "Partners",
       y = "Annual Barrels (BBL)",
       caption = "*Data from UN Comtrade Database 2023 Data"
       )+
  theme_bw()+
  coord_flip()
```

![](README_files/figure-commonmark/unnamed-chunk-2-1.png)

### Plotly Graph: Life Expectancy vs Barrels of Oil Exported

``` r
library(fuzzyjoin)
library(plotly)
```


    Attaching package: 'plotly'

    The following object is masked from 'package:ggplot2':

        last_plot

    The following object is masked from 'package:stats':

        filter

    The following object is masked from 'package:graphics':

        layout

``` r
hpi_df <- read_csv("https://raw.githubusercontent.com/highamm/ds234_quarto/main/data_online/hpi-tidy.csv")
```

    Rows: 151 Columns: 11

    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr (3): Country, GovernanceRank, Region
    dbl (8): HPIRank, LifeExpectancy, Wellbeing, HappyLifeYears, Footprint, Happ...

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
global_export_region <- stringdist_full_join(
  hpi_df,
  global_exports_clean,
  by = c("Country" = "reporterDesc"),
  max_dist = 2
)
# graph 2: Relating oil exports to life expectancy

global_export_region1 <- global_export_region %>%
  filter(Importer == "W00")%>%
  filter(quantity_bbl > 0)
  
region_plot <- ggplot(data = global_export_region1, aes(x = LifeExpectancy, y = quantity_bbl, color = Region, label = Country))+
  geom_point()+
  theme_bw()+
  scale_color_viridis_d()+
  labs(title = "Life Expectancy Based on Crude Oil Exports",
       x = " Life Expectancy",
       y = "Barrels Exported",
       caption = "*data from 2023 UN Comtrade Data & Happy Planet Index")

ggplotly(p = region_plot, tooltip = "label")
```

<div class="plotly html-widget html-fill-item" id="htmlwidget-861dd14b9be36ed34a33" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-861dd14b9be36ed34a33">{"x":{"data":[{"x":[73.5,73.5,73.5,69.400000000000006,74.200000000000003,68.700000000000003,74.099999999999994,74.099999999999994,74.099999999999994,74.099999999999994,74.099999999999994,74.099999999999994,74.099999999999994],"y":[47593325,1420845300,6365810000,2844781934.4000001,10367873490,4017782200,20,10,5,43000,40,76,681556705],"text":["Country: China","Country: China","Country: China","Country: Indonesia","Country: Malaysia","Country: Philippines","Country: Thailand","Country: Thailand","Country: Thailand","Country: Thailand","Country: Thailand","Country: Thailand","Country: Thailand"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(68,1,84,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(68,1,84,1)"}},"hoveron":"points","name":"East Asia","legendgroup":"East Asia","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[75.900000000000006,76.099999999999994,79.099999999999994,79.099999999999994,73.700000000000003,73.700000000000003,79.299999999999997,72.200000000000003,71.200000000000003,69.900000000000006,74],"y":[6623706300,3012270,47593325,1420845300,28567776.699999999,28567776.699999999,91.552000000000007,5900.5,87758700,6365810000,702376.91000000003],"text":["Country: Argentina","Country: Belize","Country: Chile","Country: Chile","Country: Colombia","Country: Colombia","Country: Costa Rica","Country: El Salvador","Country: Guatemala","Country: Guyana","Country: Peru"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(68,58,131,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(68,58,131,1)"}},"hoveron":"points","name":"Latin America","legendgroup":"Latin America","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[74.599999999999994,75.900000000000006,74.5],"y":[82006660000,25,1312046520],"text":["Country: Kuwait","Country: Syria","Country: Tunisia"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(49,104,142,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(49,104,142,1)"}},"hoveron":"points","name":"Middle East and North Africa","legendgroup":"Middle East and North Africa","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[65.200000000000003,65.400000000000006],"y":[275000,43640],"text":["Country: Myanmar","Country: Pakistan"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(33,144,140,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(33,144,140,1)"}},"hoveron":"points","name":"South Asia","legendgroup":"South Asia","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[51.100000000000001,53.200000000000003,51.600000000000001,51.600000000000001,57.399999999999999,57.399999999999999,55.399999999999999,64.200000000000003,64.200000000000003,57.100000000000001,54.200000000000003,51.399999999999999,58.600000000000001,62.5,62.5,62.5,54.700000000000003,51.899999999999999,55.399999999999999,59.299999999999997,57.100000000000001,57.100000000000001,54.100000000000001,49,49,49],"y":[386425653.80000001,568.75,3083818500,3083818500,13951740200,26,1115941190,1420845300,6365810000,2970,6,6,7.5499999999999998,6721.6899999999996,663,663,98,98,8500,63690165,13951740200,26,8500,6721.6899999999996,663,663],"text":["Country: Angola","Country: Botswana","Country: Cameroon","Country: Cameroon","Country: Congo","Country: Congo","Country: Cote d'Ivoire","Country: Ghana","Country: Ghana","Country: Kenya","Country: Malawi","Country: Mali","Country: Mauritania","Country: Namibia","Country: Namibia","Country: Namibia","Country: Niger","Country: Nigeria","Country: Rwanda","Country: Senegal","Country: Togo","Country: Togo","Country: Uganda","Country: Zambia","Country: Zambia","Country: Zambia"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(53,183,121,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(53,183,121,1)"}},"hoveron":"points","name":"Sub Saharan Africa","legendgroup":"Sub Saharan Africa","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,70.700000000000003,76.599999999999994,74.400000000000006,67,73.299999999999997,73.299999999999997,72.200000000000003,68.5,74,74,74,74.5,75.400000000000006,75.400000000000006,79.299999999999997,79.299999999999997,68.5],"y":[26047249100,39196600,266569000,975166000,1135960000,199007000,1341020000,743513000,221085000,630056000,2270340000,11117500000,48042500,449879000,621948000,2025080000,138338000,554367000,1221970000,48688000,106275000,1202210000,691039000,524839089,271429467,70665926869,3688726,3688726,32491976,4730370,33984209.600000001,1.7,33984207.899999999,25,1503530,1046.8299999999999,1503530,1046.8299999999999,16321050],"text":["Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Azerbaijan","Country: Croatia","Country: Hungary","Country: Kazakhstan","Country: Latvia","Country: Latvia","Country: Lithuania","Country: Mongolia","Country: Romania","Country: Romania","Country: Romania","Country: Serbia","Country: Slovakia","Country: Slovakia","Country: Slovenia","Country: Slovenia","Country: Ukraine"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(143,215,68,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(143,215,68,1)"}},"hoveron":"points","name":"Transition States","legendgroup":"Transition States","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[81.900000000000006,80.900000000000006,80,80,80,81.5,81.5,81.5,81.5,80.400000000000006,80.400000000000006,79.900000000000006,81.799999999999997,80.599999999999994,80,80,80,80,81.099999999999994,81.099999999999994,81.099999999999994,81.099999999999994,81.099999999999994,81.099999999999994,81.099999999999994,81.099999999999994,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,81.400000000000006,82.299999999999997,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003,80.200000000000003],"y":[14880141230.07,14880141230.07,11309230.58,6,6,152,1864,15000,119,2640400,2640400,125839000,146321911,146321911,3.2999999999999998,0.29999999999999999,2,1,293307000,1357390000,67057400,204278000,166950000,242627000,936167000,146979000,13.800000000000001,200,1286371673.4400001,400,32932700,276949300,31786000,33000800,31255916,32356900,301,260846660,26,674914100,1666261100,483933000,779410200,2305995400,3929837503,79689900,967525160,1158942000,432462900,52280600,14354305700,146275310,3194366700,710,404365800,1702984060,150,595719400],"text":["Country: Australia","Country: Austria","Country: Belgium","Country: Finland","Country: Finland","Country: France","Country: France","Country: France","Country: France","Country: Germany","Country: Germany","Country: Greece","Country: Iceland","Country: Ireland","Country: Luxembourg","Country: Luxembourg","Country: Luxembourg","Country: Luxembourg","Country: Norway","Country: Norway","Country: Norway","Country: Norway","Country: Norway","Country: Norway","Country: Norway","Country: Norway","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Spain","Country: Switzerland","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom","Country: United Kingdom"],"type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":"rgba(253,231,37,1)","opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":"rgba(253,231,37,1)"}},"hoveron":"points","name":"Western World","legendgroup":"Western World","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"y":[5,17639400,35302195.899999999,625804,2819599248,5,249566455,368.89999999999998,8895,2414205,45838450,182843600,90,1090843,1482247512,14400,2414205],"text":"Country: NA","type":"scatter","mode":"markers","marker":{"autocolorscale":false,"color":["transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent"],"opacity":1,"size":5.6692913385826778,"symbol":"circle","line":{"width":1.8897637795275593,"color":["transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent","transparent"]}},"hoveron":"points","name":"NA","legendgroup":"NA","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":40.840182648401829,"r":7.3059360730593621,"b":37.260273972602747,"l":54.794520547945211},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.611872146118724},"title":{"text":"Life Expectancy Based on Crude Oil Exports","font":{"color":"rgba(0,0,0,1)","family":"","size":17.534246575342465},"x":0,"xref":"paper"},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[47.335000000000001,83.965000000000003],"tickmode":"array","ticktext":["50","60","70","80"],"tickvals":[50,60,70,80],"categoryorder":"array","categoryarray":["50","60","70","80"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.6529680365296811,"tickwidth":0.66417600664176002,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.68949771689498},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176002,"zeroline":false,"anchor":"y","title":{"text":" Life Expectancy","font":{"color":"rgba(0,0,0,1)","family":"","size":14.611872146118724}},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[-4100332999.6849999,86106992999.985001],"tickmode":"array","ticktext":["0e+00","2e+10","4e+10","6e+10","8e+10"],"tickvals":[0,20000000000,40000000000,60000000000.000008,80000000000],"categoryorder":"array","categoryarray":["0e+00","2e+10","4e+10","6e+10","8e+10"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.6529680365296811,"tickwidth":0.66417600664176002,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.68949771689498},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176002,"zeroline":false,"anchor":"x","title":{"text":"Barrels Exported","font":{"color":"rgba(0,0,0,1)","family":"","size":14.611872146118724}},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"rgba(255,255,255,1)","line":{"color":"rgba(51,51,51,1)","width":0.66417600664176002,"linetype":"solid"},"yref":"paper","xref":"paper","layer":"below","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.8897637795275593,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.68949771689498},"title":{"text":"Region","font":{"color":"rgba(0,0,0,1)","family":"","size":14.611872146118724}}},"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"source":"A","attrs":{"179c5fce3594":{"x":{},"y":{},"colour":{},"label":{},"type":"scatter"}},"cur_data":"179c5fce3594","visdat":{"179c5fce3594":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.20000000000000001,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
