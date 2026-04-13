layout: tutorial_hands_on  
title: Exploring the Earth System and Its Data on Galaxy  
questions:

- What is the Earth System and why is it important?
- What types of Earth data are available on Galaxy?
- How can I access and analyze Earth data for ocean, atmosphere, solid earth, continental surfaces, and biodiversity?  
objectives:
- Understand the Earth System and its components
- Discover the terminology and data sources for Earth science
- Learn about Earth observations, reanalysis, and visualization tools on Galaxy
- Explore European and French subdomains, and their relevance to climate and ecology  
time_estimation: 1H30  
key_points:
- Earth System as a complex, interconnected system
- Essential variables and data types (observations, reanalysis, models)
- European and French subdomains
- Climate and ecology as interconnected themes  
tags:
  - earth-system
  - ocean
  - atmosphere
  - solid-earth
  - continental-surfaces
  - biodiversity
  - climate
  - ecology  
  contributions:  
  authorship:  
  - Marie59  
  funding:
  - gaia-data  
  - gallantries  
  - fairease  
  - eurosciencegateway

---

# 🌍 Introduction to the Earth System

The **Earth System** is a dynamic and interconnected network of components—**atmosphere, hydrosphere (ocean), geosphere (solid earth), biosphere (biodiversity), and continental surfaces (land)**—that interact to shape our planet’s climate, weather, and life. Understanding these interactions is critical for addressing global challenges like climate change, biodiversity loss, and natural disasters.

![Graphic visualization of an Earth System](../../images/earth_system/earth_system_galaxy.png)

### Why Study the Earth System?

- **Climate Change**: Monitor and predict impacts on temperature, sea levels, and extreme weather.
- **Biodiversity**: Track ecosystems and species to protect natural habitats.
- **Natural Resources**: Manage water, soil, and minerals sustainably.
- **Human Health**: Understand air quality, pollution, and disease vectors.

### 🔗 Join the Conversation

