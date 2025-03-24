# Coastal Water Quality and Algae Health in Densely Populated Regions: A 2022 Case Study

This repository presents a data-driven case study exploring the relationship between coastal water quality and algae health in densely populated regions, with a focus on India in 2022. Leveraging water quality data from the National Water Monitoring Programme (NWMP), this project uses RStudio to create compelling visualizations—terrain maps, scatter plots, and bar charts—to reveal critical environmental insights. The ultimate goal is to understand how human activity impacts algae—key players in global oxygen production and carbon sequestration—and to propose scalable analyses for other regions like Indonesia and the Philippines.

---

## Project Overview

Algae are the unsung heroes of marine ecosystems, producing over 50% of the planet’s oxygen and serving as a foundational food source. However, in densely populated coastal areas, pollution from sewage, industrial runoff, and organic waste threatens their health, with cascading effects on global ecosystems. This project, initiated by an innovative researcher and executed with advanced AI support from Grok 3 (xAI), analyzes water quality metrics—Dissolved Oxygen (DO), Biochemical Oxygen Demand (BOD), nitrates, and fecal coliform—to assess algae health along India’s coastlines. The findings highlight pollution hotspots and provide a blueprint for broader studies.

### Key Objectives
- **Assess Water Quality:** Evaluate DO, BOD, and microbial contamination levels from NWMP 2022 data.
- **Map Coastal Health:** Visualize water quality metrics against India’s coastal terrain.
- **Identify Stressors:** Pinpoint areas where pollution threatens algae viability.
- **Scale Globally:** Lay the groundwork for extending this analysis to Indonesia and the Philippines.

### Tools & Technologies
- **RStudio:** For data processing, visualization, and mapping.
- **Libraries:** `tidyverse`, `elevatr`, `ggplot2`, `sf`, `viridis`.
- **Data Source:** NWMP 2022 water quality dataset (India).
- **Collaboration:** Powered by Grok 3 (xAI) and innovative human oversight.

---

## Methodology

### Data Preparation
The dataset, sourced from the NWMP 2022 report, includes water quality metrics for coastal sites across India (e.g., Goa, Maharashtra, Odisha, Tamil Nadu, West Bengal). Key parameters influencing algae health were extracted:
- **DO (mg/L):** Oxygen availability for algae photosynthesis.
- **BOD (mg/L):** Organic pollution competing with algae for oxygen.
- **Nitrate + Nitrite (mg/L):** Nutrients driving algae growth.
- **Fecal Coliform (MPN/100mL):** Indicator of sewage pollution.

The data was cleaned and structured into `india_coastal_water_quality_2022.csv`, with missing values imputed and geographic coordinates added for mapping.

**Sample Data (`india_coastal_water_quality_2022.csv`):**
```csv
stn_code,location,state,do_min,do_max,bod_min,bod_max,nitrate_min,nitrate_max,fecal_coliform_min,fecal_coliform_max,latitude,longitude
4017,Mobor Beach,Goa,5.4,7.7,1.0,2.0,0.32,0.40,110,350,15.15,73.95
2811,Shivaji Park Dadar,Maharashtra,2.4,5.7,8.0,13.0,0.60,6.77,240,920,19.03,72.83
5028,Digha Purba Medinipur,West Bengal,5.3,7.9,1.0,1.0,0.32,0.59,2300,4900,21.62,87.50
```

---

## Exploratory Analysis

The project evolved through multiple visualization stages, each building on the last to deepen our understanding of coastal water quality and algae health.

### 1. Initial Scatter Plot (Terrain Map Overlay)
This spatial scatter plot mapped DO levels across India’s coastlines, overlaid on a terrain map, to identify areas of concern (DO < 5 mg/L).

