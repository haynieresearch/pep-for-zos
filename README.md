# Propellant Evaluation Program for z/OS
Propellant Evaluation Program for z/OS is a mainframe implementation of the program
ProPEP that traces back to the mid 1980's from the Naval Weapons Center at China Lake
and Martin Marietta with very little documentation.
The last official database was updated in 1986 by John Commings.
However, I have included an updated version last updated by
Richard Nakka https://www.nakka-rocketry.net.

Propellant Evaluation Program for z/OS is a chemical equilibrium program that
balances the chemical equations based on the propellant reactants and byproducts.
The method used by PEP is known as minimization of Gibbs free energy.

I am probably one of the few that still uses, and prefers, mainframe applications
so I am not sure how useful this will be to the general public. However, I did want
to share to help document the PEP application as a whole. Furthermore, you can most
likely use this code to recompile on other operating systems. If you have any questions,
please feel free to reach out.

### Purpose of the Program
The PEP program finds the performance of solid (or liquid) rocket propellents.

### Highlights
* The theory behind the program is described in NWC TP 6037.
* This application is running on an IBM z/OS machine, however it should be compatible with others.

![Screenshot of PEP Report](https://raw.githubusercontent.com/Haynie-Research-and-Development/pep-for-zos/master/screenshots/peprpt.png)

![Screenshot of PEP JCL](https://raw.githubusercontent.com/Haynie-Research-and-Development/pep-for-zos/master/screenshots/pepjcl.png)

![Screenshot of PEP Input](https://raw.githubusercontent.com/Haynie-Research-and-Development/pep-for-zos/master/screenshots/pepinput.png)

### Files
1. PEP
   1. PEP Fortran source code
2. CRTHERMO
   1. Fortran program to create the THERMO binary file
3. DATA.PEP.PROPEL
   1. Propellent database
4. DATA.PEP.THERMO
   1. Binary reaction product database
5. DATA.PEP.THERMO.INPUT
   1. Input file used to create THERMO from CRTHERMO
6. DATA.PEP.SETUP
   1. Information file used by PEP, you shouldn't have to alter this to run as is
7. DATA.PEP.INPUT.*
   1. Input file examples
7. REPORTS.PEP.*
   1. Sample report files
8. jcl/PEP
   1. JCL to execute the PEP program
   2. Once setup, you should only have to change FORMULA=KNDXIO
9. jcl/CRTHERMO
   1. JCL to execute the CRTHERMO program and create the THERMO database

### Options
1. Up to 10 ingredients may be input by the user to be evaluated.
2. There are also eight other options:
   1. Delete exit calculations
   2. Include ionic species in calculations
   3. Include boost velocities and nozzle design data
   4. Input pressures in atmospheres instead of PSI
   5. Increase numerical precision of species list
   6. Output a list of all combustion species considered
   7. Fix temperature
   8. Debug options

### Use
Four data files are required to operate Propellant Evaluation Program for z/OS
1. THERMO
   1. A binary file containing reaction product specie heat of formation data.
2. PROPEL
   1. A text file containing propellant ingredient data.
   2. Additional ingredients can be added to the END of this file.
   3. New lines must be exactly the same length.
3. SETUP
4. INPUT
   1. The text file containing the user's input data.

### Hints and Problems
1. Propellant weights must be greater than zero.
   1. Sometimes even small amounts of an ingredient (less than 0.5) can cause PEP to fail.
2. The output with the various DEBUG options contains no labels.
3. PEP will print out certain error messages if something is wrong such as the exhaust pressure input greater than the chamber pressure.
4. The initial weight need not equal to 100.
5. Occasionally, a set of ingredients will not converge on the steady state solution. A message indicated the reliability of the results is printed, usually the results are wrong.

### User input data
The 11 data lines required to run PEP are listed below.
If you don't understand Fortran variables, it may be a good idea to read a little. However, it's pretty straight forward if you use my examples.

Remember: The number of combined ingredients cannot exceed 10!
1. Data line #1
   1. FILNAM    A12   File name for output, leave as REPORT
2. Data line #2
   1. USER      A17   Title to be printed as identification on each page of output
3. Data line #3
   1. NIC       I5    Number of coded ingredients to read from PROPEL
   2. NIU       I5    Number of user defined ingredients
   3. IRUN      I%    Number of runs
4. Data line #4
   1. DENEXP    F12.0 Density exponent used in calculating D-ISP (density ISP) (defaults to 1.0)
5. Data line #5
   1. PROPTEMP  F12.0 Temperature of Ingredients (defaults to 298 K).  Used for temperature conditioning effects.
6. Data Line #6 - Quadratic coefficients for solid specific heat.  Used with temperature difference to adjust the system enthalpy for heating or cooling.
   1. CPA       F12.0 (defaults to 0.3 cal/gm K)
   2. CPB       F12.0 (defaults to 0.0 cal/gm K)
   3. CPC       F12.0 (defaults to 0.0 cal/gm K)
7. Data line #7 - 1-yes, 0-no
   1. KR(1)     I1    Delete exit calculations,
   2. KR(2)     I1    Include ionic species in calculations
   3. KR(3)     I1    Include boost velocities and nozzle design data
   4. KR(4)     I1    Input pressures in atmospheres instead of PSIs
   5. KR(5)     I1    "N" orders of magnitude species precision increase, 1-1 more sig. digit, 2-2 more sig. digit, etc.
   6. KR(8)     I1    Output a list of all combustion species considered
   7. KR(7)     I1    Fix chamber temperature, temperature is input in place of exit pressure
   8. IDEBUG    I1    Debug options
8. Data line #8 (used only if IDEBUG on line #7 not equal zero)
   1. KR(11)    I1    Thermo data at every guess
   2. KR(12)    I1    Values of J,M,VF,VB,PR,VA in subroutine TWITCH
   3. KR(13)    I1    Species composition every iteration
   4. KR(14)    I1    Log of equilibrium constant every guess
   5. KR(15)    I1    Classification of species each iteration, from TWITCH
9. Data line #9 (used only if NIC on line #3 not equal zero)
   1. ITAG(I)   10I5  (Note 2) Ingredient code number from PROPEL ingredient code list.  This is the LINE NUMBER of the selected ingredient in the PROPEL file.  Currently, it is also the number printed in the file, but if you add ingredients in the middle of the file, then it will be left to the interested reader to keep track of the line numbers!
10. Data line(s) #10 (repeat NUC times)
   1. 10 BLOK(I)    A30/(Note 3) Ingredient name
   2. JIE(I,L)      6(I3,A2)     Number of atoms of element
   3. JE(I,L)                    Symbol of element (must be upper case letter(s)) - up to 6 elements
   4. DH(I)         F5.0         Heat of formation, cal/gram
   5. RHO(I)        F6.0         Density of ingredient, lb/cu-in
11. Date line(s) #11 (repeat IRUN times)
    1. W1(5)        10E12.0     Chamber pressure, PSI or ATM
    2. W1(6)        (Note 4)    Exhaust pressure, PSI or ATM
    3. WING(I)                  Ingredient weights

### Notes
1. If IDEBUG in data line 7 is 0 then data line 8 is not needed.
2. If NIC in data line 3 is 0 then data line 9 is not needed.
3. If NIU is data line 3 is 0 then data line 10 is not needed. If NIU in data line 3 is not 0 then data line 10 is actually NIU data lines, one for each user defined ingredient.  These data lines must be formatted exactly as specified, ie.  A30, 6(I3,A2), F5.0, F6.0.
4. The WING(I) variable contains the ingredient weights in the order they were input.  Example:  If NIC=2 and NIU=1 there are 3 weights, the first two corresponding to NIC=2 and the last one corresponding to NIU=1.  In addition, data line 11 is actually IRUN data lines, one corresponding to each run.
5. Data lines 1, 2, 3, 4, 5, 6, 7, and at least 9 or 10 and line 11 are always input.
6. When the fix temperature is used the exit pressure is .01 ATM

### Notes on Output
1. The gram-atom amounts for each chemical element are based on the given system weight.
2. The enthalpy has units of kilocalories per system weight and the entropy has units of calories/K per system weight.
3. GAS identifies the number of moles of gas produced per system weight. Effective molecular weight is obtained by dividing GAS into system weight.  Note that although non-gases are not included in this computation this is the proper molecular weight to use in gas dynamic equations.
4. RT/V = [ R(0.08205 l-atm/mole/K) * T(K) ]/V (system vol. in liters)
5. The damped and undamped speed of sound is calculated assuming static chemical equilibrium conditions. These numbers are equal if no solids or liquids are present.
6. The chamber and exhaust composition follow in units of moles per system weight.  If one prefers to obtain partial pressures in atmospheres, multiply each composition by RT/V.  Species are solid if & follows the specie name and liquid if * follows the specie name.
7. The first line of the performance results refers to frozen flow, no chemical reactions, through the nozzle; and the second line refers to shifting flow, reactions in equilibrium, through the nozzle.  The theoretical exhaust velocity can be found by multiplying ISP by 9.806 m/sec**2.
8. IS EX is the isentropic exponent such that PV**IS EX = constant for isentropic flow near the nozzle throat.  The values of IS EX and CP/CV do not agree, because the gas in not perfect.
9. The variables T* and P* are throat temperature (in K) and pressure (in atmospheres).
10. C* is the characteristic velocity in ft/sec, the nozzle thrust coefficient, CF = 32.17 * ISP / C*.
11. ISP* is the vacuum impulse to be obtained from a sonic nozzle. That term is used in air-breathing propulsion work.
12. OPT EX is the ratio of the nozzle exit area to nozzle throat area at which exit pressure equals ambient pressure.
13. D-ISP is the density ISP.
14. A*M. is the ratio of nozzle throat area to mass flow rate expressed as in**2-sec/lb.
15. EX T is the exit plane temperature in degrees K.
16. Optional output includes boost velocities. These are shown in number pairs:  the first is the switch density and the second is the velocity in ft/sec.  Inputted densities follow in psi.  The next output shows the performance of the propellant through nozzles with expansion ratios of 1 to 100.  These include three kinds of impulses:  optimum (ambient pressure = exit pressure), vacuum (zero exit pressure), and sea level (exit pressure = 1 atm). Units are given in SI units as well as the older English units. Note that all impulses need to be corrected for nozzle half angle.
