Batch Re-name Files
================

## Motivation: Change *a lot* of file names

Assume I have the following files:

``` r
fs::dir_tree("img")
```

    #> img
    #> ├── 00-538-example-01.png
    #> ├── 00-538-example-02.png
    #> ├── 00-538-html-ref-00.png
    #> ├── 00-538-html-ref-01.png
    #> ├── 00-538-html-ref-02.png
    #> ├── 00-csv-example.png
    #> ├── 00-csv.png
    #> ├── 00-json.png
    #> ├── 00-pudding-office-01.png
    #> ├── 00-pudding-office-02.png
    #> ├── 00-r-code-example-01.png
    #> ├── 00-r-code-example-02.png
    #> ├── 00-r-code-example-03.png
    #> ├── 00-textedit.png
    #> ├── 00-wapo-covid-sparkline.png
    #> ├── 00-wapo-covid.gif
    #> └── 00-wapo-covid.png

I want to change the prefix (`00-`) to `01-` for all of the files. I can
do this with `fs` and `purrr`

## File information

``` r
dir_files <- fs::dir_info("img", regexp = "[.]png$") %>% 
    filter(type == "file") %>% 
    select(path, type, ends_with("time"))
glimpse(dir_files)
```

    #> Rows: 16
    #> Columns: 6
    #> $ path              <fs::path> "img/00-538-example-01.png", "img/00-538-exampl…
    #> $ type              <fct> file, file, file, file, file, file, file, file, file…
    #> $ modification_time <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…
    #> $ access_time       <dttm> 2021-11-24 11:53:26, 2021-11-24 11:53:26, 2021-11-2…
    #> $ change_time       <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:25, 2021-11-2…
    #> $ birth_time        <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…

Create the base name:

``` r
file_names <- dir_files %>% 
    mutate(file_name = basename(path)) %>% 
    select(path, file_name, everything())
glimpse(file_names)
```

    #> Rows: 16
    #> Columns: 7
    #> $ path              <fs::path> "img/00-538-example-01.png", "img/00-538-exampl…
    #> $ file_name         <chr> "00-538-example-01.png", "00-538-example-02.png", "0…
    #> $ type              <fct> file, file, file, file, file, file, file, file, file…
    #> $ modification_time <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…
    #> $ access_time       <dttm> 2021-11-24 11:53:26, 2021-11-24 11:53:26, 2021-11-2…
    #> $ change_time       <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:25, 2021-11-2…
    #> $ birth_time        <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…

Now we replace all prefixes:

``` r
new_file_names <- file_names %>% 
    mutate(new_file_name = 
               str_replace_all(file_name,
                pattern = "^00-", 
                replacement = "01-")) %>% 
    select(path, file_name, new_file_name, everything())
glimpse(new_file_names)
```

    #> Rows: 16
    #> Columns: 8
    #> $ path              <fs::path> "img/00-538-example-01.png", "img/00-538-exampl…
    #> $ file_name         <chr> "00-538-example-01.png", "00-538-example-02.png", "0…
    #> $ new_file_name     <chr> "01-538-example-01.png", "01-538-example-02.png", "0…
    #> $ type              <fct> file, file, file, file, file, file, file, file, file…
    #> $ modification_time <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…
    #> $ access_time       <dttm> 2021-11-24 11:53:26, 2021-11-24 11:53:26, 2021-11-2…
    #> $ change_time       <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:25, 2021-11-2…
    #> $ birth_time        <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…

Add the original file path back to the new file names:

``` r
new_paths <- new_file_names %>% 
    mutate(base_path = str_remove_all(string = path, pattern = file_name),
           new_path = paste0(base_path, new_file_name)) %>% 
    select(path, new_path, file_name, new_file_name, ends_with("time"))
glimpse(new_paths)
```

    #> Rows: 16
    #> Columns: 8
    #> $ path              <fs::path> "img/00-538-example-01.png", "img/00-538-exampl…
    #> $ new_path          <chr> "img/01-538-example-01.png", "img/01-538-example-02.…
    #> $ file_name         <chr> "00-538-example-01.png", "00-538-example-02.png", "0…
    #> $ new_file_name     <chr> "01-538-example-01.png", "01-538-example-02.png", "0…
    #> $ modification_time <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…
    #> $ access_time       <dttm> 2021-11-24 11:53:26, 2021-11-24 11:53:26, 2021-11-2…
    #> $ change_time       <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:25, 2021-11-2…
    #> $ birth_time        <dttm> 2021-11-24 11:53:24, 2021-11-24 11:53:24, 2021-11-2…

