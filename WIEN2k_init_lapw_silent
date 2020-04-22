#!/bin/csh -f
unalias rm

set name  = $0
#set bin   = $name:h  #directory of WIEN-executables
#if !(-d $bin) set bin = .
set bin = $WIENROOT
set name  = $name:t   #name of this script-file
set logfile = :log
set tmp   = (:$name)  #temporary files

#---> functions & subroutines
alias testinput  'if (! -e \!:1 || -z \!:1) goto \!:2'
alias teststatus 'if ($status) goto error'
alias teststop   'if (\!:1 == $stopafter ) goto stop;'\
                 'if (-e stop) goto stop'
alias output 'set date = `date +"(%T)"`;'\
             'printf ">   %s\t%s " "\!:*" "$date" ;'\
             'printf "\n>   %s\t%s " "\!:*" "$date" >> $dayfile'

alias exec  '($bin/x \!:*) ;'\
            'teststatus'

alias total_exec 'output \!:*;'\
                 'exec  \!:*;'\
                 'teststop \!:*'

#alias editor emacs
if($?EDITOR) then
    alias editor '$EDITOR'
else
  alias editor emacs
endif
alias editor 'ls -l'

#---> default parameters
set next #set -> start cycle with $next
set stopafter

#---> default flags
unset help #set -> help output
# options for batch usage
unset batch
unset red
unset vxc
unset ecut
#in1
unset in1_edit
unset in1_rkmax
unset in1_maxl
unset in1_vnmt
unset in1_emin
unset in1_emax
unset in1_nband
#in2
unset in2_edit
unset in2_method
unset in2_smear
unset in2_gmax
#inm
unset mix
#kgen
set kgen_numk = 1000
set kgen_shift = 1
unset kgen_x
unset kgen_y
unset kgen_z
#dstart
set sp=n
#inM
set inM_create = 0
set inM_method = "PORT"
set inM_tolf = 2.0
set inM_step0 = 0.25

