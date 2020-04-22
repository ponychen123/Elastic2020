#!/usr/bin/env python
#%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%#
#%!%!% --------------------------------- ElaStic_Analyze_B0 -------------------------------- %!%!%#
#%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%#
#
# AUTHORS:
# Rostam Golesorkhtabar and Pasquale Pavone 
# r.golesorkhtabar@gmail.com
# 
# DATE:
# Sun Jan 01 00:00:00 2012
#
# SYNTAX:
# python ElaStic_Analyze_B0.py
#        ElaStic_Analyze_B0
# 
# EXPLANATION:
# The parameters in equation of states:
# --- Murnaghan ---
#     E(V) = E0 + B0*V/B' * [(V0/V)^B' / (B'-1) + 1] - B0 V0 / (B'-1)
#     P(V) = B0/B'*((V0/V)**B' -1)
#
# --- Birch-Murnaghan ---
#     E(V) = E0 + 9B0V0/16{[(V0/V)^(2/3)-1]^3 B' + [(V0/V)^(2/3)-1]^2 [6-4(V0/V)^(2/3)]}
#     P(V) = 3/2*B0*((V0/V)**(7/3) - (V0/V)**(5/3))*(1 + 3/4*(B'-4)*[(V0/V)**(2/3) - 1])
#
# E0: Energy in equilibrium
# V0: Volume in equilibrium
# B0: Bulk modulus in equilibrium
# B': Pressure derivative of B
#
# See also: http://en.wikipedia.org/wiki/Birch-Murnaghan_equation_of_state
#__________________________________________________________________________________________________
import os
import sys
import copy
import scipy
import numpy as np
import matplotlib.pyplot as plt
from   scipy.optimize import fmin_powell

#%!%!%--- CONSTANTS ---%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!
_e    =  1.602176565e-19         # elementary charge
Bohr  =  5.291772086e-11         # Bohr to meter
Ryd2eV= 13.605698066             # Ryd to eV
Ha2Ry =  2.                      # Ha to Ryd
ToGPa = (_e*Ryd2eV)/(1e9*Bohr**3)# Ryd/[Bohr]^3 to GPa
#__________________________________________________________________________________________________

#%!%!%--- SUBROUTINS AND FUNCTIONS ---%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%
def E_eos(p0, V):
    if (eos=='M'):
        """ Murnaghan Energy"""
        E0, V0, B0, Bp = p0
        E = E0 + (B0*V/Bp*(1/(Bp-1)*(V0/V)**Bp +1)-B0*V0/(Bp-1))
    else:
        """ Birch-Murnaghan Energy"""
        E0, V0, B0, Bp = p0
        E = E0 + (9.*B0*V0/16)*(((((V0/V)**(2./3))-1.)**3.)*Bp \
               + ((((V0/V)**(2/3.))-1.)**2.)*(6.-4.*((V0/V)**(2./3.))))
    return E
#--------------------------------------------------------------------------------------------------

def snr(p0, v, e):
    """ Squared norm of residue vector calculation """
    return np.sum((e - E_eos(p0, v))**2.)
#--------------------------------------------------------------------------------------------------

def sortarray(ary1, ary2, ary3):
    lst1 = ary1.tolist()
    lst2 = ary2.tolist()
    lst3 = ary3.tolist()

    lst4 = []
    lst5 = []
    lst6 = []

    temp = copy.copy(lst1)
    temp.sort()

    for i in range(len(lst1)):
        lst4.append(lst1[lst1.index(temp[i])])
        lst5.append(lst2[lst1.index(temp[i])])
        lst6.append(lst3[lst1.index(temp[i])])

    ary4 = scipy.array(lst4)
    ary5 = scipy.array(lst5)
    ary6 = scipy.array(lst6)

    return ary4, ary5, ary6
#--------------------------------------------------------------------------------------------------

#%!%!%--- INPUT FILE READING ---%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%
print'\
\n     +-----------------------------------------------------------------+\
\n     |*****************************************************************|\
\n     |*                                                               *|\
\n     |*          WELCOME TO THE "ElaStic_Analyze_B0" PROGRAM          *|\
\n     |*        ElaStic Version 1.0.0, Release Date: 2012-01-01        *|\
\n     |*                                                               *|\
\n     |*****************************************************************|\
\n     +-----------------------------------------------------------------+'

INF = raw_input('\n>>>> Enter "strain volume energy" file name: ')
if (os.path.exists(INF)==False):
    sys.exit('\n.... Oops ERROR: There is NO '+ INF +' file !?!?!?    \n')

si, vi, ei = np.loadtxt(INF).T
si, vi, ei = sortarray(si, vi, ei)

if (len(ei) < 3): sys.exit('\n.... Oops ERROR: EOS fit needs at least 3 points.    \n')

