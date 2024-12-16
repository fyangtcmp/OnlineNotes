A modified edition of **WannierTools**. It supports the calculations of some nonlinear transport properties.
## Citation

Please cite the original edition of **WannierTools**

```
@article{WU2018,
title = "WannierTools : An open-source software package for novel topological materials",
journal = "Computer Physics Communications",
volume = "224",
pages = "405 - 416",
year = "2018",
doi = "https://doi.org/10.1016/j.cpc.2017.09.033",
url = "http://www.sciencedirect.com/science/article/pii/S0010465517303442",
issn = "0010-4655",
preprint = "arXiv:1703.07789",
author = "QuanSheng Wu and ShengNan Zhang and Hai-Feng Song and Matthias Troyer and Alexey A. Soluyanov",
keywords = "Novel topological materials, Topological number, Surface state, Tight-binding model"
}

```

## Features

1. Calculate some types of nonlinear transport conductivities, see Usage for detailed information
2. Calculate both spin and orbital magnetic moments and their contributions to the conductivities which are involved with B field
3. Add effective spin and orbital Zeeman field for spinful Wannier functions, the formulas please see the [Referenece](https://arxiv.org/abs/1512.05084).
4. Because the calculations of these conductivities are heavy, we only support the parallel version. Executing the code with only one CPU core will not work due to the asynchronous parallel algorithm

## Usage

### Spin and orbital magnetism

Include the corresponding magnetic moments for NPHC calculations.

When `Add_Zeeman_Field = .true.`, they will also decide whether spin or orbital effect are included in the Zeeman term. The spin Zeeman term is added in the `readNormalHmnR` subroutine. The orbital Zeeman term is added in the `ham_bulk_latticegauge` subroutine.

The g-factors for spin and orbital are set in the `magnetic_moments.f90`, we do not use the `Effective_gfactor` tag.

```
&SYSTEM
Add_Zeeman_Field = .false.
include_m_spin = .true.
include_m_orb = .true.
Bx = 0 ! Tesla
By = 0
Bz = 0
/
```

### Drude weight ($\sigma_{xx}/\tau$)

```
&CONTROL
drude_weight_calc = .TRUE.
/

&SYSTEM
E_FERMI = 7.1370
/

&PARAMETERS
OmegaNum = 601
OmegaMin = -0.5
OmegaMax = 0.5
Nk1 = 201
Nk2 = 201
Nk3 = 201
Eta_Arc = 0.001
/

KCUBE_BULK
0.00 0.00 0.00 ! Original point for 3D k plane
1.00 0.00 0.00 ! The first vector to define 3d k space plane
0.00 1.00 0.00 ! The second vector to define 3d k space plane
0.00 0.00 1.00 ! The third vector to define 3d k cube
```


### Intrinsic second order Hall effect ($\chi_{abc}$)

Ref: 10.1103/PhysRevLett.127.277201, 10.1103/PhysRevLett.127.277202

```
&CONTROL
sigma_SOAHC_int_calc = .TRUE. ! static mpi, fixed k-mesh
/

&PARAMETERS
OmegaNum = 601
OmegaMin = -0.5
OmegaMax = 0.5
Nk1 = 2001 ! very high value for fixed k-mesh method
Nk2 = 2001
Nk3 = 2001
Eta_Arc = 0.001
/

KCUBE_BULK
0.00 0.00 0.00 ! Original point for 3D k plane
1.00 0.00 0.00 ! The first vector to define 3d k space plane
0.00 1.00 0.00 ! The second vector to define 3d k space plane
0.00 0.00 1.00 ! The third vector to define 3d k cube
```

### Intrinsic nonlinear planar Hall effect ($\chi_{abcd}$)

Ref: 10.1103/PhysRevLett.130.126303

### Drude-like nonlinear planar Hall effect ($\chi_{abcd}/\tau^{2}$)

Ref: 10.1103/PhysRevB.108.075155

### Third order Hall effect ($\chi_{abcd}/\tau^{1}$ and $\chi_{abcd}/\tau^{3}$)

Ref: 10.1103/PhysRevB.105.045118

```
&CONTROL
sigma_NPHC_int_calc = .TRUE. ! dynamical mpi, auto adapted k-mesh
! sigma_NPHC_tau2_calc = .TRUE. ! dynamical mpi, auto adapted k-mesh
! sigma_TRAHC_calc = .TRUE. ! dynamical mpi, auto adapted k-mesh
/
 

&PARAMETERS
OmegaNum = 601
OmegaMin = -0.5
OmegaMax = 0.5
Nk1 = 201 ! it will be auto adapted with 5x5x5 local k-mesh
Nk2 = 201
Nk3 = 201
Eta_Arc = 0.001
band_degeneracy_threshold = 3.6749d-5 ! 1meV
/


KCUBE_BULK
0.00 0.00 0.00 ! Original point for 3D k plane
1.00 0.00 0.00 ! The first vector to define 3d k space plane
0.00 1.00 0.00 ! The second vector to define 3d k space plane
0.00 0.00 1.00 ! The third vector to define 3d k cube
```

### Distributions for a specific kplane

```
&CONTROL
band_geo_props_kplane_calc = .TRUE.
option_prop = ! 'SOAHC_int", "NPHC_int" or "SHC", no default value
option_sumover = 'surface' ! 'sea'=Fermi sea or 'surface'=Fermi surface
/

&PARAMETERS
Eta_arc = 0.02 ! infinite small value, like brodening
E_arc = 0.0 ! energy for calculate Fermi Arc
Nk1 = 301 ! number k points odd number would be better
Nk2 = 301 ! number k points odd number would be better
/

KPLANE_BULK ! unit is the reciprocal lattice vectors
0.00 0.00 0.00 ! center point for 3D k plane
1.00 0.00 0.00 ! The first vector to define 3d k space plane
0.00 1.00 0.00 ! The second vector to define 3d k space plane
```