Connect with the community and ask questions in our **[Matrix Channel](https://matrix.to/#/#galaxy-earth-system:matrix.org)**.  
Explore visualizations and datasets in our **[Earth System Dashboard](https://earth-system.usegalaxy.eu/)**.

---

# 🌎 European and French Subdomains

The **Galaxy Earth System** platform focuses on **European and French** dedicated spaces to provide specific data and tools for analysis. These platforms are:

  - **[Earth System European subdomain](https://earth-system.usegalaxy.eu/)**. The original subdomain, fully operational.
  - **[Earth System French lab](https://earth-system.usegalaxy.fr/)**. On this french instance the interactive tools are not working.



### 📚 Learning Pathway: Building the European Subdomain

Discover how we created the **European subdomain** on Galaxy in our dedicated **[Learning Pathway](learning-pathways/dev_tools_training.md)**. This pathway covers:

- Set up your subdomain for your community
- Build a batch tool
- Build an interactive tool

---

# 🌐 Climate and Ecology: Strongly Linked Themes

Climate and ecology are **inextricably linked**:

- **Climate** drives ecological processes (e.g., temperature affects species distribution).
- **Ecology** influences climate (e.g., forests absorb CO₂, oceans regulate heat).  
On Galaxy, you can explore these connections through:
- **Climate Data**: ERA5 reanalysis, Sentinel-5P atmospheric data.
- **Ecological Data**: OBIS biodiversity records, Sentinel-2 land cover.
- **Integrated Workflows**: Combine climate and ecological datasets to study impacts like habitat loss or carbon cycles.

💬 **Discuss these links in our [Matrix Channel](https://matrix.to/#/#galaxy-earth-system:matrix.org)**!

---

# 🗂️ Explore Earth System Components

Click on the boxes below to dive into dedicated trainings for each component of the Earth System.  
Each box includes **data sources, tools, and tutorials** to help you get started.

---

## 🌊 [Ocean: Monitoring the Blue Planet](topics/climate/tutorials/argo_pangeo/tutorial.md)

**Key Data**:

- **Argo Program**: Temperature, salinity, and biogeochemical parameters (chlorophyll, oxygen, nitrate) from floating sensors.
- **Copernicus Sentinel-3**: Sea surface temperature, topography, and color for ocean forecasting.  
**Tools**:
- [Argo Data Access](toolshed.g2.bx.psu.edu/repos/ecology/argo_getdata/argo_getdata/0.1.15+galaxy0)
- [Copernicus Data Space Ecosystem (JupyterLab)](https://usegalaxy.eu/root?tool_id=interactive_tool_copernicus_notebook)  
**Workflow**: [Full Analyse Argo Data](https://earth-system.usegalaxy.eu/u/marie.josse/w/full-analyse-argo-data)

---

## ☁️ [Atmosphere: The Invisible Shield](topics/climate/tutorials/sentinel5_data/tutorial.md)

**Key Data**:

- **Sentinel-5P**: Atmospheric composition (SO₂, NO₂, O₃, aerosol index) for air quality and climate studies.
- **ERA5 (Climate Data Store)**: Hourly atmospheric, land, and ocean wave data from 1959–present.  
**Tools**:
- [Copernicus Data Space Ecosystem](https://usegalaxy.eu/root?tool_id=interactive_tool_copernicus_notebook)
- [Climate Data Store](toolshed.g2.bx.psu.edu/repos/climate/c3s/c3s/0.2.0)  
**Use Case**: [Volcanic Activity Monitoring (La Soufrière, 2021)](topics/climate/tutorials/sentinel5_data/tutorial.md)

---

## 🏔️ [Solid Earth: The Planet’s Foundation]

**Key Data**:

- **Seismic and Volcanic Data**: Monitor earthquakes, volcanic eruptions, and tectonic activity.
- **Geodetic Data**: Measure Earth’s shape, gravity, and crustal movements.  
**Tools**:
- [QGIS Galaxy Interactive Tool](https://usegalaxy.eu/root?tool_id=interactive_tool_qgis) (for geological maps and WFS services).  
**Tutorial**: [Coming Soon!]

---

## 🌿 [Continental Surfaces: Land in Focus](topics/ecology/tutorials/ndvi_openeo/tutorial.md)

**Key Data**:

- **Sentinel-2**: Multispectral imagery for land cover, vegetation (NDVI), and soil degradation.
- **Scene Classification**: Cloud/snow masks, land cover indices (NDVI, NDSI).  
**Tools**:
- [Copernicus Data Space Ecosystem](https://usegalaxy.eu/root?tool_id=interactive_tool_copernicus_notebook)
- [QGIS Web Feature Services](topics/ecology/tutorials/QGIS_Web_Feature_Services/tutorial.md)  
**Workflow**: [From NDVI to Time Series Visualization](topics/ecology/tutorials/ndvi_openeo/tutorial.md)

---

## 🐾 [Biodiversity: Life on Earth](topics/ecology/tutorials/obisindicators/tutorial.md)

**Key Data**:

- **OBIS (Ocean Biodiversity Information System)**: Global marine species occurrence records.
- **Marine Metagenomics**: Microbial biodiversity and environmental correlations.  
**Tools**:
- [OBIS Occurrences](https://usegalaxy.eu/root?tool_id=toolshed.g2.bx.psu.edu/repos/ecology/obis_data/obis_data/0.0.2)  
**Workflow**: [Marine Omics Visualization](https://usegalaxy.eu/u/marie.josse/w/marine-omics-visualisation)

---

# 🎯 Conclusion and Next Steps

This tutorial introduced the **Earth System** and its five key components, with a focus on **European/French subdomains** and the **links between climate and ecology**. Each component has dedicated trainings, tools, and workflows on Galaxy to help you explore further.

### 🔜 What’s Next?

- **More Tutorials**: Stay tuned for new trainings on Earth System data and analysis!
- **Community**: Join our **[Matrix Channel](https://matrix.to/#/#galaxy-earth-system:matrix.org)** to share ideas and get support.
- **Feedback**: Help us improve by contributing to the **[Galaxy Training Network](https://training.galaxyproject.org/)**.

🌟 **Keep an eye on Galaxy for updates—happy exploring!** 🌟  
</canvaentity