#%!%!%--- DAIMENTION READING ---%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%
print '\
\n     Please specify the "volume unit".\
\n     [Bohr^3] ---=> 1                 \
\n     [Ang.^3] ---=> 2                 '
num = input(">>>> Choose '1' or '2': ")
if (num != 1 and num != 2 ): sys.exit("\n.... Oops ERROR: Choose '1' or '2' \n")
if (num == 1): v_unit = 'Bohr^3'; v2Bhr= 1.
if (num == 2): v_unit = 'Ang.^3'; v2Bhr= (Bohr*1e+10)**-3.

print '\
\n     Please specify the "energy unit".\
\n     Rydberg  -------=> 1             \
\n     Hartree  -------=> 2             \
\n     electron Volt --=> 3             '
num = input(">>>> Choose '1', '2', or '3': ")
if (num != 1 and num != 2 and num != 3 ):
    sys.exit("\n.... Oops ERROR: Choose '1', '2', or '3'  \n")
if (num == 1): e_unit = 'Ryd'; e2Ryd= 1.
if (num == 2): e_unit = 'Ha' ; e2Ryd= Ha2Ry
if (num == 3): e_unit = 'eV' ; e2Ryd= 1./Ryd2eV

vi  = vi*v2Bhr
ei  = ei*e2Ryd
sii = copy.copy(si)
vii = copy.copy(vi)
eii = copy.copy(ei)
#--------------------------------------------------------------------------------------------------

#%!%!%--- BULK MODULUS CALCULATIONS ---%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!
BM_out  = open('B0-vs-strain.dat', 'w')
Bp_out  = open('Bp-vs-strain.dat', 'w')

print >>BM_out, '#   Max. eta      Bulk modulus (GPa)\n#'
print >>Bp_out, '#   Max. eta            dB0/dp      \n#'

for eos in ['M', 'BM']:
    if (eos == 'M' ): 
        print >>BM_out, '\n# Murnaghan fit.'
        print >>Bp_out, '\n# Murnaghan fit.'
    if (eos == 'BM'):
        print >>BM_out, '\n# Birch-Murnaghan fit.'
        print >>Bp_out, '\n# Birch-Murnaghan fit.'

    si = copy.copy(sii)
    vi = copy.copy(vii)
    ei = copy.copy(eii)

    while(len(si) > 3): 
        emax= max(si)
        emin= min(si)
        emax= max(abs(emin), abs(emax))

        a2, a1, a0 = np.polyfit(vi, ei, 2)
        V0   = -a1/(2.*a2)
        E0   = a2*V0**2. + a1*V0 + a0
        B0   = a2*V0
        Bp   = 2.0
        p0   = [E0, V0, B0, Bp]

        viei = sorted([zip(vi, ei)])
        v, e = np.array(viei).T

        p1, fopt, direc, n_iter, n_funcalls, warnflag = \
        fmin_powell(snr, p0, args=(v, e), full_output=True)
        E0, V0, B0, Bp = p1
        B0_GPa = B0*ToGPa

        print >>BM_out, '%13.10f'%emax, '%18.6f'%B0_GPa
        print >>Bp_out, '%13.10f'%emax, '%18.6f'%Bp

        si = si.tolist()
        vi = vi.tolist()
        ei = ei.tolist()

        if (abs(si[0]+emax) < 1.e-7):
            si.pop(0)
            vi.pop(0)
            ei.pop(0)
        if (abs(si[len(si)-1]-emax) < 1.e-7):
            si.pop()
            vi.pop()
            ei.pop()
        si = scipy.array(si)
        vi = scipy.array(vi)
        ei = scipy.array(ei)

BM_out.close()
Bp_out.close()
#--------------------------------------------------------------------------------------------------

