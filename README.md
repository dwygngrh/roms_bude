# roms_bude
# running_roms_server

# tutorial running roms di server brin untuk Made

- install mobaxterm
- login ke server 
ssh -X yoga@103.116.175.4
password : yoga123

- perhatikan di dalam server ada 4 hardisk, Jangan taro file running di /home!!!!

/home

/mnt/disksdd1/

/mnt/disksdd2/

/mnt/disksdd3/

# PRE PROCESSING

Seluruh source code di letakkan di /home
dilarang meletakkan data apapun di /home kecuali file program

Siapkan nama simulasi. 

nama_simulasi=Budee_tide

- file program ROMS:
/home/yoga/CROCO/Simulation/{nama_simulasi}

buat folder nama_simulasi:

mkdir {nama_simulasi}

rsync -av master/* {nama_simulasi}/

matlab

perhatikan untuk close semua windows di matlab!!!!

klik start

- Persiapan running 

buat data forcing atmosfir , buat data forcing laut, buat data forcing pasang surut

buat di matlab

sesuaikan crocotools_param.m

CROCO_title  = 'Budee_tide';
CROCO_config = 'Budee_tide';

lonmin =  117.0;   % Minimum longitude [degree east]

lonmax = 135.6;   % Maximum longitude [degree east]

latmin = -9.6;   % Minimum latitude  [degree north]

latmax = 7.32;   % Maximum latitude  [degree north]

dl = 1/24; % ambil grid dari grid builder

N = 50;

theta_s    =  7.;
theta_b    =  1.;
hc         = 100.;
vtransform =  2.; % s-coordinate type (1: old- ; 2: new- coordinates)
                  % ! take care to define NEW_S_COORD cpp-key in cppdefs.h 

topo_smooth =  2; % 1: old ; 2: new filter (better but slower)

hmin = 20;

hmax_coast = 500;

hmax = 7500; % cek kedalaman maksimum diwilayah model

rtarget = 0.25;

n_filter_deep_topo=4;

n_filter_final=2;

coastfileplot = 'coastline_f.mat';
coastfilemask = 'coastline_f_mask.mat';
Roa=0;

interp_method = 'spline'; % Interpolation method: 'linear' or 'spline'

makeplot     = 1;         % 1: create graphics after each preprocessing step

- Pembuatan grid

bisa langsung di make_grid di croco

atau bisa buat di grid builder, copy file gridnya ke direntori CROCO_FILES

masukan nama grid hasil grid builder di crocotools_param.m 

crvname  = [CROCO_files_dir,'grid_anszel_bali.nc'];

- buat forcing laut

make_OGCM_mercator 

- buat forcing atmosfir

cd /home/yoga/CROCO/croco_tools

cd Aforc_ERA5_anszel

sesuaikan di era5_crocotools_param.py

python ERA5_request.py

kalau sudah selesai

python ERA5_convert.py

kembali ke matlab

make_ERA5

- file input running (forcing) ROMS :
/mnt/disksdd3/CROCO_IN/{nama_simulasi}

- file running ROMS :
/mnt/disksdd3/CROCO_RUN/{nama_simulasi}

- file output simulasi : 
/mnt/disksdd2/CROCO_OUT/{nama_simulasi}

# RUNNING

masuk ke direktori /home/yoga/CROCO/Simulation/{nama_simulasi}

yang diubah : 

nano cppdefs.h

-#if defined REGIONAL
/*
!====================================================================
!               REGIONAL (realistic) Configurations
!====================================================================
!
!----------------------
! BASIC OPTIONS
!----------------------
!
*/
                      /* Configuration Name */
-# define Bali

yang diubah :

nano param.h

-#elif defined REGIONAL
-# if defined Bali
      parameter (LLm0=378,   MMm0=478,  N=50)   ! BENGUELA_LR

LLm0=xi_rho-2 MMm0=eta_rho-2

- Compile

./jobcomp

- Siap running
- nano run_croco_inter.bash

- ubah wajib di bawah ini

- Scratch directory where the model is run
- SCRATCHDIR=/mnt/disksdd3/CROCO_RUN/Bali/SCRATCH

- Input directory where the croco_inter.in input file is located
- INPUTDIR=`pwd`  # prod architecture
- #INPUTDIR=`pwd`          # dev architecture

