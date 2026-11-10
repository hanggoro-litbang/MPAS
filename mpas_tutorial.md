# Tutorial Instalasi dan Menjalankan MPAS (Global & Regional)

Repositori ini berisi panduan langkah demi langkah untuk menginstal dan menjalankan Model for Prediction Across Scales (MPAS) baik untuk domain Global maupun Regional. (Sesuaikan PATH dengan konfigurasi komputer masing-masing)

---

## DAFTAR ISI
1. [Instalasi MPAS dan Library Pendukung](#bagian-1-instalasi-mpas-dan-library-pendukung)
2. [Menjalankan MPAS Global](#bagian-2-menjalankan-mpas-global)
3. [Menjalankan MPAS Regional](#bagian-3-menjalankan-mpas-regional)

---

## BAGIAN 1: Instalasi MPAS dan Library Pendukung

### 1. Persiapan Environment
Edit file `.bashrc` Anda untuk mengatur environment variables.
```bash
vi ~/.bashrc
```
Tambahkan konfigurasi berikut ke dalam `.bashrc`:
```bash
### MPAS
ulimit -s unlimited

### mpich
export PATH=/mnt-storage2/hpcadm/mpas-lib/bin:$PATH
export LD_LIBRARY_PATH=/mnt-storage2/hpcadm/mpas-lib:$LD_LIBRARY_PATH
export NETCDF=/mnt-storage2/hpcadm/mpas-lib
export PNETCDF=/mnt-storage2/hpcadm/mpas-lib
export PIO=/mnt-storage2/hpcadm/mpas-lib
export NETCDF_PATH=$NETCDF
export MPAS_EXTERNAL_LIBS="-L/mnt-storage2/hpcadm/mpas-lib/lib -lhdf5_hl -lhdf5 -ldl -lz"
export MPAS_EXTERNAL_INCLUDES="-I/mnt-storage2/hpcadm/mpas-lib/include"
```
Aktifkan `.bashrc`:
```bash
source ~/.bashrc
export mpas_lib=/mnt-storage2/hpcadm/mpas-lib
```

### 2. Instalasi Prasyarat (Libraries)

**MPICH**
```bash
wget https://www.mpich.org/static/downloads/4.0.2/mpich-4.0.2.tar.gz
tar -xzvf mpich-4.0.2.tar.gz
cd mpich-4.0.2
./configure --prefix=${mpas_lib}
make -j 4
make install
source ~/.bashrc
```

**Jasper**
```bash
wget https://www.ece.uvic.ca/~frodo/jasper/software/jasper-1.900.1.zip
unzip jasper-1.900.1.zip
cd jasper-1.900.1
./configure --prefix=${mpas_lib} --enable-shared 
make
make install
source ~/.bashrc
```

**LIBPNG**
```bash
wget https://sourceforge.net/projects/libpng/files/libpng16/1.6.37/libpng-1.6.37.tar.gz
tar -zxvf libpng-1.6.37.tar.gz
cd libpng-1.6.37
./configure --prefix=${mpas_lib}
make
make install
```

**ZLIB**
```bash
wget http://www.zlib.net/fossils/zlib-1.2.11.tar.gz
tar xzvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
CC=mpicc CFLAGS=-fPIC ./configure --prefix=${mpas_lib}
make
make install
source ~/.bashrc
```

**SZLIB**
```bash
wget https://support.hdfgroup.org/ftp/lib-external/szip/2.1.1/src/szip-2.1.1.tar.gz
tar xzvf szip-2.1.1.tar.gz
cd szip-2.1.1
CC=mpicc CFLAGS=-fPIC ./configure --prefix=${mpas_lib}
make
make install
source ~/.bashrc
```

**PnetCDF**
```bash
tar xzvf pnetcdf-1.12.3.tar.gz
cd pnetcdf-1.12.3
CC=mpicc FC=mpifort CFLAGS=-fPIC ./configure --prefix=${mpas_lib} --enable-relax-coord-bound
make
make install
source ~/.bashrc
```

**HDF5**
```bash
wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.13/hdf5-1.13.2/src/hdf5-1.13.2.tar.gz
tar xzvf hdf5-1.13.2.tar.gz
cd hdf5-1.13.2
CC=mpicc ./configure --prefix=${mpas_lib} --enable-parallel --with-zlib=${mpas_lib} --with-szlib=${mpas_lib} --enable-fortran
make -j4
make install
source ~/.bashrc
```

**NetCDF-C**
```bash
wget https://downloads.unidata.ucar.edu/netcdf-c/4.9.0/netcdf-c-4.9.0.tar.gz
tar xzvf netcdf-c-4.9.0.tar.gz
cd netcdf-c-4.9.0
LDFLAGS="-L${mpas_lib}/lib" CPPFLAGS="-I${mpas_lib}/include" LIBS="-lhdf5_hl -lhdf5 -lz -ldl" LD_LIBRARY_PATH=${mpas_lib}:$LD_LIBRARY_PATH CC=mpicc FCFLAGS="-m64" FFLAGS="-m64" ./configure --prefix=${mpas_lib} --enable-shared --enable-dap --enable-netcdf4 --enable-pnetcdf --enable-parallel-tests
make
make install
source ~/.bashrc
```

**NetCDF-Fortran**
```bash
wget https://downloads.unidata.ucar.edu/netcdf-fortran/4.6.0/netcdf-fortran-4.6.0.tar.gz
tar xzvf netcdf-fortran-4.6.0.tar.gz
cd netcdf-fortran-4.6.0
LDFLAGS="-L${mpas_lib}/lib" CPPFLAGS="-I${mpas_lib}/include" LIBS="-lhdf5_hl -lhdf5 -lz -ldl -lnetcdf" LD_LIBRARY_PATH=${mpas_lib}:$LD_LIBRARY_PATH CC=mpicc FC=mpifort ./configure --prefix=${mpas_lib} --enable-parallel-tests
make
make install
source ~/.bashrc
```

**Parallel IO (PIO)**
```bash
tar -xvzf pio2_5_9.tar.gz
cd ParallelIO-pio2_5_9
mkdir build
cd build
CC=mpicc FC=mpifort cmake -DNetCDF_C_PATH=${mpas_lib} -DNetCDF_Fortran_PATH=${mpas_lib} -DPnetCDF_PATH=${mpas_lib} -DPIO_HDF5_LOGGING=On -DPIO_USE_MALLOC=On -DPIO_ENABLE_TIMING=OFF -DCMAKE_INSTALL_PREFIX=${mpas_lib} ..
make
make install
source ~/.bashrc
```

### 3. Instalasi Tools & Model MPAS

**MPAS-Model**
```bash
cd $HOME
wget https://github.com/MPAS-Dev/MPAS-Model/archive/v7.3.tar.gz
tar -xvzf v7.3.tar.gz
cd MPAS-Model-7.3
make gfortran CORE=init_atmosphere USE_PIO2=true
ls -la init_atmosphere_model
make clean CORE=atmosphere
make gfortran CORE=atmosphere USE_PIO2=true
ls -la atmosphere_model
```

**WPS**
```bash
cd $HOME
wget https://github.com/wrf-model/WPS/archive/refs/tags/v4.6.0.tar.gz
tar -xvzf v4.6.0.tar.gz
cd WPS-4.6.0
export jasper=${mpas_lib}
export JASPERLIB=${mpas_lib}/lib
export JASPERINC=${mpas_lib}/include
export NETCDF=${mpas_lib}
./configure --nowrf
./compile >& compile.log &
```

**MPAS-Tools, GKlib, dan METIS**
```bash
# MPAS Tools
cd $HOME
wget https://github.com/MPAS-Dev/MPAS-Tools/archive/refs/tags/0.36.0.tar.gz
tar -zxvf 0.36.0.tar.gz
cd MPAS-Tools-0.36.0/mesh_tools/grid_rotate
make

# GKlib
cd $HOME
git clone https://github.com/KarypisLab/GKlib.git
cd GKlib
make config prefix=/mnt-storage2/hpcadm/mpas-lib cc=gcc
make
make install
source ~/.bashrc

# METIS
cd $HOME
git clone https://github.com/KarypisLab/METIS.git
cd METIS
make config shared=1 gklib_path=/mnt-storage2/hpcadm/mpas-lib cc=gcc prefix=/mnt-storage2/hpcadm/metis
make
make install
```

---

## BAGIAN 2: Menjalankan MPAS Global

### 1. Menyiapkan File Static
```bash
cd $HOME
export mpas_tutor=/mnt-storage2/hpcadm/mpas_tutorial
export mpas_model=$HOME/MPAS-Model-7.3
mkdir -p WDIR/240km_uniform
cd WDIR/240km_uniform
ln -sf ${mpas_tutor}/meshes/x1.10242.grid.nc .
ln -sf ${mpas_model}/init_atmosphere_model .
cp ${mpas_model}/namelist.init_atmosphere .
cp ${mpas_model}/streams.init_atmosphere .
```

Edit `namelist.init_atmosphere`:
```text
&nhyd_model
      config_init_case = 7
/
&data_sources
      config_geog_data_path = '/mnt-storage1/src/GEOG'
      config_landuse_data = 'MODIFIED_IGBP_MODIS_NOAH'
      config_topo_data = 'GMTED2010'
      config_vegfrac_data = 'MODIS'
      config_albedo_data = 'MODIS'
      config_maxsnowalbedo_data = 'MODIS'
      config_supersample_factor = 3
/
&preproc_stages
      config_static_interp = true
      config_native_gwd_static = true
      config_vertical_grid = false
      config_met_interp = false
      config_input_sst = false
      config_frac_seaice = false
/
```

Edit `streams.init_atmosphere`:
```xml
<immutable_stream name="input"
                  type="input"
                  filename_template="x1.10242.grid.nc"
                  input_interval="initial_only"/>

<immutable_stream name="output"
                  type="output"
                  filename_template="x1.10242.static.nc"
                  packages="initial_conds"
                  output_interval="initial_only" />
```

Jalankan model inisialisasi:
```bash
./init_atmosphere_model
tail -f log.init_atmosphere.0000.out
```

Plot data statis (opsional):
```bash
cp ${mpas_tutor}/ncl_scripts/plot_terrain.ncl .
vi plot_terrain.ncl 
# ubah fname = getenv("FNAME") menjadi fname = "x1.10242.static.nc"
ncl plot_terrain.ncl
ls -al terrain.pdf
display terrain.pdf
```

### 2. Interpolasi Data Met (GFS)
```bash
cd $HOME/WDIR/240km_uniform
ln -sf ${mpas_tutor}/met_data/GFS\:2014-09-10_00 .
rm -rf log.init_atmosphere*
mkdir bak
cp x1.10242.static.nc bak
```

Edit kembali `namelist.init_atmosphere` untuk tahap interpolasi:
```text
&nhyd_model
      config_init_case = 7
      config_start_time = '2014-09-10_00:00:00'
/
&dimensions
      config_nvertlevels = 41
      config_nsoillevels = 4
      config_nfglevels = 38
      config_nfgsoillevels = 4
/
&data_sources
      config_met_prefix = 'GFS'
      config_use_spechumd = false
/
&vertical_grid
      config_ztop = 30000.0
      config_nsmterrain = 1
      config_smooth_surfaces = true
      config_dzmin = 0.3
      config_nsm = 30
      config_tc_vertical_grid = true
      config_blend_bdy_terrain = false
/
&interpolation_control
      config_extrap_airtemp = 'linear'
/
&preproc_stages
       config_static_interp = false
       config_native_gwd_static = false
       config_vertical_grid = true
       config_met_interp = true
       config_input_sst = false
       config_frac_seaice = true
/
```

Edit `streams.init_atmosphere`:
```xml
<immutable_stream name="input"
                  type="input"
                  filename_template="x1.10242.static.nc"
                  input_interval="initial_only"/>

<immutable_stream name="output"
                  type="output"
                  filename_template="x1.10242.init.nc"
                  packages="initial_conds"
                  clobber_mode="overwrite"
                  output_interval="initial_only" />
```

Jalankan kembali inisialisasi:
```bash
./init_atmosphere_model
```

### 3. Update Sea Surface Temperature (SST)
```bash
ln -sf ${mpas_tutor}/met_data/SST\:2014-09-* .
```
Edit `namelist.init_atmosphere`:
```text
&nhyd_model
     config_init_case = 8
     config_start_time = '2014-09-10_00:00:00'
     config_stop_time = '2014-09-20_00:00:00'
/
&data_sources
     config_sfc_prefix = 'SST'
     config_fg_interval = 86400
/
&preproc_stages
     config_static_interp = false
     config_vertical_grid = false
     config_met_interp = false
     config_input_sst = true
     config_frac_seaice = true
/
```
Edit `streams.init_atmosphere` (tambahkan bagian ini):
```xml
<immutable_stream name="surface"
                  type="output"
                  filename_template="x1.10242.sfc_update.nc"
                  filename_interval="none"
                  packages="sfc_update"
                  output_interval="86400"/>
```
Jalankan proses:
```bash
./init_atmosphere_model
ncdump -v xtime x1.10242.sfc_update.nc
```

### 4. Integrasi Model (Running Model)
```bash
ln -s ${mpas_model}/atmosphere_model .
cp ${mpas_model}/namelist.atmosphere .
cp ${mpas_model}/streams.atmosphere .
cp ${mpas_model}/stream_list.atmosphere.* .
ln -s ${mpas_model}/src/core_atmosphere/physics/physics_wrf/files/* .
```

Edit `namelist.atmosphere`:
```text
&nhyd_model
     config_dt = 1200.0
     config_start_time = '2014-09-10_00:00:00'
     config_run_duration = '5_00:00:00'
     config_len_disp = 240000.0
/
&decomposition
     config_block_decomp_file_prefix = 'x1.10242.graph.info.part.'
/
&physics
     config_radtlw_interval = '01:00:00'
     config_radtsw_interval = '01:00:00'
/
```

Edit `streams.atmosphere`:
```xml
<immutable_stream name="input"
                  type="input"
                  filename_template="x1.10242.init.nc"
                  input_interval="initial_only"/>

<stream name="surface"
        type="input"
        filename_template="x1.10242.sfc_update.nc"
        filename_interval="none"
        input_interval="86400">
    <file name="stream_list.atmosphere.surface"/>
</stream>
```

Jalankan Model:
```bash
ln -sf ${mpas_tutor}/meshes/x1.10242.graph.info.part.4 .
mpiexec -n 4 ./atmosphere_model
```

---

## BAGIAN 3: Menjalankan MPAS Regional (Contoh 3km Jawa)

### 1. Pre-Processing WPS (Ungrib)
```bash
cd $HOME
conda create -n mpas -c conda-forge numpy cartopy netCDF4
export mpas_tutor=/mnt-storage2/hpcadm/mpas_tutorial
export mpas_model=$HOME/MPAS-Model-7.3
mkdir -p WDIR/3km_jawa
./MPAS-Model-7.3/testing_and_setup/atmosphere/setup_atm_run_dir WDIR/3km_jawa
cd WDIR/3km_jawa
ln -sf ../../WPS-4.6.0/ungrib/Variable_Tables/Vtable.GFS Vtable
ln -sf ../../WPS-4.6.0/link_grib.csh .
ln -sf ../../WPS-4.6.0/ungrib.exe .
./link_grib.csh /mnt-storage2/hpcadm/GFS/gfs.0p25.2024092800.f0*
ln -sf ../../WPS-4.6.0/namelist.wps .
```

Edit `namelist.wps`:
```text
&share
 wrf_core = 'ARW',
 max_dom = 2,
 start_date = '2024-09-28_00:00:00','2019-09-04_12:00:00',
 end_date   = '2024-09-29_00:00:00','2019-09-04_12:00:00',
 interval_seconds = 10800
/
&ungrib
 out_format = 'WPS',
 prefix = 'GFS',
/
```
Jalankan Ungrib:
```bash
./ungrib.exe
```

### 2. Membangun Domain (MPAS-Tools & Limited Area)
Rotasi Grid (Pusat di Jawa):
```bash
cd $HOME/MPAS-Tools-0.36.0/mesh_tools/grid_rotate
vi namelist.input
```
Isi `namelist.input`:
```text
&input
   config_original_latitude_degrees = 0
   config_original_longitude_degrees = 0
   config_new_latitude_degrees = -7.231
   config_new_longitude_degrees = 109.769
   config_birdseye_rotation_counter_clockwise_degrees = 0
/
```
Jalankan rotasi grid:
```bash
./grid_rotate /mnt-storage2/hpcadm/meshes/x20.835586.grid.nc $HOME/WDIR/3km_jawa/jawa.global.grid.nc
```

Buat Region Area:
```bash
cd $HOME/MPAS-Limited-Area
cp docs/points-examples/india.circle.pts region.pts
vi region.pts
# Ubah nama titik lokasi dan nilai lat/lon sesuai kebutuhan
conda activate mpas
./create_region region.pts $HOME/WDIR/3km_jawa/jawa.global.grid.nc
# Ini akan membuat file *.grid.nc (misal: jawa.grid.nc)
```

Plot Mesh (Opsional):
```bash
cp /mnt-storage2/hpcadm/mpas_tutorial/ncl_scripts/plot_mesh.ncl .
vi plot_mesh.ncl
# ubah fname = "jawa.grid.nc"
# ubah res@mpCenterLatF = -7. dan res@mpCenterLonF = 109.
conda activate ncl_stable
ncl plot_mesh.ncl
display atm_mesh.pdf
```

Dekomposisi Graph METIS:
```bash
cp jawa.gr* $HOME/WDIR/3km_jawa/
cd $HOME/WDIR/3km_jawa
gpmetis -m -contig -niter=200 jawa.graph.info 4
gpmetis -m -contig -niter=200 jawa.graph.info 32
```

### 3. Setup Model Regional (Static, Terrain Blending, LBC)

**Jalankan Static (Geogrid)**:
```bash
cd $HOME/WDIR/3km_jawa
cp ${mpas_tutor}/namelists_template/namelist.init_atmosphere1 namelist.init_atmosphere
cp ${mpas_tutor}/namelists_template/streams.init_atmosphere1 streams.init_atmosphere
./init_atmosphere_model
# Output: *.static.nc
```

**Jalankan Terrain Blending (Metgrid)**:
```bash
cp ${mpas_tutor}/namelists_template/namelist.init_atmosphere2 namelist.init_atmosphere
cp ${mpas_tutor}/namelists_template/streams.init_atmosphere2 streams.init_atmosphere
./init_atmosphere_model
# Output: *.init.nc
```

**Buat LBC (Lateral Boundary Conditions)**:
```bash
cp ${mpas_tutor}/namelists_template/namelist.init_atmosphere3 namelist.init_atmosphere
cp ${mpas_tutor}/namelists_template/streams.init_atmosphere3 streams.init_atmosphere
./init_atmosphere_model
# Output: lbc.*
```

### 4. Menjalankan Prediksi Regional (Atmosphere Model)
```bash
cd $HOME/WDIR/3km_jawa
cp ${mpas_tutor}/namelists_template/namelist.atmosphere namelist.atmosphere
cp ${mpas_tutor}/namelists_template/streams.atmosphere streams.atmosphere
./atmosphere_model
```

### 5. Post-Processing (Konversi Output ke LatLon)
```bash
ln -sf $HOME/convert_mpas/convert_mpas .
vi target_domain
# Isi koordinat target domain sesuai gambar (misal startlat=-4 endlat=-8 startlon=104 endlon=108 nlat=200 nlon=200)

./convert_mpas jawa.grid.nc diag.2025-03-03_12.00.00.nc
mv latlon diag.2025-03-03_12.00.00_latlon.nc

./convert_mpas jawa.grid.nc history.2025-03-03_12.00.00.nc
mv latlon history.2025-03-03_12.00.00_latlon.nc
```