#%!%!%--- PLOT PREPARATION ---%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%!%
B0_par  = open('B0.par', 'w')
Bp_par  = open('Bp.par', 'w')
print >>B0_par,'                                              \
# Grace project file                                          \
\npage size 840, 595                                          \
\nwith g0                                                     \
\n    view 0.300, 0.130, 1.350, 0.930                         \
\n    subtitle "Murnaghan and Birch-Murnaghan fit .vs. strain"\
\n    subtitle font 4                                         \
\n    subtitle size 1.200                                     \
\n    xaxis  label "\\xh\\N\\f{}\\smax\\N"                    \
\n    xaxis  label font 4                                     \
\n    xaxis  label char size 2.25                             \
\n    xaxis  label place spec                                 \
\n    xaxis  label place 0.000, 0.090                         \
\n    xaxis  tick place rounded true                          \
\n    xaxis  tick major linewidth 3.5                         \
\n    xaxis  tick minor linewidth 3.5                         \
\n    xaxis  ticklabel offset auto                            \
\n    xaxis  ticklabel char size 1.60                         \
\n    xaxis  ticklabel font 4                                 \
\n    xaxis  ticklabel start type auto                        \
\n    xaxis  ticklabel stop type auto                         \
\n    yaxis  label "Bulk modulus (GPa)"                       \
\n    yaxis  label font 4                                     \
\n    yaxis  label char size 1.600                            \
\n    yaxis  label place spec                                 \
\n    yaxis  label place 0.000, 0.150                         \
\n    yaxis  tick place rounded true                          \
\n    yaxis  tick major linewidth 3.5                         \
\n    yaxis  tick minor linewidth 3.5                         \
\n    yaxis  ticklabel offset auto                            \
\n    yaxis  ticklabel char size 1.60                         \
\n    yaxis  ticklabel font 4                                 \
\n    yaxis  ticklabel start type auto                        \
\n    yaxis  ticklabel stop type auto                         \
\n    legend 0.89, 0.88                                       \
\n    legend font 4                                           \
\n    legend char size 1.500                                  \
\n    legend box linewidth 2.5                                \
\n    frame linewidth 3.5                                     \
\n    s0 symbol 1                                             \
\n    s0 symbol size 0.750                                    \
\n    s0 symbol color 4                                       \
\n    s0 symbol fill color 4                                  \
\n    s0 symbol fill pattern 1                                \
\n    s0 symbol char 65                                       \
\n    s0 line linewidth 3.0                                   \
\n    s0 line color 4                                         \
\n    s0 legend  "Murnaghan"                                  \
\n    s1 symbol 2                                             \
\n    s1 symbol size 0.750                                    \
\n    s1 symbol color 2                                       \
\n    s1 symbol fill color 2                                  \
\n    s1 symbol fill pattern 1                                \
\n    s1 symbol char 65                                       \
\n    s1 line linewidth 3.0                                   \
\n    s1 line color 2                                         \
\n    s1 legend  "Birch-Murnaghan"                            '
B0_par.close()

print >>Bp_par,'                                              \
# Grace project file                                          \
\npage size 840, 595                                          \
\nwith g0                                                     \
\n    view 0.300, 0.130, 1.350, 0.930                         \
\n    subtitle "dB/dP .vs. strain"                            \
\n    subtitle font 4                                         \
\n    subtitle size 1.200                                     \
\n    xaxis  label "\\xh\\N\\f{}\\smax\\N"                    \
\n    xaxis  label font 4                                     \
\n    xaxis  label char size 2.25                             \
\n    xaxis  label place spec                                 \
\n    xaxis  label place 0.000, 0.090                         \
\n    xaxis  tick place rounded true                          \
\n    xaxis  tick major linewidth 3.5                         \
\n    xaxis  tick minor linewidth 3.5                         \
\n    xaxis  ticklabel offset auto                            \
\n    xaxis  ticklabel char size 1.60                         \
\n    xaxis  ticklabel font 4                                 \
\n    xaxis  ticklabel start type auto                        \
\n    xaxis  ticklabel stop type auto                         \
\n    yaxis  label "dB/dP"                                    \
\n    yaxis  label font 4                                     \
\n    yaxis  label char size 1.600                            \
\n    yaxis  label place spec                                 \
\n    yaxis  label place 0.000, 0.150                         \
\n    yaxis  tick place rounded true                          \
\n    yaxis  tick major linewidth 3.5                         \
\n    yaxis  tick minor linewidth 3.5                         \
\n    yaxis  ticklabel offset auto                            \
\n    yaxis  ticklabel char size 1.60                         \
\n    yaxis  ticklabel font 4                                 \
\n    yaxis  ticklabel start type auto                        \
\n    yaxis  ticklabel stop type auto                         \
\n    legend 0.89, 0.88                                       \
\n    legend font 4                                           \
\n    legend char size 1.500                                  \
\n    legend box linewidth 2.5                                \
\n    frame linewidth 3.5                                     \
\n    s0 symbol 1                                             \
\n    s0 symbol size 0.750                                    \
\n    s0 symbol color 4                                       \
\n    s0 symbol fill color 4                                  \
\n    s0 symbol fill pattern 1                                \
\n    s0 symbol char 65                                       \
\n    s0 line linewidth 3.0                                   \
\n    s0 line color 4                                         \
\n    s0 legend  "Murnaghan"                                  \
\n    s1 symbol 2                                             \
\n    s1 symbol size 0.750                                    \
\n    s1 symbol color 2                                       \
\n    s1 symbol fill color 2                                  \
\n    s1 symbol fill pattern 1                                \
\n    s1 symbol char 65                                       \
\n    s1 line linewidth 3.0                                   \
\n    s1 line color 2                                         \
\n    s1 legend  "Birch-Murnaghan"                            '
Bp_par.close()

os.system('xmgrace B0-vs-strain.dat -param B0.par -saveall B0-vs-strain.agr &')
os.system('xmgrace Bp-vs-strain.dat -param Bp.par -saveall Bp-vs-strain.agr &')
#--------------------------------------------------------------------------------------------------