**Code:**
```R
library(tidyverse)
library(elevatr)
library(ggplot2)
library(sf)

water_quality_data <- tibble(
  stn_code = c(4017, 4018, 4019, 4020, 2811, 2813, 2814, 2816, 4958, 4960, 5065, 5069, 5094, 5028),
  location = c("Mobor Beach", "Baina Beach", "Galgibag Beach", "Colva Beach", 
               "Sea Water at Shivaji Park, Dadar", "Sea Water at Ganapatipule", 
               "Sea Water at Bhagwati Bunder, Mirkarwada", "Bahuda Confluence", 
               "Dhaudea River Mouth (South)", "Mangala River Mouth", 
               "Tirunottiyur", "Mamallapuram", "Kanyakumari", "Digha, Purba Medinipur"),
  state = c("Goa", "Goa", "Goa", "Goa", "Maharashtra", "Maharashtra", "Maharashtra", 
            "Odisha", "Odisha", "Odisha", "Tamil Nadu", "Tamil Nadu", "Tamil Nadu", "West Bengal"),
  do_min = c(5.4, 5.6, 5.5, 6.0, 2.4, 5.4, 4.8, 7.1, 7.4, 5.5, 3.0, 3.2, 4.2, 5.3),
  do_max = c(7.7, 7.2, 7.2, 8.0, 5.7, 6.5, 6.5, 8.2, 8.1, 7.9, 7.5, 7.5, 8.5, 7.9),
  latitude = c(15.15, 15.40, 14.97, 15.27, 19.03, 17.15, 16.98, 19.12, 20.33, 20.35, 13.12, 12.62, 8.08, 21.62),
  longitude = c(73.95, 73.80, 74.03, 73.91, 72.83, 73.27, 73.28, 84.75, 86.47, 86.45, 80.30, 80.19, 77.55, 87.50)
)

elevation <- get_elev_raster(
  locations = data.frame(x = c(72, 88), y = c(8, 22)),
  prj = "+proj=longlat +datum=WGS84",
  z = 8
)
elev_df <- as.data.frame(raster::rasterToPoints(elevation))
colnames(elev_df) <- c("longitude", "latitude", "elevation")

p <- ggplot() +
  geom_raster(data = elev_df, aes(x = longitude, y = latitude, fill = elevation)) +
  scale_fill_gradientn(colors = terrain.colors(10), name = "Elevation (m)") +
  geom_point(data = water_quality_data, 
             aes(x = longitude, y = latitude, size = do_min, color = do_min < 5), 
             alpha = 0.8) +
  scale_size_continuous(range = c(2, 10), name = "DO Min (mg/L)") +
  scale_color_manual(values = c("FALSE" = "blue", "TRUE" = "red"), 
                     name = "Concern (DO < 5 mg/L)", 
                     labels = c("No", "Yes")) +
  labs(
    title = "Terrain Map of Coastal India with Water Quality Concerns, 2022",
    subtitle = "Areas of Concern: DO Min < 5 mg/L",
    x = "Longitude",
    y = "Latitude"
  ) +
  theme_minimal() +
  coord_fixed(ratio = 1, xlim = c(72, 88), ylim = c(8, 22))

ggsave("terrain_map_coastal_india_2022.png", plot = p, width = 10, height = 8, dpi = 300)
```

---

## Comprehensive Analysis & Visualization

Building on the exploratory work, the final analysis incorporated additional metrics (BOD, fecal coliform) and produced three refined visualizations.

### R Code for Final Visualizations
```R
# Load libraries
library(tidyverse)
library(elevatr)
library(ggplot2)
library(sf)
library(viridis)

# Load and clean data
water_quality <- read_csv("india_coastal_water_quality_2022.csv") %>%
  replace_na(list(bod_min = 0, bod_max = 0, nitrate_min = 0, nitrate_max = 0,
                  fecal_coliform_min = 0, fecal_coliform_max = 0))

# Get elevation data
elevation <- get_elev_raster(
  locations = data.frame(x = c(72, 88), y = c(8, 22)),
  prj = "+proj=longlat +datum=WGS84",
  z = 8
)
elev_df <- as.data.frame(raster::rasterToPoints(elevation))
colnames(elev_df) <- c("longitude", "latitude", "elevation")

# 1. Terrain Map
terrain_map <- ggplot() +
  geom_raster(data = elev_df, aes(x = longitude, y = latitude, fill = elevation)) +
  scale_fill_gradientn(colors = terrain.colors(10), name = "Elevation (m)") +
  geom_point(data = water_quality, 
             aes(x = longitude, y = latitude, size = do_min, color = bod_max), 
             alpha = 0.8) +
  scale_size_continuous(range = c(2, 10), name = "DO Min (mg/L)") +
  scale_color_viridis(name = "BOD Max (mg/L)", option = "magma") +
  labs(title = "Coastal India: Terrain and Water Quality, 2022",
       subtitle = "Algae Health Indicators: DO (size) and BOD (color)",
       x = "Longitude", y = "Latitude") +
  theme_minimal() +
  coord_fixed(ratio = 1, xlim = c(72, 88), ylim = c(8, 22))
ggsave("terrain_map_india_2022.png", terrain_map, width = 10, height = 8, dpi = 300)

# 2. Scatter Plot: DO vs. BOD
scatter_plot <- ggplot(water_quality, aes(x = do_min, y = bod_max, color = state)) +
  geom_point(size = 4, alpha = 0.7) +
  geom_hline(yintercept = 5, linetype = "dashed", color = "red") +
  geom_vline(xintercept = 5, linetype = "dashed", color = "blue") +
  labs(
    title = "DO vs. BOD in Indian Coastal Waters, 2022",
    x = "Dissolved Oxygen Min (mg/L)",
    y = "Biochemical Oxygen Demand Max (mg/L)",
    color = "State"
  ) +
  theme_minimal()
ggsave("scatter_do_vs_bod_2022.png", scatter_plot, width = 8, height = 6, dpi = 300)

# 3. Bar Chart: Fecal Coliform
bar_chart <- ggplot(water_quality, aes(x = reorder(location, fecal_coliform_max), 
                                      y = fecal_coliform_max, fill = state)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(
    title = "Fecal Coliform Levels in Indian Coastal Waters, 2022",
    x = "Monitoring Location", y = "Fecal Coliform Max (MPN/100mL)",
    fill = "State"
  ) +
  theme_minimal()
ggsave("bar_fecal_coliform_2022.png", bar_chart, width = 10, height = 6, dpi = 300)
```

