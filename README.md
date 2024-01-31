# Gaia Benchmark Stars, enhanced version 3

This enhanced version of the Gaia Benchmark Stars catalogue is based on Table 6 from [Soubiran et al. (2023)](https://ui.adsabs.harvard.edu/abs/2023arXiv231011302S).

Starting from the above mentioned table, I have performed several individual crossmatches, which are saved in the directory *individual_crossmatches* and combined them to one *Master Crossmatch*. Note that not all 192 stars have entries in 2MASS (191, not gam Sge / HIP98337) or Gaia DR3 (179 of 192).

## 1) Master Crossmatch



## 2) Individual Crossmatches

### 2.1) Match with 2MASS identifiers and 2MASS J2000 coordinates via VizieR

I have reformated their Table 6 and used the VizieR ([Ochsenbein et al., 2000](https://ui.adsabs.harvard.edu/abs/2000A&AS..143...23O)) via **gbs_v3_hip_id.txt** to match the HIP identifiers with the closest and brightest 2MASS identifiers and their 2MASS J2000 coordinates via the [VizieR crossmatching interface with catalog II/246/out](https://vizier.cds.unistra.fr/viz-bin/VizieR?-source=II/246). The resulting table is saved as **gbs_v3_tmass_id_original.fits** file and has a two new identifier columns of *tmass_id* (2MASS identifier string) and *hip_int* (Hipparcos identifiers integers) which allow the additional crossmatches on the [Gaia archive](https://gea.esac.esa.int/archive/).

### 2.2) Match with 2MASS

I have uploaded the catalog **gbs_v3_tmass_id_original.fits** in the [Gaia archive](https://gea.esac.esa.int/archive/) as **gbs_v3** and then performed the following crossmatching query with 2MASS data from [Skrutskie et al. (2006)](https://ui.adsabs.harvard.edu/abs/2006AJ....131.1163S):
~~~~sql
SELECT
my_table.hip_int,
tmass.*
FROM gaiadr1.tmass_original_valid as tmass
INNER JOIN
	user_sbuder.gbs_v3 as my_table
	ON my_table.tmass_id = tmass.designation
~~~~

The resulting table is saved as **individual_crossmatches/gbs_v3_tmass-result.fits** and has entries for 189 of the 192 stars. The following stars do not have a 2MASS ID: gam Sge, alf Cen A, alf Cen B.

### 2.3) Match with Gaia DR3

I have uploaded the catalog **gbs_v3_tmass_id_original.fits** in the [Gaia archive](https://gea.esac.esa.int/archive/) as **gbs_v3** and then performed the following crossmatching query with Gaia DR3 by [Gaia Collaboration, Vallenari et al. (2023)](https://ui.adsabs.harvard.edu/abs/2023A&A...674A...1G) as well as the inferred distances from [Bailer Jones et al. (2021)](https://ui.adsabs.harvard.edu/abs/2021AJ....161..147B) via the 2MASS identifier crossmatches by [Torra et al. (2021)](https://ui.adsabs.harvard.edu/abs/2021A&A...649A..10T):
~~~~sql
SELECT
my_table.hip_int,
my_table.tmass_id,
calj.*,
gaia.*
FROM gaiadr3.gaia_source as gaia
LEFT OUTER JOIN
	external.gaiaedr3_distance as calj
	ON calj.source_id = gaia.source_id
LEFT OUTER JOIN
	gaiadr3.tmass_psc_xsc_best_neighbour AS tmassxmatch
	ON tmassxmatch.source_id = gaia.source_id
LEFT OUTER JOIN
	gaiadr3.tmass_psc_xsc_join AS tmass_join
	ON tmass_join.clean_tmass_psc_xsc_oid = tmassxmatch.clean_tmass_psc_xsc_oid
LEFT OUTER JOIN
	gaiadr1.tmass_original_valid as tmass
	ON tmass.designation = tmass_join.original_psc_source_id
INNER JOIN
	user_sbuder.gbs_v3 as my_table
	ON my_table.tmass_id = tmass.designation
~~~~

Because of the missing 2MASS ID for gam Sge (HIP98337), I have performed a special match just with the Gaia DR3 source_id for this star:

~~~~sql
SELECT
calj.*,
gaia.*
FROM gaiadr3.gaia_source as gaia
LEFT OUTER JOIN
	external.gaiaedr3_distance as calj
	ON calj.source_id = gaia.source_id
WHERE
	gaia.source_id = 1823067317695766784
~~~~

I have then concatenated the two resulting table and added the HIP integer as well as 2MASS *None*.
The resulting table is saved as **individual_crossmatches/gbs_v3_gaiadr3-result.fits** and has entries for 180 of the 192 stars.

### 2.4) Match with AllWISE

Because of the lower number of Gaia DR3 matches, I had to use a hybrid crossmatching approach with the AllWISE catalog ([Cutri et al., 2013](http://cdsads.u-strasbg.fr/abs/2014yCat.2328....0C)).

I have performed a positional crossmatch via TOPCAT by [Taylor et al. (2005)](http://adsabs.harvard.edu/abs/2005ASPC..347...29T) with the 2MASS J2000 coordinates within a 1 arc minute circle.

I then inspected complicated matches by hand/eye in the Aladin ([Bonnarel et al., 2000](https://ui.adsabs.harvard.edu/abs/2000A&AS..143...33B)) interface and looked for the correct matches between 2MASS and AllWISE colored images as well as catalog entries. Complicated cases includes matches with more than 3 arsec distance between 2MASS and AllWISE, inconsistent maginutde behaviour between 2MASS and AllWISE or no match at all. I have uploaded a catalog with only HIP and AllWISE IDs (**individual_crossmatches/gbs_v3_allwise_id.fits**) and performed the following crossmatch with it:

~~~~sql
SELECT
my_table.hip_int,
allwise.*
FROM gaiadr1.allwise_original_valid as allwise
INNER JOIN
	user_sbuder.gbs_v3_allwise as my_table
	ON my_table.designation = allwise.designation
~~~~

The resulting table is saved as **individual_crossmatches/gbs_v3_allwise-result.fits** and has entries for 179 of the 192 stars.

I have also checked the crossmatches via Gaia DR3 and 2MASS to be consistent with my results:

~~~~sql
SELECT
my_table.hip_int,
my_table.tmass_id,
gaia.source_id,
allwise.*
FROM gaiadr3.gaia_source as gaia
LEFT OUTER JOIN
	gaiadr3.allwise_best_neighbour AS wisexmatch
	ON wisexmatch.source_id = gaia.source_id
LEFT OUTER JOIN
	gaiadr1.allwise_original_valid as allwise
	ON allwise.allwise_oid = wisexmatch.allwise_oid
LEFT OUTER JOIN
	gaiadr3.tmass_psc_xsc_best_neighbour AS tmassxmatch
	ON tmassxmatch.source_id = gaia.source_id
LEFT OUTER JOIN
	gaiadr3.tmass_psc_xsc_join AS tmass_join
	ON tmass_join.clean_tmass_psc_xsc_oid = tmassxmatch.clean_tmass_psc_xsc_oid
LEFT OUTER JOIN
	gaiadr1.tmass_original_valid as tmass
	ON tmass.designation = tmass_join.original_psc_source_id
INNER JOIN
	user_sbuder.gbs_v3 as my_table
	ON my_table.tmass_id = tmass.designation
~~~~

### 2.5) Match with HIPPARCOS (New Reduction)

I have uploaded the catalog **gbs_v3_tmass_id_original.fits** in the [Gaia archive](https://gea.esac.esa.int/archive/) as **gbs_v3** and then performed the following crossmatching query with the new reduction of Hipparcos by van Leeuwen et al. (2007, http://adsabs.harvard.edu/abs/2007A%26A...474..653V):
~~~~sql
SELECT
vanleeuwen.*
FROM public.hipparcos_newreduction as vanleeuwen
INNER JOIN user_sbuder.gbs_v3 as gbs
	ON gbs.hip_int = vanleeuwen.hip
~~~~

The resulting table is saved as **individual_crossmatches/gbs_v3_hipparcos-result.fits** and has entries for 192 of the 192 stars.