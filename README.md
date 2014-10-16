PDBregex
========

Regular expressions to parse ATOM records from [RCSB Protein Data Bank](http://www.rcsb.org/pdb/home/home.do) (PDB) files.


This repository shall provide reasonably robust regular expressions to parse the atomic data stored in PDB files.
A testset of problematic PDB files shall be provided as well.

Structural biologists usually want the information stored in the PDB lines that read like this
```
ATOM   1317  C   LYS A 188A     57.157  42.972  41.187  1.00 14.33           C  
ATOM   1318  O   LYS A 188A     56.236  43.769  41.358  1.00 14.64           O  
ATOM   1319  CB  LYS A 188A     59.043  44.258  42.166  1.00 16.98           C  
```

The documentation of the PDB format can be found in the
[official documentation](http://www.wwpdb.org/documentation/format33/v3.3.html). And the section about ATOM lines can be found [here](http://www.wwpdb.org/documentation/format33/sect9.html#ATOM).


Issues with the PDB format
========

To quote from the [RCSB homepage](http://www.rcsb.org/pdb/static.do?p=file_formats/index.jsp):
> The Protein Data Bank (PDB) format provides a standard representation for macromolecular structure data derived from X-ray diffraction and NMR studies. This representation was created in the 1970's and a large amount of software using it has been written.  

Back in the 1970's column oriented text files in uppercase ASCII where state of the art. Processing these files can be a nightmare as there are many subtleties that you might not realize at first because only 1 out of 100 PDB files might be affected. 

To me the most important problems are:
* Residue IDs are _not_ numeric. 

  There is an annoying feature of PDB files: The insertion code (iCode) stored in column 27 is a single character that is actually part of the residue ID. From the docs: 

> Alphabet letters are commonly used for insertion code. The insertion code is used when two residues have the same numbering. The combination of residu enumbering and insertion code defines the unique residue.  

Insertion codes are routinely used when biologists use black magic to insert one ore more extra residues into the sequence of a protein (insertion mutations). This means, you can have a residue ALA 450 followed by GLY 450A which is followed by LEU 451. If the insertion code is ignored, this would lead to two residues with the same ID. Some tools might just crash, others just skip one of the two residues.

* Non-proteinogenic amino acids

 Non-standard amino acids exists. In nature and in experiment. In nature they are rare, in experiments they are routinely inserted by crystallographers. For example they often feel the desire to exchange methionine (MET) with selenomethionie (MSE) because SE produces a strong signal in MAD phasing experiments (http://en.wikipedia.org/wiki/Selenomethionine).
In PDB files all non-standard amino acids must be encoded in a HETATM line. This is just a normal ATOM line with "ATOM   " being replaced with HETATM. Many programs fail to process these lines at all, leading to false gaps in the sequence.

* HETATMS in general

You cannot trust HETATMS. Atoms that are stored in a HETATM line can be anything: Water, some chemical compound, a non-standard amino acid - and sometimes even a normal residue (this is probably illegal, but some PDBs do it).

* Large files
 
Of course, when you get to many atoms, the atom serial will no longer fit into columns 7-11. You will run out chain letters, too.

* Whitespace

Dont try to split an ATOM line at the spaces. The x, y and z coordinates are formatted as (8.3)-floats. The numbers can touch. These are valid ATOM lines:

```
ATOM    663  N   GLY A  96    8886.5548834.0248887.681  1.00 22.44           N  
ATOM    664  CA  GLY A  96    8885.1778834.3888887.962  1.00 22.21           C  
ATOM    665  C   GLY A  96    8884.1718833.2568887.921  1.00 21.12           C  
ATOM    666  O   GLY A  96    8883.0248833.4308888.339  1.00 20.66           O  
```

So why not use a column-oriented approach to parse?
========
Maybe you should! For many applications this might be faster and allow for easier debugging in case of erroneous PDB files. However, regular expressions tend to be portable across different programming languages. 

TODO
========
* More examples should follow.
* Prepare testset of problematic PDB files
* I am sure, I once stubmled over negative residue IDs...
* Prepare true column-based parsing and compare speed
