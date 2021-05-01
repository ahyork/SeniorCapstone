---
layout: post
title: Tracking Adsorbate Positions to locate Adsorption Sites
author: Arthur York
---

# Importance of Tracking Positions

PorousMaterials.jl is intended for use with metal-organic frameworks, but it can be used to model a wide range of nano-porous materials. There are some significant benefits of modelling non-crystalline nano-porous materials, one of which being that a simulation can track the positions of adsorbates while x-ray diffraction cannot due to the unordered nature of the structure. When an experimentalist reached out to our group with this issue, I was put in charge of adapting our existing code to store and output the positions of the molecules during the simulation so that we can infer adsorption sites. 

# Taking Snapshots of Individual Molecules

The first method used for tracking positions was to capture the position of every molecule at a given cycle and output their coordinates to an xyz file. An interesting benefit of this method is that by formatting the xyz file correctly, one can obtain a "movie" where the atoms can be seen moving around. This is interesting to visualize where atoms appear in the system, but because the simulation is an exploration of state space and independent of time the movement of atoms is not indicative of how they move in the real system.

There are two keyword arguments used in the [muVT\_sim](https://simonensemble.github.io/PorousMaterials.jl/stable/manual/mof_simulations/#PorousMaterials.gcmc_simulation) function call to control the snapshots. The first is write\_adsorbate\_snapshots which serves as a toggle for whether or not the simulation will open a file and write information to it at given intervals. The other is snapshot\_frequency. This determines how many cycles between each snapshot being taken. A value of 1 means a snapshot is taken every cycle, 2 means every other cycle and so on. The default is that snapshots will not be taken (it is time consuming and if they aren't needed it speeds up the simulation), and that a snapshot is taken every cycle.

During the simulation, a filestream object is opened before any cycles occur with a filename generated by the [muVT\_output\_filename](https://simonensemble.github.io/PorousMaterials.jl/stable/manual/mof_simulations/#PorousMaterials.gcmc_result_savename) function with an xyz extension. If the user does not want to create snapshots, the filestream object is created for scoping purposed but never opened. Nothing is output to the file during the burn cycles, but every snapshot\_frequency sample cycles the current array of molecules will be output to the file using the write\_xyz file shown below. The xyz file needs to have the molecule positon in cartesian coordinates, but because the molecules are stored in fractional coordinates for the simulation the simulation box must be passed in to convert the fractional to cartesian coordinates.  

```julia
function write_xyz(box::Box, molecules::Array{Molecule{Frac}, 1}, xyz_file::IOStream)
    num_atoms = sum([mol.atoms.n for mol in molecules])
    @printf(xyz_file, "%s\n", num_atoms)
    for molecule in molecules
        for i = 1:molecule.atoms.n
            x = Cart(molecule.atoms[i].coords, box)
            @printf(xyz_file, "\n%s %f %f %f", molecule.atoms.species[i], x.x...)
        end
    end
end
```

At the end of the simulation, the file is closed. and can be read into a vizualization software such as ViSiT to view their positions. The image below shows Xenon atoms (blue) inside the CaSDB MOF. The dark blue box is the simulation box used for the simulation. The unit cell for CaSDB was replicated by (3, 6, 2) to create the full simulation box.

![single snapshot](../assets/img/xe_in_casdb_snapshot.png)