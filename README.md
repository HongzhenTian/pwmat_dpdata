**dpdata** is a python package for manipulating DeePMD-kit, VASP, LAMMPS data formats.
dpdata only works with python 3.x.


# Installation
One can download the source code of dpdata by 
```bash
git clone https://github.com/deepmodeling/dpdata.git dpdata
```
then use `setup.py` to install the module
```bash
cd dpdata
python setup.py install
```

`dpdata` can also by install via pip
```bash
pip3 install dpdata
```


# Quick start

This section gives some examples on how dpdata works. Firstly one needs to import the module in a python 3.x compatible code.
```python
import dpdata
```
The typicall workflow of `dpdata` is 

1. Load data from vasp or lammps or deepmd-kit data files.
2. Manipulate data 
3. Dump data to in a desired format


## Load data
```python
d_poscar = dpdata.System('POSCAR', fmt = 'vasp/poscar')
```
or let dpdata infer the format (`vasp/poscar`) of the file from the file name extension
```python
d_poscar = dpdata.System('my.POSCAR')
```
The number of atoms, atom types, coordinates are loaded from the `POSCAR` and stored to a data `System` called `d_poscar`.
A data `System` (a concept used by [deepmd-kit](https://github.com/deepmodeling/deepmd-kit)) contains frames that has the same number of atoms of the same type. The order of the atoms should be consistent among the frames in one `System`. 
It is noted that `POSCAR` only contains one frame.
If the multiple frames stored in, for example, a `OUTCAR` is wanted, 
```python
d_outcar = dpdata.LabeledSystem('OUTCAR')
```
The labels provided in the `OUTCAR`, i.e. energies, forces and virials (if any), are loaded by `LabeledSystem`. It is noted that the forces of atoms are always assumed to exist. `LabeledSystem` is a derived class of `System`.

The `System` or `LabeledSystem` can be constructed from the following file formats with the `format key` in the table passed to argument `fmt`:

The Class `dpdata.MultiSystems`  can read data  from a dir which may contains many files of different systems, or from single xyz file which contains different systems.

Use `dpdata.MultiSystems.from_dir` to read from a  directory, `dpdata.MultiSystems` will walk in the directory 
Recursively  and  find all file with specific file_name. Supports all the file formats that `dpdata.LabeledSystem` supports.

Use  `dpdata.MultiSystems.from_file` to read from single file. Now only support quip/gap/xyz  format file.

For example, for `quip/gap xyz` files, single .xyz file may contain many different configurations with different atom numbers and atom type.

The following commands relating to `Class dpdata.MultiSystems` may be useful.
```python
# load data

xyz_multi_systems = dpdata.MultiSystems.from_file(file_name='tests/xyz/xyz_unittest.xyz',fmt='quip/gap/xyz')
vasp_multi_systems = dpdata.MultiSystems.from_dir(dir_name='./mgal_outcar', file_name='OUTCAR', fmt='vasp/outcar')

# use wildcard
vasp_multi_systems = dpdata.MultiSystems.from_dir(dir_name='./mgal_outcar', file_name='*OUTCAR', fmt='vasp/outcar')

# print the multi_system infomation
print(xyz_multi_systems)
print(xyz_multi_systems.systems) # return a dictionaries

# print the system infomation
print(xyz_multi_systems.systems['B1C9'].data)

# dump a system's data to ./my_work_dir/B1C9_raw folder
xyz_multi_systems.systems['B1C9'].to_deepmd_raw('./my_work_dir/B1C9_raw')

# dump all systems
xyz_multi_systems.to_deepmd_raw('./my_deepmd_data/')


```

| Software| format | multi frames | labeled | class	    | format key    |
| ------- | :---   | :---:        | :---:   | :---          | :---          |
| vasp	  | poscar | False        | False   | System	    | 'vasp/poscar' | 
| vasp    | outcar | True         | True    | LabeledSystem | 'vasp/outcar' |	
| vasp    | xml    | True         | True    | LabeledSystem | 'vasp/xml'    |	
| lammps  | lmp    | False        | False   | System        | 'lammps/lmp'  |
| lammps  | dump   | True         | False   | System        | 'lammps/dump' |
| deepmd  | raw    | True         | False   | System	    | 'deepmd/raw'  |
| deepmd  | npy    | True         | False   | System        | 'deepmd/npy'  |
| deepmd  | raw    | True         | True    | LabeledSystem | 'deepmd/raw'  |
| deepmd  | npy    | True         | True    | LabeledSystem | 'deepmd/npy'  |
| gaussian| log    | False        | True    | LabeledSystem | 'gaussian/log'|
| gaussian| log    | True         | True    | LabeledSystem | 'gaussian/md' |
| siesta| output    | False        | True    | LabeledSystem | 'siesta/output'|
| siesta| aimd_output    | True         | True    | LabeledSystem | 'siesta/aimd_output' |
| cp2k    | output | False        | True    | LabeledSystem | 'cp2k/output' |
| QE      | log    | False        | True    | LabeledSystem | 'qe/pw/scf'   |
| QE      | log    | True         | False   | System        | 'qe/cp/traj'  |
| QE      | log    | True         | True    | LabeledSystem | 'qe/cp/traj'  |
|quip/gap|xyz|True|True|MultiSystems|'quip/gap/xyz'|

## Access data
These properties stored in `System` and `LabeledSystem` can be accessed by operator `[]` with the key of the property supplied, for example
```python
coords = d_outcar['coords']
```
Available properties are (nframe: number of frames in the system, natoms: total number of atoms in the system)

| key		|  type		| dimension		| are labels	| description 
| ---		| ---		| ---			| ---		| ---
| 'atom_names'	| list of str	| ntypes		| False		| The name of each atom type
| 'atom_numbs'	| list of int	| ntypes		| False		| The number of atoms of each atom type
| 'atom_types'	| np.ndarray	| natoms		| False		| Array assigning type to each atom
| 'cells'	| np.ndarray	| nframes x 3 x 3	| False		| The cell tensor of each frame
| 'coords'	| np.ndarray	| nframes x natoms x 3	| False		| The atom coordinates
| 'energies'	| np.ndarray	| nframes		| True		| The frame energies
| 'forces'	| np.ndarray	| nframes x natoms x 3	| True		| The atom forces
| 'virials'	| np.ndarray	| nframes x 3 x 3	| True		| The virial tensor of each frame



## Dump data
The data stored in `System` or `LabeledSystem` can be dumped in 'lammps/lmp' or 'vasp/poscar' format, for example:
```python
d_outcar.to_lammps_lmp('conf.lmp', frame_idx=0)
```
The first frames of `d_outcar` will be dumped to 'conf.lmp'
```python
d_outcar.to_vasp_poscar('POSCAR', frame_idx=-1)
```
The last frames of `d_outcar` will be dumped to 'POSCAR'.


The data stored in `LabeledSystem` can be dumped to deepmd-kit raw format, for example
```python
d_outcar.to_deepmd_raw('dpmd_raw')
```
Or a simpler command:
```python
dpdata.LabeledSystem('OUTCAR').to_deepmd_raw('dpmd_raw')
```
Frame selection can be implemented by
```python
dpdata.LabeledSystem('OUTCAR').sub_system([0,-1]).to_deepmd_raw('dpmd_raw')
```
by which only the first and last frames are dumped to `dpmd_raw`.