## Do it for one element

Now we can extract one element and rename it to see if it works:

``` r
fs::dir_create("img-test")
fs::file_copy(path = "raw-images/00-538-example-01.png", 
              new_path = "img-test/00-538-example-01.png", 
              overwrite = TRUE)
fs::dir_tree("img-test")
```

    #> img-test
    #> └── 00-538-example-01.png

``` r
file.rename(from = "img-test/00-538-example-01.png", 
            to = "img-test/01-538-example-01.png")
```

    #> [1] TRUE

We can check to verify

``` r
fs::dir_tree("img-test", regexp = "01-")
```

    #> img-test
    #> └── 01-538-example-01.png

Now we do it for all the new names in `new_path`.

``` r
old_file_paths <- new_paths$path
head(old_file_paths)
```

    #> img/00-538-example-01.png  img/00-538-example-02.png  
    #> img/00-538-html-ref-00.png img/00-538-html-ref-01.png 
    #> img/00-538-html-ref-02.png img/00-csv-example.png

``` r
new_file_paths <- new_paths$new_path
head(new_file_paths)
```

    #> [1] "img/01-538-example-01.png"  "img/01-538-example-02.png" 
    #> [3] "img/01-538-html-ref-00.png" "img/01-538-html-ref-01.png"
    #> [5] "img/01-538-html-ref-02.png" "img/01-csv-example.png"

``` r
map2(.x = old_file_paths, .y = new_file_paths, file.rename)
```

    #> [[1]]
    #> [1] TRUE
    #> 
    #> [[2]]
    #> [1] TRUE
    #> 
    #> [[3]]
    #> [1] TRUE
    #> 
    #> [[4]]
    #> [1] TRUE
    #> 
    #> [[5]]
    #> [1] TRUE
    #> 
    #> [[6]]
    #> [1] TRUE
    #> 
    #> [[7]]
    #> [1] TRUE
    #> 
    #> [[8]]
    #> [1] TRUE
    #> 
    #> [[9]]
    #> [1] TRUE
    #> 
    #> [[10]]
    #> [1] TRUE
    #> 
    #> [[11]]
    #> [1] TRUE
    #> 
    #> [[12]]
    #> [1] TRUE
    #> 
    #> [[13]]
    #> [1] TRUE
    #> 
    #> [[14]]
    #> [1] TRUE
    #> 
    #> [[15]]
    #> [1] TRUE
    #> 
    #> [[16]]
    #> [1] TRUE

Verify

``` r
fs::dir_tree("img", regexp = "01-")
```

    #> img
    #> ├── 01-538-example-01.png
    #> ├── 01-538-example-02.png
    #> ├── 01-538-html-ref-00.png
    #> ├── 01-538-html-ref-01.png
    #> ├── 01-538-html-ref-02.png
    #> ├── 01-csv-example.png
    #> ├── 01-csv.png
    #> ├── 01-json.png
    #> ├── 01-pudding-office-01.png
    #> ├── 01-pudding-office-02.png
    #> ├── 01-r-code-example-01.png
    #> ├── 01-r-code-example-02.png
    #> ├── 01-r-code-example-03.png
    #> ├── 01-textedit.png
    #> ├── 01-wapo-covid-sparkline.png
    #> └── 01-wapo-covid.png

## Reset `img/`

``` r
# remove contents of img/
fs::dir_delete("img")
# get 00- files from raw-images/
img_00_files <- fs::dir_ls("raw-images/", regexp = "/00-")
# create img again
new_img_path <- fs::dir_create("img/")
# copy files over 
purrr::quietly(map2(.x = img_00_files, .y = new_img_path, 
     .f = fs::file_copy, overwrite = TRUE))
```

    #> function (...) 
    #> capture_output(.f(...))
    #> <bytecode: 0x7faa13e780f8>
    #> <environment: 0x7faa13e77e20>

