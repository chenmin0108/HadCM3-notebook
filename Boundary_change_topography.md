This document summarises how to modify **HadCM3 land surface boundary conditions, topography-related ancillaries, and deep-time experiment setup** using UMUI + external processing tools.

---

# 1. Where land surface boundary conditions are defined

In UMUI:


Model Selection
→ Atmosphere
→ Ancillary and Input data fields
→ Climatologies and potential climatologies
→ Effective vegetation parameters


You will see:


qrparm.veg (vegetation file)


This file is read from:


$MY_ANCIL


---

# 2. Key directory variables

In UMUI:


Model Selection → Sub Model Independent → File and Directory naming


- `MY_ANCIL` → location of ancillary files


Model Selection → Sub Model Independent → Script inserts and modifications


- `ANCIL_ROOT` → root directory for ancillary processing

---

# 3. Locate and inspect boundary files

Typical location:


/home/swsvalde/ancil/preind2/qrparm.veg


On BlueCrystal:

bash xconv qrparm.veg

So:

~swsvalde instead of /home/swsvalde
# 4. Convert to NetCDF

Copy file locally, then convert:

module add apps/convsh/1.91
module add apps/cdo-1.9.5

um2nc qrparm.veg qrparm.veg.nc

# 5. Edit vegetation / soil fields in R

Example: modify snow-free albedo
library(ncdf4)

nc = nc_open("qrparm.veg_orig.nc")
albedo = ncvar_get(nc, "field322_snp_srf")
lats = ncvar_get(nc, "latitude")
lons = ncvar_get(nc, "longitude")
nc_close(nc)

albedo_new <- albedo

for (yy in 1:73) {
  for (xx in 1:96) {
    if (lats[yy] > 60 & lons[xx] > 180) {
      if (!is.na(albedo[xx,yy])) {
        albedo_new[xx,yy] = 0.74
      }
    }
  }
}

nc = nc_open("qrparm.veg_new.nc", write = TRUE)
ncvar_put(nc, "field322_snp_srf", albedo_new)
nc_close(nc)

# 6. Convert NetCDF back to UM ancillary format

On BlueCrystal:

module add apps/convsh/1.91
module unload vtune
~ggdjl/bin/xancil0.62 &

Then:

Atmosphere ancillary files → vegetation parameters

Steps:

select input NetCDF
ensure variables auto-fill
fix missing variables manually if needed
output name:
qrparm.veg_new

(no .nc extension)

# 7. Use modified boundary file in UMUI


Go to:

Atmosphere → Ancillary and Input data fields → Effective vegetation parameters

Replace:

qrparm.veg

with:

qrparm.veg_new

Then:

Process → Compile → Run
# 8. Deep-time simulation framework (Valdes-type setup)

Experiment structure in UMUI

Search:

Owner = ggdjl

Main folder:

xpti

Contains:

xptiz → long 0Ma-style run (3000 years)
xptia → short 10-year restart run
9. Running a 0Ma test simulation

Copy experiment xptia into your workspace.

Ensure restart structure:

~/dump2hold/

Create directories:

mkdir -p ~/dump2hold/execs_deeptime_djl
mkdir -p ~/dump2hold/dumps_deeptime_djl/tfks

Copy executables:

cp ~ggdjl/dump2hold/execs_deeptime_djl/Scotese_08_djl.exec \
   ~/dump2hold/execs_deeptime_djl/

Copy restart dumps:

cp ~ggdjl/dump2hold/dumps_deeptime_djl/tfks/tfksaa#da000003000c1+ \
   ~/dump2hold/dumps_deeptime_djl/

cp ~ggdjl/dump2hold/dumps_deeptime_djl/tfks/tfksao#da000003000c1+ \
   ~/dump2hold/dumps_deeptime_djl/
10. Key differences for paleo simulations

From comparing xptia and xptib, required changes are:

(1) Solar constant
variable: SOLAR_AGE

choose from:

~ggdjl/scotese/co2_all_04_nt.dat
(2) CO2 concentration

From:

co2_all_04_nt.dat
(3) Land-sea mask / land points

From:

landfrac_nt.dat
(4) Number of land points

From same file:

final column
(5) Restart files

Simulation naming scheme:

tfks[a-z]
tfkS[a-z]
tfKs[a-z]
tfKS[a-z]
tFks[a-e]

Example:

102.6 Ma → tfksu
(6) Island definitions

Location:

~ggdjl/scotese/islands

Each file contains:

Number of islands
Max segments per island
Total segments
(ignore last value)
11. Summary of required changes for paleo runs

To run a new time period you must update:

Solar constant (SOLAR_AGE)
Land/sea mask (MY_ANCIL)
Number of land points
CO2 concentration
Restart dumps
Island configuration
12. Final workflow
Modify vegetation / soil fields (NetCDF + R)
Convert back via xancil
Update UMUI ancillary path
Update deep-time forcing parameters
Set correct restart files
Compile on bp1 / bc5
Run model
