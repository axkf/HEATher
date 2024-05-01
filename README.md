--------------------------------------------------------------------------------
                                  HEATher
--------------------------------------------------------------------------------

author:       https://github.com/axkf
dependencies: bash 4.4.20(1) , GNU awk 4.2.1
version:      2024-04-27


  HEATher is a tool that calculates the chemical potential and all of its compo-
  nents from vasp phonon calculations (IBRION 5 and 6) of molecules. It can be 
  used to get values for specific temperatures and pressure values or output 
  formula strings, that can be utilized as input for other programs.


------------------------------ GENERAL USAGE -----------------------------------

    path/HEATher [-options] FILES

------------------------------ FILES / INPUT -----------------------------------


  HEATher accepts molecular phonon calculations in OUTCAR format, irrespective of 
  file name or extension. If multiple files are given, they will be processed in 
  order and deleted from the argument list before processing options. Hence,  
  options and files can be given in any order. At the moment piping is not
  supported.


-------------------------- OPTIONS AND PRINTING --------------------------------


PARAMETERS:

  To get values, each file needs its own set of temperature- (in K, at least 2K) 
  and pressure values (in Pa), which are to be supplied as a space-separated list 
  via "-T" and "-P" using integer, floating point or exponential notation. 
  Unfortunately, the symmetry parameter sigma, which is needed for the moment of 
  inertia calculation, needs to be supplied manually by using "-s". If only 
  one value is given, this value is used for all files, if none is given or the 
  number of values given differs from that of the files the program defaults to 
  -s 1 and/or -T 298.15 and/or -P 1.01325E+05. For example
  
    path/HEATher -s 2 5 -T 1000 file1 file2 -p

  is equivalent to
  
    path/HEATher -s 2 5 -T 1000 1000 -P 1.01325E+05 1.01325E+05 file1 file2 -p .
  
  
T- and P-SERIES:
  
  By using another delimiter it is possible to expand T- and or P-arguments 
  to generate a T-/P-series. Characters bash might misinterpret should be 
  avoided, escaped or single-quoted ("," , ":" , "_" and "#" are fine). Naturally, 
  characters used within numbers should not be used as well. All rules discussed 
  above still apply, since each such list is treated as a single argument. For 
  example
  
    path/HEATher -s 2 5 -T 1300,1400 30,40,50 -P 1E4 10,100,1E3,1E4 file1 file2 -p
  
  prints one T-series for file1 and four T-series for file2 (one for each value 
  of P).
  
  
CORRECTIONS:
  
  Three correction options (-cH, -cS, -cG) were implemented to manipulate the output 
  by adding additional terms, factors or exponents. This can be used to account for 
  electronic degeneracy, for example
  
    path/HEATher -s 2 -T 1000 -cG "-R*T*log(3)/1000" file1
  
  or
  
    path/HEATher -s 2 -T 1000 -cS "+R*log(3)" file1
  
  for a triplett state, internal rotations and/or empirical corrections. All 
  operators implemented in GNU awk as well as some fundamental constants (see 
  chapter EXPORT) can be used. Make sure there is an operator connecting the 
  corrections to the original expression and to include unit conversions. Since the 
  logic of assigning arguments to files is similar to that of T, P and sigma, there 
  should be no space inside the terms. Finally, the terms should also be quoted to 
  avoid interference of bash.
  

PRINTING:

  Calculated values can be printed by calling the -p option. By default this prints  
  the sum formula, energy, zero-point energy, enthalpy change, evaluated enthalpy 
  correction and total enthalpy, entropy correction and total entropy, as well as
  the correction of the Gibbs free energy and its total value. All values are given
  in kJ/mol except for the entropy (J/mol/K). Please use the correction options, to 
  convert to another unit.
  
  Contents and format of the output table are freely customizable by specifying the 
  desired arrys (see EXPORT) and their format as a printf-string. For example
  
    -p '"%-10s %.2f\n" ${A_HTR_CMPND[$number]} ${A_HTR_VAL_energy[$number]}'
  
  prints the sum formulas, as well as the energies as floating point numbers with a 
  precision of two digits. Printing is done for the whole array ($number being the 
  counting variable). In case of array-appending (see below) calling -p is thus only 
  necessary in the last call of the script. Please quote the format string as shown
  in the example. For safety reasons any semicolon in the string will be deleted.
  

APPEND ARRAYS:
  
  To group calculations with similar parameters/corrections the option "-a" can be 
  used. This disables the array reset at the start of the program to append the 
  following entries instead of overwriting. Available arrays are listed below.
  
  
--------------------------------- EXPORT ---------------------------------------


  Using the shell utility "source" via
  
    source path/HEATher [-options] FILES
  
  or
  
    . path/HEATher [-options] FILES

  allows for direct access of all calculated values via indexed arrays:
	
    A_HTR_VAL_energy
    A_HTR_VAL_zpe
    A_HTR_VAL_H_trans
    A_HTR_VAL_H_rot
    A_HTR_VAL_H_vib
    A_HTR_VAL_H_diff
    A_HTR_VAL_H_corr
    A_HTR_VAL_H_total
    A_HTR_VAL_S_trans
    A_HTR_VAL_S_rot
    A_HTR_VAL_S_vib
    A_HTR_VAL_S_corr
    A_HTR_VAL_S_total
    A_HTR_VAL_G_corr
    A_HTR_VAL_G_total
  
  Moreover, a formula string is generated for each of the properties and their 
  components, which can be used as input for further calculations and plotting:

    A_HTR_FML_H_trans
    A_HTR_FML_H_rot
    A_HTR_FML_H_vib
    A_HTR_FML_H_diff
    A_HTR_FML_H_corr
    A_HTR_FML_H_total
    A_HTR_FML_S_trans
    A_HTR_FML_S_rot
    A_HTR_FML_S_vib
    A_HTR_FML_S_corr
    A_HTR_FML_S_total
    A_HTR_FML_G_corr
    A_HTR_FML_G_total

  Since there is unfortunately no easily portable sum function, the latter can 
  however get quite lengthy for bigger molecules. To enhance readability, none
  of the formulas contains values for the fundamental constants used, the latter
  are however exported as well:
  
    NA=6.02214076E+23 1/mol
    h=6.62607015E-34 Js
    kB=1.380649E-23 J/K
    R=-8.31446261815324 J/mol/K
    F=96485 cG/mol
    u=1.6605390666050E-27 kg
    pi=3.141592653589793

  Some basic structure information is availlable via 
  
    A_HTR_ELEM
    A_HTR_ELEZ
    A_HTR_CMPND

  which contain strings of all involved elements, their respective number of 
  atoms and a sum formula (all sorted using IUPAC pseudo-electronegativity). To 
  link these to the respective files, two additional arrays are created
  
    A_HTR_FILES
    A_HTR_INDEX

  the first of which contains the file name for each number, while the second 
  can be used to find the respective number for each of the files. Using the 
  series function duplicates the respective files, to make each calculated value 
  accessible independantly, thus changing the numbering in A_HTR_FILES. In this 
  case, A_HTR_INDEX stores all respective numbers as a string. Usually, all arrays
  are reset at the start of the program. This can however be disabled with the 
  tag "-a".