``` r
# verify
fs::dir_tree("img")
```

    #> img
    #> ├── 00-538-example-01.png
    #> ├── 00-538-example-02.png
    #> ├── 00-538-html-ref-00.png
    #> ├── 00-538-html-ref-01.png
    #> ├── 00-538-html-ref-02.png
    #> ├── 00-csv-example.png
    #> ├── 00-csv.png
    #> ├── 00-json.png
    #> ├── 00-pudding-office-01.png
    #> ├── 00-pudding-office-02.png
    #> ├── 00-r-code-example-01.png
    #> ├── 00-r-code-example-02.png
    #> ├── 00-r-code-example-03.png
    #> ├── 00-textedit.png
    #> ├── 00-wapo-covid-sparkline.png
    #> ├── 00-wapo-covid.gif
    #> └── 00-wapo-covid.png

## `batch_rename_files()`

Bundle as a function:

``` r
batch_rename_files <- function(path, pattern, replace, prefix, extension) {
    
    # get file_ext
    file_ext <- paste0("[.]", extension, "$")
    
    # get dir_files
    dir_files <- fs::dir_info(path = path, regexp = file_ext) 
    dir_files <- dplyr::filter(dir_files, type == "file") 
    dir_files <- dplyr::select(dir_files, path, type, ends_with("time"))
    
    # define pattern
    regex_pattern <- as.character(pattern)
    # define replace
    regex_replace <- as.character(replace)
    
        # prefix
    if (prefix == TRUE) {
        
        regex <- paste0("^", regex_pattern)
         # pattern
    } else { 
        
        regex <- as.character(regex_pattern)
       
    } 

    file_names <- mutate(dir_files, file_name = basename(path))
    
    new_file_names <- mutate(file_names, 
           new_file_name = 
                    stringr::str_replace_all(file_name,
                    pattern = regex, 
                    replacement = regex_replace))
    
    new_paths <- mutate(new_file_names, 
                    base_path = 
                    stringr::str_remove_all(string = path, pattern = file_name),
                    new_path = paste0(base_path, new_file_name)) %>% 
    select(path, new_path, file_name, new_file_name, ends_with("time"))
    
    old_file_paths <- new_paths$path
    new_file_paths <- new_paths$new_path
    
    # return(new_file_paths)
    
    purrr::quietly(purrr::map2(.x = old_file_paths, .y = new_file_paths,
                               file.rename))
    
}
```

## Set up tests

``` r
test_01_path <- "test-01/"
fs::dir_create(test_01_path)
test_01_files <- fs::dir_ls("raw-images/", regexp = "/00-")
purrr::quietly(map2(test_01_files, .y = test_01_path, 
     .f = fs::file_copy, overwrite = TRUE))
```

``` r
test_02_path <- "test-02/"
fs::dir_create(test_02_path)
test_02_files <- fs::dir_ls("raw-images/", regexp = "-1[.]png$")
purrr::quietly(map2(test_02_files, .y = test_02_path, 
     .f = fs::file_copy, overwrite = TRUE))
```

## Test 1

Test renaming `00-` to `01-`:

``` r
batch_rename_files(path = "test-01", pattern = "00-", replace = "01-", prefix = TRUE, extension = "png")
```

    #> function (...) 
    #> capture_output(.f(...))
    #> <bytecode: 0x7faa13e780f8>
    #> <environment: 0x7faa119d4918>

``` r
fs::dir_tree("test-01")
```

    #> test-01
    #> ├── 00-wapo-covid.gif
    #> ├── 01-538-example-01.png
    #> ├── 01-538-example-02.png
    #> ├── 01-538-html-ref-00.png
    #> ├── 01-538-html-ref-01.png
    #> ├── 01-538-html-ref-02.png
    #> ├── 01-csv-example.png
    #> ├── 01-csv.png
    #> ├── 01-json.png
    #> ├── 01-pudding-office-01.png
    #> ├── 01-pudding-office-02.png
    #> ├── 01-r-code-example-01.png
    #> ├── 01-r-code-example-02.png
    #> ├── 01-r-code-example-03.png
    #> ├── 01-textedit.png
    #> ├── 01-wapo-covid-sparkline.png
    #> └── 01-wapo-covid.png