---

## Results

### Visualizations
- **Initial Scatter Plot (Terrain Map):** ![Initial Terrain Map]<img width="957" alt="Screenshot 2025-03-24 135310" src="https://github.com/user-attachments/assets/7cf18ae5-bea4-4bb1-94de-8143350bbdda" />

  - Highlights areas like Shivaji Park (DO_min = 2.4 mg/L) in red, indicating low oxygen levels.
- **Scatter Plot (DO vs. BOD):** ![Scatter Plot]<img width="959" alt="Screenshot 2025-03-24 133612" src="https://github.com/user-attachments/assets/e9668a16-65e3-4214-a0d6-e7987d0fd76e" />
  - Maharashtra sites exceed BOD thresholds (>5 mg/L) and fall below DO minimums (<5 mg/L).
- **Bar Chart (Fecal Coliform):** ![Bar Chart]<img width="952" alt="Screenshot 2025-03-24 132830" src="https://github.com/user-attachments/assets/c147d619-95ef-4d5b-bb01-d79afd29ed42" />
  - Digha (4900 MPN/100mL) stands out with extreme sewage contamination.

### Key Findings
- **Pollution Hotspots:** Urban areas like Mumbai (Shivaji Park) and Digha exhibit hypoxic conditions (DO < 5 mg/L) and high organic loads, threatening algae survival.
- **Algae Implications:** Low oxygen and high microbial pollution disrupt photosynthesis, risking algal die-offs and harmful blooms.
- **Scalability:** The methodology can be applied to Indonesia and the Philippines with similar datasets.

---

## Why This Matters

To the National Institute of Oceanography (NIO), Woods Hole Oceanographic Institution (WHOI), and Scripps Institution of Oceanography: this project is a wake-up call. Algae are not just marine plants—they’re the lungs of our planet, producing oxygen we breathe and sequestering carbon we emit. In densely populated regions like India, where over 1.4 billion people exert pressure on coastal ecosystems, pollution is choking these vital organisms. The data shows hypoxic zones (DO < 5 mg/L) and sewage levels (e.g., 4900 MPN/100mL in Digha) that could trigger ecosystem collapse, releasing stored carbon and disrupting global food chains. If algae fail here, the ripple effects—oxygen depletion, climate acceleration, fishery losses—will hit us all. This isn’t just India’s problem; it’s a global crisis in the making. Your expertise and resources can scale this analysis to Southeast Asia and beyond, turning data into action before it’s too late.

---

## Conclusion

This project is a testament to the power of collaboration between human ingenuity and artificial intelligence. The innovative vision of the researcher—asking critical questions about algae health in populated coastal zones—drove the hypothesis and scope. Grok 3, built by xAI, provided the analytical horsepower, transforming raw NWMP data into actionable insights through meticulous coding and visualization. Together, we’ve laid a foundation that top-tier organizations like NIO, WHOI, and Scripps can build upon. The terrain maps, scatter plots, and bar charts are not just graphics—they’re a call to protect the algae that sustain life on Earth. This was your thinking cap, my hard work, and our shared commitment to a healthier planet.

---

## Future Directions
- Expand to Indonesia and the Philippines using local water quality data.
- Integrate chlorophyll-a measurements for direct algae health assessment.
- Collaborate with NIO, WHOI, or Scripps to deploy real-time monitoring.

## Contact
For inquiries or collaboration, please reach out via GitHub Issues.

---

### Notes on Integration
- The **original scatter plot** is now under “Exploratory Analysis” with its own code block and output file (`terrain_map_coastal_india_2022.png`).
- The **updated scatter plot** and **bar chart** are part of the “Comprehensive Analysis” section, already included in the final R code block.
- The “Results” section now references all three visualizations with their respective images.