- AGRIF input file which defines the position of child grids
- AGRIF_FILE=AGRIF_FixedGrids.in

- Directory where the croco input NetCDF files (croco_grd.nc, ...) are stored
- MSSDIR=/mnt/disksdd3/CROCO_RUN/Bali/CROCO_FILES

- Directory where the croco output and restart NetCDF files (croco_his.nc, ...) are stored
- MSSOUT=/mnt/disksdd2/CROCO_OUT/Bali/
- DT=180
- Start and End year
- NY_START=2021
- NY_END=2021
- Start and End month
- NM_START=1
- NM_END=12

-  Restart file - RSTFLAG=0 --> No Restart
-                 RSTFLAG=1 --> Restart
- RSTFLAG=0 # untuk running pertama kali RSTFLAG=1 untuk running dari hotstart
- buka croco_inter.in
- title:
-         Selat Bali internal wave
- time_stepping: NTIMES   dt[sec]  NDTFAST  NINFO
-                NUMTIMES     TIMESTEP    NFAST     1
- S-coord: THETA_S,   THETA_B,    Hc (m)
-           6.0d0     1.0d0      200.0d0

- perintah running :
- nohup ./run_croco_inter.bash >> logrun.txt &

- untuk cek running masuk ke direktori : cd /mnt/disksdd3/CROCO_RUN/Bali/SCRATCH/

- tail -f croco_Y2021M01.out 

- ctrl + c untuk keluar dari tail -f


# POST PROCESSING


- Direktori post processing

/mnt/disksdd2/CROCO_POST/Erin/


- pengenalan file output di direktori output
- 
croco_avg_Y*.nc  --> file rata rata harian 3D

croco_his_Y*.nc --> file sesaat per jam 3D

croco_surf_Y*.nc --> file sesaat surface

croco_surf_avg_Y*.nc --> file rata rata harian surface

croco_diags_eddy_avg_Y*.nc --> file eddies rata rata harian

- pembuatan file untuk analisa

rata rata per musim lakukan untuk semua file avg, surf, surf_avg mwnggunakan perintah ncra.

*** contoh  ncra /mnt/disksdd2/CROCO_OUT/Sulawesi_notide/croco_surf_avg_Y*M01.nc /mnt/disksdd2/CROCO_OUT/Sulawesi_notide/croco_surf_avg_Y*M02.nc /mnt/disksdd2/CROCO_OUT/Sulawesi_notide/croco_surf_avg_Y*M03.nc croco_surf_avg_JFM_notide.nc***

ncra croco_avg_Y*M01.nc croco_avg_Y*M02.nc croco_avg_Y*M03.nc croco_avg_JFM_tide.nc

ncra croco_avg_Y*M06.nc croco_avg_Y*M07.nc croco_avg_Y*M08.nc croco_avg_JJA_tide.nc

ncdiff untuk membuat perbedaan antara dua file nectdf

ncra/ncdiff fileinput.nc fileoutput.nc

erin : lakukan rata-rata tahunan

- download data dari marine copernicus untuk sst per bulan di 

https://data.marine.copernicus.eu/product/SST_GLO_SST_L4_NRT_OBSERVATIONS_010_001/download?dataset=METOFFICE-GLO-SST-L4-NRT-OBS-SST-MON-V2

- download data sea surface height per bulan dari :

- https://data.marine.copernicus.eu/product/SEALEVEL_EUR_PHY_L4_NRT_OBSERVATIONS_008_060/download?dataset=dataset-duacs-nrt-europe-merged-allsat-phy-l4

- masukan koordinatnya sesuai dengan lokasi studi 

- untuk erin :

- lonmin =  117.08;   % Minimum longitude [degree east]
- lonmax =  127.08;   % Maximum longitude [degree east]
- latmin = -1.5;   % Minimum latitude  [degree north]
- latmax = 7.97;   % Maximum latitude  [degree north]


- download per bulan 2019-2021

- rata rata perbulan atau per musim sesuai kebutuhan

- berhubung data cmems tidak ada dimensi waktu di netcdf maka harus dilakukan prosedur sbb:

- ncks -O -h --mk_rec_dmn time file_input file_output

- contoh:

- ncks -O -h --mk_rec_dmn time sst_201901.nc sst_201901.nc  

- setelah itu bisa dilakukan rata rata dengan ncra
