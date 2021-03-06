---
title: "DTI Analysis using rcamino for HCP data"
author: "John Muschelli"
date: "2017-12-06"
output: 
  html_document:
    keep_md: true
    theme: cosmo
    toc: true
    toc_depth: 3
    toc_float:
      collapsed: false
    number_sections: true      
---

All code for this document is located at [here](https://raw.githubusercontent.com/muschellij2/neuroc/master/DTI_analysis_rcamino_hcp/index.R).



# Resources and Goals


Much of this work has been adapted by the Tutorial for DTI analysis from ISMRM 2015: [http://camino.cs.ucl.ac.uk/index.php?n=Tutorials.ISMRM2015](http://camino.cs.ucl.ac.uk/index.php?n=Tutorials.ISMRM2015).  Also, some of the model fitting (such as using gradient information) has been taken from http://camino.cs.ucl.ac.uk/index.php?n=Tutorials.HCP.

We will show you a few steps that have been implemented in `rcamino`: `camino_pointset2scheme`, `camino_modelfit`, `camino_fa`, `camino_md`, and `camino_dteig`.  

# Data Location


# Reading in the Data
First, we download the data from HCP.  You must have your access keys set (see [Getting Data from the Human Connectome Project (HCP)](../neurohcp/index.html)).  

We will use the `neurohcp` package to download one subject data.


```r
library(neurohcp)
hcp_id = "100307"
r = download_hcp_dir(
  paste0("HCP/", hcp_id, "/T1w/Diffusion"), 
  verbose = FALSE)
print(basename(r$output_files))
```

```
[1] "bvals"                   "bvecs"                  
[3] "data.nii.gz"             "grad_dev.nii.gz"        
[5] "nodif_brain_mask.nii.gz"
```

It contains 4 files:

1.  `data.nii.gz` - a 4D image of the DWI data.
2.  `nodif_brain_mask.nii.gz` - A brain mask of the DTI data
3.  `bvals` - a text file with the b-values
4.  `bvecs` - a text file with the b-vectors as the first 3 columns. 


## Creating 


As `dtifit` requires the b-values and b-vectors to be separated, and this data has b-values of $1000$ when the b-vectors is not zero.  **This is very important and you must know where your b-values and b-vectors are when doing your analyses and what units they are in.**  



```r
library(rcamino)
camino_set_heap(heap_size = 10000)
outfiles = r$output_files
names(outfiles) = nii.stub(outfiles, bn = TRUE)
scheme_file = camino_fsl2scheme(
  bvecs = outfiles["bvecs"], bvals = outfiles["bvals"],
  bscale = 1)
```

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/fsl2scheme -bvecfile /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b5fd527a4//bvecs -bvalfile /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b5fd527a4//bvals -bscale 1   >  /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b517d29e01.scheme
```

The imaging scheme contains measurements at b=5, b=1000, b=2000, and b=3000 s / mm^2.

## Subsetting data
By selecting a subset of the measurements, we can reduce processing time and memory requirements. Also, the high b-value shells aren't optimal for estimating the diffusion tensor. So we'll select data from the b=5 and b=1000 shells, which is still higher angular resolution than most DTI (90 directions).

If you have sufficient RAM, you can load the whole data set and extract a subset:

```r
camino_ver = packageVersion("rcamino")
if (camino_ver < "0.5.2") {
  source("https://neuroconductor.org/neurocLite.R")
  neuroc_install("rcamino")  
}
sub_data_list = camino_subset_max_bval(
  infile = outfiles["data"],
  schemefile = scheme_file,
  max_bval = 1500,
  verbose = TRUE) 
```

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/split4dnii -inputfile /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b5fd527a4/data.nii.gz -outputroot /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b5439819d9_
```

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/image2voxel -imagelist '/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b536c9183a.txt' -outputfile '/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b54c55a3ef.Bfloat' -outputdatatype float
```

```r
sub_data = sub_data_list$image
sub_scheme = sub_data_list$scheme
```

This process may take a while, and using the `RNifti` package may be quicker.


```r
nim = RNifti::readNifti(outfiles["data"])
sub_data = tempfile(fileext = ".nii.gz")
sub_scheme_res = camino_subset_max_bval_scheme(
  schemefile = scheme_file, max_bval = 1500,
  verbose = TRUE)
nim = nim[,,, sub_scheme$keep_files]
RNifti::writeNifti(image = nim, file = sub_data)
sub_scheme = sub_scheme_res$scheme
rm(list = "nim");
for (i in 1:10) gc(); 
```


# Fit the diffusion tensor


```r
# wdtfit caminoProc/hcp_b5_b1000.Bfloat caminoProc/hcp_b5_b1000.scheme \
# -brainmask 100307/T1w/Diffusion/nodif_brain_mask.nii.gz -outputfile caminoProc/wdt.Bdouble
# 
mod_file = camino_modelfit(
  infile = sub_data, scheme = sub_scheme, 
  mask = outfiles["nodif_brain_mask"], 
  gradadj = outfiles["grad_dev"],
  model = "ldt_wtd")
```

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/modelfit -inputfile '/private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b54c55a3ef.Bfloat' -outputfile '/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b548434e0f.Bdouble' -inputdatatype float -schemefile /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b51867c0a3.scheme -bgmask /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b5fd527a4/nodif_brain_mask.nii.gz -maskdatatype float -model ldt_wtd -gradadj /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b5fd527a4/grad_dev.nii.gz
```


## Getting FA vlaues


```r
# fa -inputfile caminoProc/wdt_dt.nii.gz -outputfile caminoProc/wdt_fa.nii.gz
fa_img = camino_fa_img(
  infile = mod_file,
  header = outfiles["nodif_brain_mask"],
  retimg = FALSE)
```

```
cat '/private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b548434e0f.Bdouble' |  /Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/fa -inputmodel dt -outputdatatype double > '/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b51173dae.Bdouble'
```

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/voxel2image -inputfile /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b51173dae.Bdouble -header /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b5fd527a4/nodif_brain_mask.nii.gz -outputroot /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b51cd26b11_ -components 1 -gzip 
```





### Visualizing FA images

We want to read the FA image into `R`:

```r
fa_nii = readnii(fa_img)
```

In order to visualize the values, we are going to read in the mask so that we don't visualize non-brain values:

```r
mask = readnii(outfiles["nodif_brain_mask"])
```


```r
hist(mask_vals(fa_nii, mask = mask), breaks = 1000)
```

![](index_files/figure-html/fa_hist-1.png)<!-- -->

Using `ortho2`, we can visualize these FA maps:

```r
ortho2(fa_nii)
```

![](index_files/figure-html/ortho_fa-1.png)<!-- -->



## Getting MD vlaues


```r
# md -inputfile caminoProc/wdt_dt.nii.gz -outputfile caminoProc/wdt_md.nii.gz
md_img = camino_md_img(
  infile = mod_file,
  header = outfiles["nodif_brain_mask"],
  retimg = FALSE)
```

```
cat '/private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b548434e0f.Bdouble' |  /Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/md -inputmodel dt -outputdatatype double > '/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b51845b1d4.Bdouble'
```

```
/Library/Frameworks/R.framework/Versions/3.4/Resources/library/rcamino/camino/bin/voxel2image -inputfile /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b51845b1d4.Bdouble -header /private/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T/RtmpuyYwcz/file94b5fd527a4/nodif_brain_mask.nii.gz -outputroot /var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b579de1bf_ -components 1 -gzip 
```



### Visualizing MD images

We want to read the MD image into `R`:

```r
md_nii = readnii(md_img)
```


```r
hist(mask_vals(md_nii, mask = mask), breaks = 1000)
```

![](index_files/figure-html/md_hist-1.png)<!-- -->

```r
md2 = md_nii
md2[ md2 < 0] = 0
hist(mask_vals(md2, mask = mask), breaks = 1000)
```

![](index_files/figure-html/md_hist-2.png)<!-- -->

Using `ortho2`, we can visualize these MD maps:

```r
ortho2(md_nii)
```

![](index_files/figure-html/ortho_md-1.png)<!-- -->

```r
ortho2(md2)
```

![](index_files/figure-html/ortho_md-2.png)<!-- -->

```r
rb_md = robust_window(md2, probs = c(0, 0.9999))
ortho2(rb_md)
```

![](index_files/figure-html/ortho_md-3.png)<!-- -->


```r
hist(mask_vals(rb_md, mask = mask), breaks = 1000)
```

![](index_files/figure-html/md_hist2-1.png)<!-- -->

# Export DTs to NIfTI

Using `camino_dt2nii`, we can export the diffusion tensors into NIfTI files.  We see the result is the filenames of the NIfTI files, and that they all exist (otherwise there'd be an errors.) 

```r
# dt2nii -inputfile caminoProc/wdt.Bdouble -header 100307/T1w/Diffusion/nodif_brain_mask.nii.gz \
# -outputroot caminoProc/wdt_
mod_nii = camino_dt2nii(
  infile = mod_file,
  header = outfiles["nodif_brain_mask"])
```


```r
# dteig -inputfile caminoProc/wdt.Bdouble -outputfile caminoProc/wdt_eig.Bdouble
eigen_image = camino_dteig(infile = mod_file)
```


We can read these DT images into `R` again using `readnii`, but we must set `drop_dim = FALSE` for diffusion tensor images because the pixel dimensions are zero and `readnii` assumes you want to drop "empty" dimensions


```r
dt_imgs = lapply(mod_nii, readnii, drop_dim = FALSE)
```

## Downloading T1 image

For image registration to a template, we will use the subject-level 

```r
r_t1_mask = download_hcp_file(
  file.path(
    "HCP", hcp_id, "T1w", 
    "brainmask_fs.nii.gz"), 
  verbose = FALSE
)
print(r_t1_mask)
```

```
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/brainmask_fs.nii.gz"
```

```r
t1_mask = readnii(r_t1_mask)
r_t1 = download_hcp_file(
  file.path(
    "HCP", hcp_id, "T1w", 
    "T1w_acpc_dc_restore.nii.gz"), 
  verbose = FALSE
)
print(r_t1)
```

```
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/T1w_acpc_dc_restore.nii.gz"
```

```r
t1 = readnii(r_t1)
brain = mask_img(t1, t1_mask)
hist(mask_vals(brain, t1_mask), breaks = 2000)
```

![](index_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
rob = robust_window(brain, probs = c(0, 0.9999), mask = t1_mask)
hist(mask_vals(rob, t1_mask), breaks = 2000)
```

![](index_files/figure-html/unnamed-chunk-2-2.png)<!-- -->


## Rigid-body Registration of DTI to T1

Here we can register the FA image to the T1-weighted image using a rigid-body transformation.   We could have also used the MD image or the diffusion data directly, such as the mean over the tensors.

```r
library(extrantsr)
```

```
Warning: multiple methods tables found for 'pixdim'

Warning: multiple methods tables found for 'pixdim'
```

```

Attaching package: 'extrantsr'
```

```
The following objects are masked from 'package:oro.nifti':

    origin, origin<-
```

```r
rigid = registration(
  filename = fa_img,
  template.file = rob,
  correct = FALSE,
  verbose = FALSE,
  typeofTransform = "Rigid")
rigid_trans = rigid$fwdtransforms
aff = R.matlab::readMat(rigid$fwdtransforms)
aff = aff$AffineTransform.float.3.3

double_ortho(rob, rigid$outfile)
```

![](index_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Alternatively, we can also do a brain mask to brain mask transformation, which we can estimate using a last squares metric.  This should be sufficient for what we need and more robust to artifacts in the T1 or the FA map, so we'll use this transformation.


```r
rigid_mask = registration(
  filename = outfiles["nodif_brain_mask"],
  template.file = r_t1_mask,
  correct = FALSE,
  typeofTransform = "Rigid",
  affMetric = "meansquares")
```

```
# Running Registration of file to template
```

```
# Applying Registration output is
```

```
$fwdtransforms
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b54f457f480GenericAffine.mat"

$invtransforms
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b54f457f480GenericAffine.mat"
```

```
# Applying Transformations to file
```

```
# Writing out file
```

```
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b5242842fb.nii.gz"
```

```
# Reading data back into R
```

```r
rigid_mask_trans = rigid_mask$fwdtransforms

aff_mask = R.matlab::readMat(rigid_mask$fwdtransforms)
aff_mask = aff_mask$AffineTransform.float.3.3

double_ortho(t1_mask, rigid_mask$outfile, NA.x = FALSE)
```

![](index_files/figure-html/mask_rig-1.png)<!-- -->

## Non-linear Registration of T1 to template

Here we will use symmetric normalization (SyN) to register the Winsorized skull-stripped brain image of the HCP subject to the Eve template.  


```r
library(EveTemplate)
eve_brain_fname = EveTemplate::getEvePath(what = "Brain")
eve_brain = readnii(eve_brain_fname)
nonlin = registration(
  filename = rob,
  template.file = eve_brain_fname,
  correct = FALSE,
  typeofTransform = "SyN")
```

```
# Running Registration of file to template
```

```
# Applying Registration output is
```

```
$fwdtransforms
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b535a1df4b1Warp.nii.gz"      
[2] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b535a1df4b0GenericAffine.mat"

$invtransforms
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b535a1df4b0GenericAffine.mat" 
[2] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b535a1df4b1InverseWarp.nii.gz"
```

```
# Applying Transformations to file
```

```
# Writing out file
```

```
[1] "/var/folders/1s/wrtqcpxn685_zk570bnx9_rr0000gr/T//RtmpuyYwcz/file94b51bcb23e6.nii.gz"
```

```
# Reading data back into R
```

```r
double_ortho(eve_brain, nonlin$outfile)
```

![](index_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
nonlin_trans = nonlin$fwdtransforms
```


### Registering DTI to Eve

Now, we can use the transformed images from the rigid-body transformations above and apply this non-linear transformation to those FA and MD registered images.  The one problem is that the rigid-body registration interpolates the data and the non-linear registration interpolates the data.  

We can compose the transforms so that the data is only interpolated once. 

In `ants_apply_transforms`, which calls `antsApplyTransforms`, the transform list must be specified in reverse order to which they are done.  We want to perform the rigid body transformation then the non-linear registration, but need the composed list of transforms to first have the non-linear transformation then the rigid-body transformation.

Here we apply this composed transformation to the FA and MD values.


```r
composed = c(nonlin_trans, rigid_mask_trans)
fa_eve = ants_apply_transforms(
  fixed = eve_brain_fname,
  moving = fa_img,
  transformlist = composed)
double_ortho(eve_brain, fa_eve)
```

![](index_files/figure-html/composed-1.png)<!-- -->

```r
md_eve = ants_apply_transforms(
  fixed = eve_brain_fname,
  moving = rb_md,
  transformlist = composed)
double_ortho(eve_brain, md_eve)
```

![](index_files/figure-html/composed-2.png)<!-- -->

Now we can perform this in a number of subjects and then do a population-level analysis in the template space. 