#---> handling of input options
echo ">   ($name) options: $argv" >> $logfile
alias sb 'shift; breaksw' #definition used in switch
while ($#argv)
  switch ($1)
  case -[H|h]:
    set help; sb
  case -[b]:
    set batch; sb
  case -red: 
    shift; set red = $1; sb
  case -vxc: 
    shift; set vxc = $1; sb
  case -ecut: 
    shift; set ecut = $1; sb

  case -in1_rkmax: 
    shift; set in1_rkmax = $1; set in1_edit = 1; sb
  case -in1_maxl: 
    shift; set in1_maxl = $1; set in1_edit = 1; sb
  case -in1_vnmt: 
    shift; set in1_vnmt = $1; set in1_edit = 1; sb
  case -in1_emin: 
    shift; set in1_emin = $1; set in1_edit = 1; sb
  case -in1_emax: 
    shift; set in1_emax = $1; set in1_edit = 1; sb
  case -in1_nband: 
    shift; set in1_nband = $1; set in1_edit = 1; sb

  case -in2_method:
    shift; set in2_method = $1; set in2_edit = 1; sb
  case -in2_smear:
    shift; set in2_smear = $1; set in2_edit = 1; sb
  case -in2_gmax:
    shift; set in2_gmax = $1; set in2_edit = 1; sb

  case -mix: 
    shift; set mix = $1; sb

  case -kgen_numk: 
    shift; set kgen_numk = $1; sb
  case -kgen_shift:
    shift; set kgen_shift = $1; sb
  case -kgen_mesh:
    shift; set kgen_x = $1 ; shift; set kgen_y = $1; shift; set kgen_z = $1; sb
  case -kgen_y:
    shift; set kgen_y = $1; sb
  case -kgen_z:
    shift; set kgen_z = $1; sb

  case -inM_method:
    set inM_create = 1; shift; set inM_method = $1; sb
  case -inM_tolf:
    set inM_create = 1; shift; set inM_tolf = $1; sb
  case -inM_step0:
    set inM_create = 1; shift; set inM_step0 = $1; sb

  case -sp:
    set sp=y; sb

  case -e: 
    shift; set stopafter = $1; sb
  case -s:
    shift; set next  = $1; sb
  default: 
    echo "ERROR: option $1 does not exist!!"; sb
  endsw
end
if ($?help) goto help

#echo $batch
#echo "        red = " $red
#echo "        vxc = " $vxc
#echo "       ecut = " $ecut
##in1
#echo "  in1_rkmax = " $in1_rkmax
#echo "   in1_maxl = " $in1_maxl
#echo "   in1_vnmt = " $in1_vnmt
#echo "   in1_emin = " $in1_emin
#echo "   in1_emax = " $in1_emax
#echo "  in1_nband = " $in1_nband
##in2
#echo " in2_method = " $in2_method
#echo "  in2_smear = " $in2_smear
#echo "   in2_gmax = " $in2_gmax
##inm
#echo "        mix = " $mix
##klist
#echo "  kgen_numk = " $kgen_numk
#echo " kgen_shift = " $kgen_shift
##dstart
#echo "         sp = " $sp
##exit

#---> path- and file-names
set file    = `pwd`
set file    = $file:t       #tail of file-names
set dayfile = $file.dayfile #main output-file

#---> starting out
if ($next != "") goto start #start with optional program
set next = 'setrmt'         #default start with lstart
if (-e $dayfile) then       #start with last program
  echo "" >> $dayfile
  set next = `grep '>' $dayfile |tail -1l |awk '{print $2}'`
endif
if ($next == "") then
   set next = 'setrmt'              #default start with nn
else if ($next == 'stop' ) then
   set next = 'setrmt'              #default start with nn
else if !($next == 'setrmt') then 
   set b=r
   if (! $?batch) then
   echo "continue with " $next  "or restart with setrmt (c/r)"
   set b=($<)
   endif
   if ($b == 'r') then 
   set next='setrmt'
   endif
endif

start: #initalization of input-files

printf "\n\n    start \t(%s) " "`date`"	> $dayfile
echo next is $next 
goto $next

setrmt:
testinput	$file.struct error
if (! $?batch) then
  echo "Automatic determination of RMTs. Please specify the desired RMT reduction "
  echo "compared to almost touching spheres."
  echo "Typically, for a single calculation just hit enter, for force minimization"
  echo "use 1-5; for volume effects you may need even larger reductions."
  echo " "
  echo "Enter reduction in %"
  set red=($<)
endif
if ($?red ) then
  if ($red == '') then
    setrmt_lapw $file 
  else
    setrmt_lapw $file -r $red
  endif
  set b
  if (! $?batch) then
    echo "Do you want to accept these radii; discard them; or rerun setRmt (a/d/r):"  
    set b=($<)
  endif
  if ($b == 'r') then
        goto setrmt
  else if ($b == 'd') then
        goto nn
  else
        cp $file.struct_setrmt $file.struct
  endif
endif
  
nn:
testinput $file.struct error
if (! $?batch) then
  total_exec nn
else
  echo 2 | $bin/x nn   
  teststatus    
endif
set bb1 = `grep WARNING: $file.outputnn`
set bb2 = `grep ERROR  $file.outputnn`
if (! $?batch) then
  echo "-----> check in " $file.outputnn " for overlapping spheres, "
  echo "       coordination and nearest neighbor distances" 
  editor $file.outputnn
endif
if ( $#bb1 != 0 ) then
if ($?batch) then
  set b=y
else
  echo "-----> DO YOU WANT TO USE THE NEW" $file.struct_nn "file (y/n)"
  set b=($<)
endif
  if ($b == 'y') then 
    cp $file.struct $file.struct_init
    cp $file.struct_nn $file.struct
    echo " Original struct file saved to" $file.struct_init
    echo " CREATE A NEW" $file.inst "FILE with PROPER ATOMS"
    if (-e $file.inst) rm -f $file.inst
    instgen_lapw
    goto nn
  endif
endif
if ($?batch) then
  set b=c
else
  echo "-----> continue with sgroup or edit the" $file.struct "file (c/e)" 
  set b=($<)
endif
if ($b == 'e') then 
  editor $file.struct
  goto nn
endif
if ( $#bb2 != 0 ) then
  echo "WARNING: YOU SHOULD PROBABLY MODIFY YOUR STRUCT FILE (see " $file.outputnn ")"
  #exit 9
endif

sgroup:
total_exec sgroup 
grep 'of point group' $file.outputsgroup | grep -v this | grep -v Short 
grep 'space group' $file.outputsgroup | grep -v Note
grep '\!\!' $file.outputsgroup 
set restart_cond = `grep -c 'warning:' $file.outputsgroup`
if (! $?batch) then
  echo "-----> check in " $file.outputsgroup " for proper symmetry, compare"
  echo "       with your struct file and later with " $file.outputs
  editor $file.outputsgroup
  echo "       sgroup has also produced a new struct file based on your old one."
  echo "       If you see warnings above, consider to use the newly generated "
  echo "       struct file, which you can view (edit) now."
  echo "-----> continue with symmetry or edit $file.struct_sgroup ? (c/e)" 
  set b=($<)
  if ($b == 'e') then 
    editor $file.struct_sgroup
    echo "-----> Do you want to use the new struct file (and generate new $file.inst) ? (y/n) "    
    set bb=($<)
    if ($bb == 'y') then 
      cp $file.struct_sgroup $file.struct
      echo " generating a new $file.inst file:"
      if (-e $file.inst) rm -f $file.inst
      instgen_lapw
      goto nn
    endif
  endif
else
  if ($restart_cond != 0) then
    cp $file.struct_sgroup $file.struct
    echo " generating a new $file.inst file:"
    if (-e $file.inst) rm -f $file.inst
    instgen_lapw
    goto nn
  endif
endif

symmetry:
testinput $file.struct error
if (-e $file.in2_sy ) rm $file.in2_sy
total_exec symmetry
set bb = `grep SHIFTED $file.outputs`
if ( $#bb == 9 ) then
  echo "WARNING: YOU MUST MOVE THE ORIGIN (see " $file.outputs ")"
  if ($?batch) then
    exit (3)
  endif
endif
if (! $?batch) then
  echo "-----> check in " $file.outputs " the symmetry operations, "
  echo "       the point symmetries and compare with results from sgroup"
  editor $file.outputs 
  echo "-----> continue with lstart or edit the" $file.struct_st "file (c/e/x)" 
  set b=($<)
  if ($b == 'e') then 
    editor $file.struct_st
    mv $file.struct_st $file.struct
    goto symmetry
  endif
  if ($b == 'x') then 
    if (-e $file.struct_st ) then
      cp $file.struct $file.struct_orig
      cp $file.struct_st $file.struct
    endif
    exit (0)
  endif
  if ( $#bb == 9 ) then
  echo "STOP: YOU MUST MOVE THE ORIGIN OF THE UNIT CELL (see " $file.outputs ")"
  exit 9
  endif
endif

if ($stopafter == 'lstart') then
     exit
endif
lstart:
if (! -e $file.inst) instgen_lapw
testinput $file.inst error
if (! $?batch) then
  total_exec lstart
else
  if(! $?vxc) set vxc=13
  if(! $?ecut) set ecut=-6.
  $bin/x lstart -d
  lstart lstart.def <<EOF
$vxc
$ecut
EOF
  teststatus
endif
set bb=`grep nstop $file.outputst`
if ( $#bb > 2 ) then
echo "ERROR \!\!\! $bb"
echo "You have to change your atomic configuration in $file.inst "
endif
set bb=`grep IZ $file.outputst`
if ( $#bb > 2 ) then
echo "ERROR \!\!\! $bb"
echo "You have to change your atomic configuration in $file.inst or Z in $file.struct "
endif
set bb=`grep WARNING $file.outputst`
if ( $#bb > 2 ) then
grep WARNING $file.outputst
endif

if (! $?batch) then
  echo "       check in " $file.outputst " how much core charge leaks out  "
  echo "       eventually you need to select a smaller ECORE or larger spheres"
  editor $file.outputst
  echo "-----> continue with kgen or edit the" $file.inst "file (c/e)" 
  set b=($<)
  if ($b == 'e') then 
    editor $file.inst
    goto lstart
  endif
endif
cat $file.in2_ls $file.in2_sy > $file.in2_st

inputfiles:
if (! $?batch) then
  echo "-----> in " $file.in1_st " select   RKmax ( usually 5.0 - 9.0 )" 
  editor $file.in1_st
  echo "-----> in " $file.in2_st " select   LM's, GMAX and Fermi-Energy method" 
  editor $file.in2_st
  echo "-----> in " $file.inm_st " Reduce mixing factor and PW-scaling for difficult cases (localized magnetic systems)"
  editor $file.inm_st
else

  if($?in1_edit) then
     if (! $?in1_rkmax) set in1_rkmax = `head -n 2 $file.in1_st | tail -n 1 | awk '{print $1}'`
     if (! $?in1_maxl)  set in1_maxl  = `head -n 2 $file.in1_st | tail -n 1 | awk '{print $2}'`
     if (! $?in1_vnmt)  set in1_vnmt  = `head -n 2 $file.in1_st | tail -n 1 | awk '{print $3}'`
     if (! $?in1_emin)  set in1_emin  = `tail -n 1 $file.in1_st | sed s/"K-VECTORS FROM UNIT:."//g | awk '{print $1}'`
     if (! $?in1_emax)  set in1_emax  = `tail -n 1 $file.in1_st | sed s/"K-VECTORS FROM UNIT:."//g | awk '{print $2}'`
     if (! $?in1_nband) set in1_nband = `tail -n 1 $file.in1_st | sed s/"K-VECTORS FROM UNIT:."//g | awk '{print $3}'`

     set n_lines = `grep -c . $file.in1_st`
     set n_head = `echo "$n_lines-1" | bc -l`
     set n_tail = `echo "$n_lines-3" | bc -l`

     head -n 1 $file.in1_st >.in
     echo $in1_rkmax $in1_maxl $in1_vnmt | awk '{printf "%6.2f%9i%5i%32s\n", $1,$2,$3," (R-MT*K-MAX; MAX L IN WF, V-NMT"}' >>.in
     head -n $n_head $file.in1_st | tail -n $n_tail >>.in
     set in1_unit = `tail -n 1 $file.in1_st | sed s/"K-VECTORS FROM UNIT:"//g | awk '{print $1}'`
     echo $in1_unit $in1_emin $in1_emax $in1_nband | awk '{printf "%20s%1s%7.1f%10.1f%6i", "K-VECTORS FROM UNIT:",$1,$2,$3,$4}' >> .in
     echo "   emin/emax/nband" >> .in

     mv $file.in1_st $file.in1_st_lstart
     mv .in $file.in1_st
  endif

  if($?in2_edit) then
     if (! $?in2_method) set in2_method = `head -n 3 $file.in2_st | tail -n 1 | awk '{print $1}'`
     if (! $?in2_smear)  set in2_smear  = `head -n 3 $file.in2_st | tail -n 1 | awk '{print $2}'`
     if (! $?in2_gmax)   set in2_gmax   = `tail -n 2 $file.in2_st | head -n 1 | awk '{print $1}'`

     set n_lines = `grep -c . $file.in2_st`
     set n_head = `echo "$n_lines-2" | bc -l`
     set n_tail = `echo "$n_lines-5" | bc -l`

     head -n 2 $file.in2_st >.in
     echo $in2_method $in2_smear | awk '{printf "%-5s%10.5f%46s\n", $1,$2,"          (GAUSS,ROOT,TEMP,TETRA,ALL      eval)"}' >>.in
     head -n $n_head $file.in2_st | tail -n $n_tail >>.in
     echo $in2_gmax | awk '{printf "%6.2f%14s\n", $1,"          GMAX"}' >>.in
     tail -n 1 $file.in2_st >>.in

     cp $file.in2_st $file.in2_st_0
     mv .in $file.in2_st
  endif

  if($?mix) then
    sed "2s/^...../$mix /" $file.inm_st >.in
    mv .in $file.inm_st
  endif

endif


output inputfiles prepared
echo ' '
  if (-e $file.struct_st ) then
      cp $file.struct $file.struct_orig
      cp $file.struct_st $file.struct
  endif
  if (-e $file.in0_st ) cp $file.in0_st $file.in0
  if (-e $file.in1_st ) cp $file.in1_st $file.in1
  if (-e $file.in2_st ) cp $file.in2_st $file.in2
  if (-e $file.inc_st ) cp $file.inc_st $file.inc
  if (-e $file.inm_st ) cp $file.inm_st $file.inm
  if (-e $file.inq_st ) cp $file.inq_st $file.inq

set b = `grep INVERSION $file.outputs`
if ( $#b == 7 ) then
output inputfiles for lapw1c/2c prepared, no inversion present
echo ' '
  if (-e $file.in1 ) mv $file.in1 $file.in1c
  if (-e $file.in2 ) mv $file.in2 $file.in2c
endif

kgen:
testinput $file.struct error #. 
if (! $?batch) then
  total_exec kgen
  echo "-----> check in " $file.klist " number of generated K-points" 
  editor $file.klist 
  echo "-----> continue with dstart or execute kgen again or exit (c/e/x)" 
  set b=($<)
  if ($b == 'e') then 
    #  editor $file.outputkgen
    goto kgen
  endif
  if ($b == 'x') exit (0) 
else
  set inversion
  if (-e $file.in1c ) set inversion=1
  $bin/x kgen -d
####$inversion and long output now automatically
  if ($?kgen_x) then
    kgen kgen.def <<EOF
0
$kgen_x $kgen_y $kgen_z
$kgen_shift
EOF
    teststatus
  else
    kgen kgen.def <<EOF
$kgen_numk
$kgen_shift
EOF
    teststatus
  endif
endif

dstart:
set complex
set compl
testinput $file.in1c dstart1
set complex=-c
set compl=c
dstart1:
testinput $file.in1$compl error
total_exec dstart $complex
set bb1 = `grep :WARN $file.outputd`
echo $bb1
if (! $?batch) then
  echo "-----> check in " $file.outputd " if gmax > gmin, normalization" 
  editor $file.outputd 
  if ( $#bb1 != 0 ) then
    echo "Do you want to edit $file.in2$compl, increase GMAX, and rerun dstart (y/n)"
    set b=($<)
    if ($b != 'n') then 
      editor $file.in2$compl
      goto dstart
    endif
  endif
endif
  if (  ! -z $file.in0_std && -e $file.in0_std ) then  
   cp $file.in0_std $file.in0
   echo "-----> new $file.in0 generated" 
  endif
   cp $file.clmsum new_super.clmsum
if (! $?batch) then
  echo "-----> do you want to perform a spinpolarized calculation ? (n/y)" 
  set sp=($<)
endif
if ($sp == 'y') then 
  total_exec dstart -up $complex
  if (! $?batch) then
    editor $file.outputdup
  endif
  total_exec dstart -dn $complex
  if (! $?batch) then
    editor $file.outputddn
  endif
   cp $file.clmup new_super.clmup
   cp $file.clmdn new_super.clmdn
else
# fix nband in case.in1
  set a=`grep K-VECTORS $file.in1_st|grep -v red|cut -c21-`
  set aa=`echo $a[4] |grep -e \[0-9\]`     # get only numbers
  @ a2 = $aa / 2
  sed "s/ $aa / $a2   red /" $file.in1_st > .tmp
  mv .tmp  $file.in1_st
  cp $file.in1_st $file.in1$compl
  goto stop
endif

if ( $?batch) goto stop
echo "-----> do you want to perform an antiferromagnetic calculation ? (N/y)" 
set b=($<)
if ($b != 'y') goto stop 

afminput:
echo "I hope you have        flipped the spin of the AF-atom"
echo "                       made zero spin for non-magnetic atoms"
echo "in $file.inst "
echo "-----> do you want to continue or edit $file.inst ? (c/e)"
set b=($<)
if ($b == 'e') then 
  editor $file.inst
  goto lstart
endif
total_exec  afminput
echo "You can now use     runafm_lapw   for scf"
echo "BUT  PLEASE  NOTE:  AFMINPUT and CLMCOPY  are NOT WELL TESTED"
echo "You must test your results with an unconstraint  runsp_lapw  afterwards" 
echo "and recheck the rules generated by afminput"
editor $file.outputafminput
editor $file.inclmcopy_st
cp $file.inclmcopy_st $file.inclmcopy

stop: #normal exit

#inM
if ( $inM_create == 1 ) then
   echo $inM_method $inM_tolf $inM_step0 | awk '{printf "%-4s%5.2f%5.2f\n", $1, $2, $3}' > $file.inM
   set n_noneq_atom = `head -n 2 $file.struct | tail -n 1 | cut -c 28-30`
   set i = 1
   while ( $i <= $n_noneq_atom )
      echo "1.0 1.0 1.0 1.0" >> $file.inM
      @ i++
   end
   echo "-----> $file.inM generated"
endif

printf "%s\n\n" "init_lapw finished" >> $dayfile
printf "\n   stop\n" >> $dayfile
exit 0 

error: #error exit
printf "\n   stop error\n" >> $dayfile
exit 1

help: #help exit 
cat << theend

PROGRAM: $0

PURPOSE: initialisation of the l/apw-package WIEN2k
         to be called within the case-directory
         has to be located in WIEN-executable directory
         needs case.struct  file

USAGE:   $name [OPTIONS] [FLAGS]

FLAGS:
-h/-H -> help
-b    -> batch (non-interactive) mode (see possible options below, SGROUP is always ignored !)
-sp   -> in batch mode: select spin-polarized calculation

OPTIONS:
-red X                --> in batch mode: RMT reduction by X % (default: RMT not changed)
-vxc X                --> in batch mode: VXC option (default: 13 = PBE )
-ecut X               --> in batch mode: energy seperation between core/valence (default: -6.0 Ry)
-in1_rkmax X          --> in batch mode: RKMAX  (default: 7.0, not changed)
-in1_maxl X           --> in batch mode: MAXL
-in1_vnmt X           --> in batch mode: V-NMT
-in1_emin X           --> in batch mode: EMIN
-in1_emax X           --> in batch mode: EMAX
-in1_nband X          --> in batch mode: NBAND

-in2_method METHOD    --> in batch mode: Integration METHOD
-in2_smear X          --> in batch mode: smearing
-in2_gmax X           --> in batch mode: GMAX

-mix X                --> in batch mode: set mixing to X  (default: 0.2, not changed)

-kgen_numk X          --> in batch mode: use X k-points in full BZ (default: 1000)
-kgen_mesh Xx Xy Xy   --> in batch mode: use Xx, Xy, and Xz k-points in full BZ
-kgen_shift X         --> in batch mode: X=1: shift; X=0: don't shift

-inM_method METHOD    --> in batch mode: minimization method
-inM_tolf X           --> in batch mode: force tolerance
-inM_step0 X          --> in batch mode: size of first geometry step

-s PROGRAM            --> start with PROGRAM ($next)
-e PROGRAM            --> exit after PROGRAM ($stopafter)

theend

exit 1
