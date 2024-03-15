# RFSAR
This a proof of concept for a GUI Tool for Floodmapping using Random Forests Machine Learning and Synthetic Aperture Radar.
This Tool was developed in Java for the GEE Code Editor API. 
# Methodology 
This section details the algorithm, as illustrated in Figure 3, which is applied for SAR-based flood mapping using the RF classifier in the case study. The process encompasses compiling input features, sample design, implementing the Random Forest classifier, followed by post-processing, and concluding with performance assessment.

![image](https://github.com/paulhosch/RFSAR/assets/39274609/ffd6b5d2-b8f8-4c0b-9557-dc56dd78fa31)

## Environment 
Google Earth Engine (GEE) provides a powerful cloud-based platform for geospatial analysis on a planetary scale. The backbone of GEE is its multi-petabyte, analysis-ready data catalogue. The data catalogue includes a wide range of geospatial datasets, encompassing observations from various satellite systems, environmental variables, and datasets related to weather, climate, land cover and topography. Coupled with the in-house implementation of Random Forest, and its substantial computational resources, GEE stands as the ideal platform for the use case addressed in this study.



##	Input Feature Compilation
In this study, the variables or used to split each node during the RF training are referred to simply as input features. Esfandiari et al. (2020) investigated the impact of various input features and their combinations on the accuracy of flood prediction models. Their research primarily focused on geological features and excluded SAR imagery features, such as VV- or VH-polarization bands. They found the highest predictive accuracy was achieved using features such as altitude or Height Above Nearest Drainage (HAND), slope, aspect, proximity to rivers, and land use or land cover. Additionally, they concluded that the inclusion of additional input features does not necessarily enhance accuracy, while including correlated input features may actually decrease the accuracy of the flood mapping results. This finding aligns with the independent findings by Millard & Richardson (2015). Consequently, this study has selected VV, VH, slope, aspect, HAND, and land cover as input features, which will be discussed in this section in detail.

![Bildschirmfoto 2024-03-15 um 17 39 46](https://github.com/paulhosch/RFSAR/assets/39274609/3f1da08e-6e70-4187-be24-8691505f7d7a)

###	SAR Imagery 

  
Figure 4: Spatial Visualization of the SAR Backscatter in dB 
for the Polarizations VH (a) and VV (b); Sentinel-1 Coverage of Subset 1 on March 19, 2021.

The methodology was devised for use with C-band Sentinel-1 (S1) satellite imagery, at a spatial resolution of 10 meters. The Sentinel-1 mission, managed by the European Space Agency (ESA), comprises a constellation of two identical satellites, Sentinel-1A and Sentinel-1B which orbit 180 degrees apart. This configuration offers a revisit time of six days at the equator. Sentinel-1's SAR system operates with dual polarization capabilities, simultaneously transmitting and receiving horizontal (H) and vertical (V) polarizations (European Space Agency, n.d.). Polarization refers to the orientation of the electric field vector in the plane perpendicular to the direction of wave propagation. Through the relationship of H and V components, any desired polarization can be calculated for both transmission and reception (Goverment of Canada, 2015). Ground objects present unique polarization signatures, and certain scattering mechanisms are more discernible with specific polarizations. For instance, double-bounce scattering in inundated urban areas is more evident in the VV (vertical transmit, vertical receive) polarization, whereas some vegetation-induced diffuse volume scattering might be more prominent in the VH (vertical transmit, horizontal receive) polarization. As a result, Sentinel-1's simultaneous acquisition of VH and VV polarization provides dual-image bands that provide a comprehensive insight into the surface backscattering characteristics. Research by Pelich et al. (2022) on the Sentinel-1 VV and VH polarization bands found that in urban areas the use of both polarizations increased the flood detection capacity by more than 5%.  Therefore, both available polarization bands are used for analysis in this study.
The European Commission (EC) and ESA have agreed to adopt a policy of free and open access (Fletcher, 2012). Consequently, the GEE Data Catalogue provides a daily updated repository of all Ground Range Detected (GRD) scenes. Internally, the raw data undergoes pre-processing before publication, which includes border noise removal, thermal noise removal, radiometric calibration, and terrain correction, resulting in the backscatter coefficient (σ°) (Google Earth Engine, n.d.). The backscatter coefficient quantifies the radar cross-section per unit ground area, which, due to its wide dynamic range, is represented in the logarithmic scale as decibels, as visualized on a greyscale in Figure 4.   
In this study image quality and interpretability is enhanced by further reducing speckle noise. This is achieved by applying the Refined Lee Filter, as proposed by Yommy et al. (2015). This filter effectively diminishes noise while preserving essential image features such as edges, texture, and patterns. For further details on the algorithm applied, the reader is referred to (Yommy et al., 2015). 

![image](https://github.com/paulhosch/RFSAR/assets/39274609/e24e51c2-47e2-407d-b05b-05096cc94d2e)

### Terrain Derivatives
 
Figure 5: Spatial Visualization of the MERIT DEM Derivatives
Aspect (a) and Slope (b) Used as Input Features: Coverage of Subset 1.

The research employed the Multi-Error-Removed Improved-Terrain (MERIT) Digital Elevation Model (DEM), as described by Yamazaki et al. (2017), to compute topographical derivatives such as slope and aspect for use as input features. MERIT DEM enhances the accuracy of existing elevation models by correcting systematic errors – including absolute bias, stripe noise, speckle noise, and vegetation bias – leveraging a combination of satellite data and advanced filtering methodologies (Yamazaki et al., 2017).
The MERIT DEM offers elevation details at a relatively high resolution of 3 arc-seconds, which corresponds to about 90 meters at the equator. For an in-depth explanation of the techniques used to refine the DEM, one should refer to the original publication by Yamazaki et al. (2017).
The slope of the terrain is calculated from the rate of elevation change at each pixel from its immediate four neighbouring pixels, with the result expressed in degrees. The aspect, indicating the direction of the steepest descent from each pixel, is similarly calculated based on the elevation change relative to its four closest neighbours, and this is also expressed in degrees. The resulting slope and aspect layers are employed as input features to the RF classification and are visualized in Figure 5.
![image](https://github.com/paulhosch/RFSAR/assets/39274609/79742ed3-1db5-4679-94ba-319c60db20af)

###	Height Above Nearest Drainage and Landcover

 
Figure 6: Spatial Visualization of Merit Hydro HAND (a) and DEA Land Cover (b)
Coverage of Subset 1

The HAND layer, Figure 6 (a), utilized as input features is sourced from the MERIT Hydro global hydrography dataset, which offers a resolution scale of 3 arcsecond, approximately equivalent to 92.77 meters, and vertical increments of 0.1 meters. For a comprehensive description of the methodologies used to derive the HAND metric, readers are referred to Yamazaki et al. (2019).
The Digital Earth Australia Land Cover Dataset, Figure 6 (b), employed as an input feature layer, is published by Geoscience Australia as part of the Digital Earth Australia Program. It provides an annual land cover classification at a resolution of 25 meters utilizing the taxonomy of the FAO’s Land Cover Classification System (LCCS), Version 2. For its use as an input feature, the dataset is simplified to the base Level 3 classification. For a comprehensive description of the methodologies used to derive the Land Cover classification, readers are directed to the Dataset's Description (Lucas et al., 2019).

![image](https://github.com/paulhosch/RFSAR/assets/39274609/681cb925-530b-46f5-809e-c2a2a5fd2d08)