## Reset `test-01/`

``` r
# remove contents of img/
fs::dir_delete("test-01/")
# get 00- files from raw-images/
img_00_files <- fs::dir_ls("raw-images/", regexp = "/00-")
# create img again
test_01_img_path <- fs::dir_create("test-01/")
# copy files over 
purrr::quietly(map2(.x = img_00_files, .y = test_01_img_path, 
     .f = fs::file_copy, overwrite = TRUE))
```

    #> function (...) 
    #> capture_output(.f(...))
    #> <bytecode: 0x7faa13e780f8>
    #> <environment: 0x7faa0d7e2a10>

``` r
# verify
fs::dir_tree("test-01/")
```

    #> test-01/
    #> ├── 00-538-example-01.png
    #> ├── 00-538-example-02.png
    #> ├── 00-538-html-ref-00.png
    #> ├── 00-538-html-ref-01.png
    #> ├── 00-538-html-ref-02.png
    #> ├── 00-csv-example.png
    #> ├── 00-csv.png
    #> ├── 00-json.png
    #> ├── 00-pudding-office-01.png
    #> ├── 00-pudding-office-02.png
    #> ├── 00-r-code-example-01.png
    #> ├── 00-r-code-example-02.png
    #> ├── 00-r-code-example-03.png
    #> ├── 00-textedit.png
    #> ├── 00-wapo-covid-sparkline.png
    #> ├── 00-wapo-covid.gif
    #> └── 00-wapo-covid.png

## Test 2

Test renaming `-1.png` to `.png`:

``` r
batch_rename_files(path = "test-02", pattern = "-1", replace = "", prefix = FALSE, extension = "png")
```

    #> function (...) 
    #> capture_output(.f(...))
    #> <bytecode: 0x7faa13e780f8>
    #> <environment: 0x7faa13e23058>

``` r
fs::dir_tree("test-02")
```

    #> test-02
    #> ├── facet_geo-sol.png
    #> ├── facet_wrap-2-sol.png
    #> ├── facet_wrap-2vars-2.png
    #> ├── facet_wrap-2vars.png
    #> ├── facet_wrap-cols.png
    #> ├── facet_wrap-rows.png
    #> ├── facet_wrap-show.png
    #> ├── facet_wrap-sol.png
    #> ├── facet_wrap.png
    #> ├── facet_wrap_paginate-sol.png
    #> └── plot.png

## Reset `test-02/`

``` r
# remove contents of img/
fs::dir_delete("test-02/")
# get 00- files from raw-images/
img_1_suffix_files <- fs::dir_ls("raw-images/", regexp = "-1[.]png$")
# create img again
test_02_img_path <- fs::dir_create("test-02/")
# copy files over 
purrr::quietly(map2(.x = img_1_suffix_files, .y = test_02_img_path, 
     .f = fs::file_copy, overwrite = TRUE))
```

    #> function (...) 
    #> capture_output(.f(...))
    #> <bytecode: 0x7faa13e780f8>
    #> <environment: 0x7faa158474a0>

``` r
# verify
fs::dir_tree("test-02/")
```

    #> test-02/
    #> ├── facet_geo-sol-1.png
    #> ├── facet_wrap-1-1.png
    #> ├── facet_wrap-2-sol-1.png
    #> ├── facet_wrap-2vars-1-1.png
    #> ├── facet_wrap-2vars-2-1.png
    #> ├── facet_wrap-cols-1-1.png
    #> ├── facet_wrap-rows-1-1.png
    #> ├── facet_wrap-show-1.png
    #> ├── facet_wrap-sol-1.png
    #> ├── facet_wrap_paginate-sol-1.png
    #> └── plot-1.png
