#!/bin/tcsh -f
#
# AUTHOR:
# Rostam Golesorkhtabar
# r.golesorkhtabar@gmail.com
#
# DATE:
# Tue Jan 1  00:00:00  2013
#
# SYNTAX:
# ElaStic_Save_Clean_WIEN2k
#
# EXPLANATION:
#
#
set Dir_path = $cwd
set Dirn_list = `ls -d ??t??`
foreach Dirn ($Dirn_list)
   cd $Dirn
   set Dirn_num_list = `ls -d ${Dirn}_??`
   foreach Dirn_num ($Dirn_num_list)
      cd $Dirn_num
      save_lapw ${Dirn_num}_Converged
      clean_lapw -s
      cd ..
   end
   cd ..
end
exit
