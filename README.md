# Automated-data-analysis-for-in-situ-TEM-videos

<a name="readme-top"></a>

<br />
<div align="center">
  <h3 align="center">a lattice-based automated atomic resolution TEM image process and analysis pipeline</h3>
  <p align="center">
    <br />
    <br />
    <br />
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
    </li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

Atomic resolution transmission electron microscopy (TEM) image gives tremendous structural imformation of the materials. However, only limited information was utilized in most of the TEM related researches due to the great challenge in quantatitive analysis of TEM images. And the quality of data analysis relies very much on the experience of the researchers. For in situ atomic resolution TEM, the challange is even greater since the dataset typically contains thousands of images.  
To tackle this issue, automating image process and data mining process is necessary. One of the most important tasks is the grain segmentation, which could give the morphologies of nanocrystals and the grain boundaries of polycrystals. This is also the prerequsite step for automated strain analysis, defect analysis, etc. 
Therefore, for this project, we are motivated to develop automated grain segmentation alogrithm for atomic resolution TEM images (HRTEM, STEM, etc.) of crystalline samples based on lattice pattern. 
There are mainly three modules: atom orientation mapping, grain segmentation and atom tracing. Video drift correction is also included as preprocessing in the seperate notebook. 

### Atom orientation mapping

In HRTEM images, atom columns are visible when the crystal is on zone axis. The orientation of atoms can be determined from the arrangement of the neighbor atoms. Here we borrow the idea of template matching (P. M. Larsen et al. Robust Structural Identification via Polyhedral Template Matching, Modelling Simul. Mater. Sci. Eng. 24, 055007 (2016), doi:10.1088/0965-0393/24/5/055007). Following is the flowchart:  
<img src="image/OrientationMapping.png" alt="Logo" width="500" height="500">  
Image is first processed by FFT filter. Then atoms in the image are detected by LoG blob detection. Voronoi analysis is used to represent the local environment of atoms. Template matching is therefore conducted on each voronoi cell of the atoms, yielding the orientation of the atoms. In this way, we get the atom orientation mapping.  
For the usage, please refer to Atom_orientation_mapping.ipynb.

### Grain segmentation

The feature used for grain segmentation of HRTEM images is the lattice translational symmetry, which is reflected in the FFT patterns. In the FFT space, the periodic order is reflected as peaks of amplitude, while the location information is saved in the imaginary part of the FFT signal. For one grain, it has multiple orders (periodicity of atomic planes). By utilizing this characteristic, we conduct the grain segmentation in the following order.   
<img src="image/GrainSeg.png" alt="Logo" width="750" height="500">  
FFT was firstly performed on the HRTEM image. Then diffraction spots was detected using LoG blob detection. Then each spot was selected and masked for inverse FFT, which gives the rough location for the origin of that order (grain location). By comparing the overlapping, we can find the spots (periodic orders) for the same grain. These spots are grouped as the spots for same grain. Then, we select and mask the spots for same grain, perform inverse FFT and get the real space image that filtered other forbidden orders. After that, LoG blob detection was used to detect atom locations in the iFFT images. This gives the atom locations for that grain. However, due to the signal leakage issue of cropped FFT, the observed order in iFFT image would be larger than the real periodic order of that grain. Therefore, more atoms would be detected in iFFT image than the real case. To takle this issue, we also perform LoG detection in the original image. By comparing the atom locations in iFFT image and original image, we then filtered the falsely detected atoms in iFFT image. (Due to the probable overlap of falsely detected atoms with real atomic locations, this process is complicated. It was solved by s-t graph cut.) By repeating this on the FFT spots of each grain, we finally get the atoms labelled based on the parent grains.   
For the usage, please refer to Grain_Segmentation_HRTEM.ipynb.

### Atom trajectory tracking
Motions of atoms between frames are traced in in situ videos with atomic resolution. For example, in the following images showing the shear displacement of two grains caused by shear coupled grain boundary migration, The atomic displacement is plotted:  
<img src="image/DispMap.png" alt="Logo" width="400" height="250">  
Statistical analysis can be conducted on the distribution of the atom displacement:  
<img src="image/DispDist.png" alt="Logo" width="400" height="250">  
This can be used to reflect and analyze the displacement of atoms driven by stress, diffusion, etc. 
For the usage, please refer to Atom trajectory tracing.ipynb. 
<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Strain mapping
There are generally two methods for strain mapping of HRTEM images, respectively the Peak Pair Algorithm (PPA) in real space and Geometric Phase Analyais (GPA) in 
reciprocal space. Unfortornately, both of these two methods can only take one structure as the reference for strain calculation, which makes them not applicable to the exhibit the strain in multiple lattices simutaneously and consistently. To solve this issue, we adopt the method of PPA in real space and generalize it to polycrystalline regions by defining a consistent template as the reference, and incooperating rotational component for the matching process (as we did in the orientation mapping). Clearly, with the definition of consistent template, this method can then be applied to in situ videos to show the evolution of strain.  
Before we start, we would like to deal with the drift noise of STEM images. If you work with HRTEM image, please feel free to ignore this part. We use lattice relaxation method to complete the task of denoise:  
<img src="image/LatticeRelaxation.png" alt="Logo" width="700" height="450">  
We try to rebalance the regularity of the lattice with the observation of lattice points. By balance the image and lattice force, we can achieve the effect of denoise. Since the boundary atoms are fixed, the total strain is conserved in this process. The strength of denoise can be adjusted by the ratio between image force and lattice force.  
Then we can start to conduct strain mapping on polycrystal images. To begin with, we need to select a reference lattice for strain mapping. This is done by manually select the reference area and two FFT spots, as shown in the following Figure.  
<img src="image/AutoTemplate.png" alt="Logo" width="700" height="220">  
Then, the atom locations are found in a subpixel way. After that, the relative locations of the atoms and their neighbors give the information of strain as well as orientation. The final result is shown in the following:  
<img src="image/strain_mapping.png" alt="Logo" width="700" height="630">  
By taking the averaged lattice of the first frame as reference, we are able to get the time resolved strain mapping. Then a lot of analysis can be conducted. For example, we can select an area and get the temporal evolution of averaged strain in that area, as shown in the following:  
<img src="image/Strain_evolution.png" alt="Logo" width="1300" height="550">  
<!-- ROADMAP -->
## Roadmap

- [x] Atom orientation mapping
- [x] Grain segmentation
- [x] Polycrystal strain mapping
- [ ] Crystallographic defect detection


<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Yuan Tian - tiany17@uci.com

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## Acknowledgments

This project is supported by Army Research Office (ARO) Project about grain boundary dynamics. The code was developed initially for the analysis of in situ videos of grain boundary dynamics in polycrystal materials. The data was collected at Irvine Materials Research Institute (IMRI), UCI. 

<p align="right">(<a href="#readme-top">back to top</a>)</p>



