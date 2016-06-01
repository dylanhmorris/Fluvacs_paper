FluVacs Figures - Draft
================
Kari and JT
May 2, 2016

-   [Figure 3 Coverage plots](#figure-3-coverage-plots)
-   [Initial data processing](#initial-data-processing)
-   [Frequency of Variants](#frequency-of-variants)
-   [Diversity measures](#diversity-measures)
    -   [SNV count per sample](#snv-count-per-sample)
    -   [Shannon's Entropy](#shannons-entropy)
-   [Heat map](#heat-map)
-   [Antigenic Sites](#antigenic-sites)
    -   [All data](#all-data-2)
    -   [High quality](#high-quality-2)

``` r
# read in the csvs and add a season column to use later
titer.2004.5 <- read.csv("../Titers_status_2004-2005.csv", stringsAsFactors = F)
titer.2004.5$season <- "04-05"
titer.2005.6 <- read.csv("../Titers_status_2005-2006.csv", stringsAsFactors = F)
titer.2005.6$season <- "05-06"
titer.2007.8 <- read.csv("../Titers_status_2007-2008.csv", stringsAsFactors = F)
titer.2007.8$season <- "07-08"
```

``` r
var.2007.8 <- read.csv("../data/processed/Run_1293/Variants/all.sum.csv", stringsAsFactors = F)
x <- read.csv("../data/processed/Run_1304/Variants/all.sum.csv", stringsAsFactors = F)  # the rest of these samples
var.2007.8 <- rbind(var.2007.8, x)  # combine both runs
other.seasons <- read.csv("../data/processed/Run_1412/Variants/all.sum.csv", 
    stringsAsFactors = F)


var.2004.5.df <- processing(data.df = other.seasons, meta.df = titer.2004.5, 
    pval = 0.01, phred = 35, mapq = 30, read_cut = c(32, 94))

var.2004.5.df <- subset(var.2004.5.df, season == "04-05")
var.2005.6.df <- processing(data.df = other.seasons, meta.df = titer.2005.6, 
    pval = 0.01, phred = 35, mapq = 30, read_cut = c(32, 94))
var.2005.6.df <- subset(var.2005.6.df, season == "05-06")

var.2007.8.df <- processing(data.df = var.2007.8, meta.df = titer.2007.8, pval = 0.01, 
    phred = 35, mapq = 30, read_cut = c(32, 94))

all.df <- rbind(var.2004.5.df, var.2005.6.df)
all.df <- rbind(all.df, var.2007.8.df)

all.df <- subset(all.df, freq.var >= 0.01)
##### Now for the duplicate runs #######

var.2004.5.df2 <- read.csv("../data/processed/2004_2005/Variants/all.sum.csv", 
    stringsAsFactors = F)
var.2004.5.df2 <- processing(data.df = var.2004.5.df2, meta.df = titer.2004.5, 
    pval = 0.01, phred = 35, mapq = 30, read_cut = c(32, 94))

var.2005.6.df2 <- read.csv("../data/processed/2005-2006/Variants/all.sum.csv", 
    stringsAsFactors = F)
var.2005.6.df2 <- processing(data.df = var.2005.6.df2, meta.df = titer.2005.6, 
    pval = 0.01, phred = 35, mapq = 30, read_cut = c(32, 94))

var.2007.8.df2 <- read.csv("../data/processed/2007-2008/Variants/all.sum.csv", 
    stringsAsFactors = F)
var.2007.8.df2 <- processing(data.df = var.2007.8.df2, meta.df = titer.2007.8, 
    pval = 0.01, phred = 35, mapq = 30, read_cut = c(32, 94))

### Join duplicates #####

dups.2004.5 <- join_dups(data1.df = var.2004.5.df, data2.df = var.2004.5.df2)
dups.2005.6 <- join_dups(data1.df = var.2005.6.df, data2.df = var.2005.6.df2)
dups.2007.8 <- join_dups(data1.df = var.2007.8.df, data2.df = var.2007.8.df2)

##### Merge duplicates with intial data that was >1e5

qual.2004.5 <- high_qual(data1.df = var.2004.5.df, dups.df = dups.2004.5, titer = 1000)  # only duplicates above 1e3 kept
qual.2005.6 <- high_qual(data1.df = var.2005.6.df, dups.df = dups.2005.6, titer = 1000)
qual.2007.8 <- high_qual(data1.df = var.2007.8.df, dups.df = dups.2007.8, titer = 1000)
all.qual.df <- rbind(qual.2004.5, qual.2005.6)
all.qual.df <- rbind(all.qual.df, qual.2007.8)

all.qual.df <- subset(all.qual.df, freq.var >= 0.01)
```

Figure 3 Coverage plots
-----------------------

These are just from the first runs. I'm not including the duplicates here - should I? These have a sliding window of 100 with a step of 100 no overlap.

``` r
cov.2007.8 <- rbind(read.csv("../data/processed/Run_1293/deepSNV/all.coverage.csv", 
    stringsAsFactors = F), read.csv("../data/processed/Run_1304/deepSNV/all.coverage.csv", 
    stringsAsFactors = F))

other.seasons.cov <- read.csv("../data/processed/Run_1412/deepSNV/all.coverage.csv", 
    stringsAsFactors = F)


cov.2004.5 <- subset(other.seasons.cov, Id %in% titer.2004.5$Sample)
cov.2005.6 <- subset(other.seasons.cov, Id %in% titer.2005.6$Sample)


cov.plot.04.05 <- cov_plot(cov.2004.5, "2004-2005", 100, 100)
cov.plot.04.05
```

<img src="FluVacs_Figures_files/figure-markdown_github/coverage-1.png" style="display: block; margin: auto;" />

``` r
cov.plot.05.06 <- cov_plot(cov.2005.6, "2005-2006", 100, 100)
cov.plot.05.06
```

<img src="FluVacs_Figures_files/figure-markdown_github/coverage-2.png" style="display: block; margin: auto;" />

``` r
cov.plot.07.08 <- cov_plot(cov.2007.8, "2007-2008", 100, 100)
cov.plot.07.08
```

<img src="FluVacs_Figures_files/figure-markdown_github/coverage-3.png" style="display: block; margin: auto;" />

If we plot on a log scale the bars are well above 0.

Initial data processing
=======================

So after all that work we are left with all the samples (all seasons) in all.df and only those variants from high quality data in all.qual.df. This includes samples with &gt;10<sup>5</sup> genomes per microliter or &gt; 10<sup>3</sup> genomes per microliter but were sequenced in duplicate.

Frequency of Variants
=====================

``` r
ggplot(subset(all.df, freq.var <= 0.99), aes(x = freq.var)) + geom_histogram(color = "white", 
    binwidth = 0.01) + scale_y_log10() + ggtitle("All samples")
```

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-2-1.png" style="display: block; margin: auto;" />

``` r
ggplot(subset(all.qual.df, freq.var <= 0.99), aes(x = freq.var)) + geom_histogram(color = "white", 
    binwidth = 0.01) + scale_y_log10() + ggtitle("Only high quality samples")
```

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-2-2.png" style="display: block; margin: auto;" />

Diversity measures
==================

SNV count per sample
--------------------

``` r
count_muts <- function(data.df) {
    ddply(data.df, ~Lauring_Id + Vax + season, function(x) dim(x)[1])
}

all.snv <- count_muts(all.df)
qual.snv <- count_muts(all.qual.df)
```

### Whole genome

#### all data

``` r
ggplot(data = subset(all.snv, !is.na(Vax)), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Number of SNV")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

#### High quality

``` r
ggplot(data = subset(qual.snv, !is.na(Vax)), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y", dotsize = 0.7) + ylab("Number of SNV")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

### HA NA

#### all data

``` r
all.snv <- count_muts(subset(all.df, chr %in% c("HA", "N_A", "NR")))
ggplot(data = subset(all.snv, !is.na(Vax)), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Number of SNV")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

#### High quality

``` r
qual.snv <- count_muts(subset(all.qual.df, chr %in% c("HA", "N_A", "NR")))

ggplot(data = subset(qual.snv, !is.na(Vax)), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Number of SNV")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-6-1.png" style="display: block; margin: auto;" />

### Anitigenic sites

``` r
Ha_antigenic <- function(df) {
    # non_coding<-29
    
    df <- mutate(df, AA_pos = (pos - 1)%/%3 + 1)
    
    
    df$antigenic <- F
    antigenic_sites <- c(122, 124, 126, 130, 131, 132, 133, 135, 137, 138, 140, 
        142, 143, 144, 145, 146, 150, 152, 168, 128, 129, 155, 156, 157, 158, 
        159, 160, 163, 164, 165, 186, 187, 188, 189, 190, 192, 193, 194, 196, 
        197, 198, 44, 45, 46, 47, 48, 50, 51, 53, 54, 273, 275, 276, 278, 279, 
        280, 294, 297, 299, 300, 304, 305, 307, 308, 309, 310, 311, 312, 96, 
        102, 103, 117, 121, 167, 170, 171, 172, 173, 174, 175, 176, 177, 179, 
        182, 201, 203, 207, 208, 209, 212, 213, 214, 215, 216, 217, 218, 219, 
        226, 227, 228, 229, 230, 238, 240, 242, 246, 247, 248, 57, 59, 62, 63, 
        67, 75, 78, 80, 81, 82, 83, 86, 87, 88, 91, 92, 94, 109, 260, 261, 262, 
        265)
    AA_anti <- antigenic_sites + 17
    # antigenic sites: Site A:
    # 122,124,126,130,131,132,133,135,137,138,140,142,143,144,145,146,150,152,168
    # Site B:
    # 128,129,155,156,157,158,159,160,163,164,165,186,187,188,189,190,192,193,194,196,197,198
    # Site C:
    # 44,45,46,47,48,50,51,53,54,273,275,276,278,279,280,294,297,299,300,304,305,307,308,309,310,311,312
    # Site D:
    # 96,102,103,117,121,167,170,171,172,173,174,175,176,177,179,182,201,203,207,208,209,212,213,214,215,216,217,218,219,226,227,228,229,230,238,240,242,246,247,248
    # Site E:
    # 57,59,62,63,67,75,78,80,81,82,83,86,87,88,91,92,94,109,260,261,262,265 All
    # sites:
    # 122,124,126,130,131,132,133,135,137,138,140,142,143,144,145,146,150,152,168,128,129,155,156,157,158,159,160,163,164,165,186,187,188,189,190,192,193,194,196,197,198,44,45,46,47,48,50,51,53,54,273,275,276,278,279,280,294,297,299,300,304,305,307,308,309,310,311,312,96,102,103,117,121,167,170,171,172,173,174,175,176,177,179,182,201,203,207,208,209,212,213,214,215,216,217,218,219,226,227,228,229,230,238,240,242,246,247,248,57,59,62,63,67,75,78,80,81,82,83,86,87,88,91,92,94,109,260,261,262,265
    # Cluster Transitions: 145,155,156,158,159,189,193
    
    
    
    df$antigenic[df$chr == "HA" & df$AA_pos %in% AA_anti] <- T
    return(df)
}


all.df <- Ha_antigenic(all.df)

all.df.counts <- count_muts(subset(all.df, antigenic == T))

ggplot(data = subset(all.df.counts, !is.na(Vax)), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y", dotsize = 0.8) + ylab("Number of SNV") + ggtitle("all samples")

all.qual.df <- Ha_antigenic(all.qual.df)

all.qual.df.counts <- count_muts(subset(all.qual.df, antigenic == T))
ggplot(data = subset(all.qual.df.counts, !is.na(Vax)), aes(y = V1, x = Vax)) + 
    geom_dotplot(stackdir = "center", binaxis = "y", dotsize = 0.8) + ylab("Number of SNV") + 
    ggtitle("quality samples")

all.df.counts <- count_muts(subset(all.df, antigenic == T & freq.var < 0.9))

ggplot(data = subset(all.df.counts), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Number of SNV") + ggtitle("all samples frequency<90%")



all.qual.df.counts <- count_muts(subset(all.qual.df, antigenic == T & freq.var < 
    0.9))
ggplot(data = subset(all.qual.df.counts), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Number of SNV") + ggtitle("quality samples (frequency<90%)")
```

Shannon's Entropy
-----------------

For the entropy calculations I am assuming 39000 potential variants. This is not right but probably close enough. It is just used to normalize the data. In the future we can use the fasta reference files to get the exact number.

### Whole genome

``` r
possible_vars <- 3900  # fix to be accurate

H <- function(x) {
    x <- subset(x, freq.var < 0.9)
    H_pos <- ddply(x, ~chr + pos, summarize, wt = 1 - sum(freq.var), H = -sum(c(freq.var * 
        log(freq.var), wt * log(wt))))
    sum(H_pos$H)/(possible_vars/3)
}

all.H <- ddply(all.df, ~Lauring_Id + Vax + season, H)

ggplot(data = subset(all.H), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Shannon's Entropy") + scale_y_log10() + ggtitle("all samples")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-8-1.png" style="display: block; margin: auto;" />

``` r
qual.H <- ddply(all.qual.df, ~Lauring_Id + Vax + season, H)
ggplot(data = subset(qual.H), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Shannon's Entropy") + scale_y_log10() + ggtitle("quality samples")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-8-2.png" style="display: block; margin: auto;" />

### HA NA

``` r
all.HA.NA.H <- ddply(subset(all.df, chr %in% c("HA", "N_A", "NR")), ~Lauring_Id + 
    Vax + season, H)

ggplot(data = subset(all.HA.NA.H), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Shannon's Entropy") + scale_y_log10() + ggtitle("all samples")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

``` r
qual.HA.NA.H <- ddply(subset(all.qual.df, chr %in% c("HA", "N_A", "NR")), ~Lauring_Id + 
    Vax + season, H)
ggplot(data = subset(qual.H), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Shannon's Entropy") + scale_y_log10() + ggtitle("quality samples")
```

    ## `stat_bindot()` using `bins = 30`. Pick better value with `binwidth`.

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-9-2.png" style="display: block; margin: auto;" />

### Anitigenic sites

``` r
all.ant.H <- ddply(subset(all.df, antigenic == T), ~Lauring_Id + Vax + season, 
    H)

ggplot(data = subset(all.ant.H), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Shannon's Entropy") + scale_y_log10() + ggtitle("all samples")




qual.ant.H <- ddply(subset(all.qual.df, antigenic == T), ~Lauring_Id + Vax + 
    season, H)

ggplot(data = subset(qual.ant.H), aes(y = V1, x = Vax)) + geom_dotplot(stackdir = "center", 
    binaxis = "y") + ylab("Shannon's Entropy") + scale_y_log10() + ggtitle("quality samples")
```

Heat map
========

Theses are made with the quality samples

``` r
make_heat_map(subset(all.qual.df, season == "07-08"))
```

    ## Aggregation function missing: defaulting to length

    ##    mutation      variable value segment
    ## 1 PB2_A182G 07-08.79.TRUE    -1     PB2
    ## 2 PB2_G186A 07-08.79.TRUE    -1     PB2
    ## 3 PB2_T201C 07-08.79.TRUE    -1     PB2
    ## 4 PB2_G204A 07-08.79.TRUE    -1     PB2
    ## 5 PB2_G209A 07-08.79.TRUE    -1     PB2
    ## 6 PB2_G220A 07-08.79.TRUE    -1     PB2

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

``` r
make_heat_map(subset(all.qual.df, season == "05-06"))
```

    ## Aggregation function missing: defaulting to length

    ##    mutation       variable value segment
    ## 1  PB2_G96A 05-06.509.TRUE    -1     PB2
    ## 2 PB2_C191T 05-06.509.TRUE    -1     PB2
    ## 3 PB2_A225G 05-06.509.TRUE    -1     PB2
    ## 4 PB2_G236A 05-06.509.TRUE    -1     PB2
    ## 5 PB2_A264G 05-06.509.TRUE    -1     PB2
    ## 6 PB2_G282A 05-06.509.TRUE    -1     PB2

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-11-2.png" style="display: block; margin: auto;" />

``` r
make_heat_map(subset(all.qual.df, season == "04-05"))
```

    ##    mutation       variable     value segment
    ## 1  PB2_G96A 04-05.402.TRUE -1.000000     PB2
    ## 2 PB2_T249C 04-05.402.TRUE -1.000000     PB2
    ## 3 PB2_A316G 04-05.402.TRUE  1.998882     PB2
    ## 4 PB2_C462T 04-05.402.TRUE -1.000000     PB2
    ## 5 PB2_C465T 04-05.402.TRUE  1.999526     PB2
    ## 6 PB2_A486G 04-05.402.TRUE -1.000000     PB2

<img src="FluVacs_Figures_files/figure-markdown_github/unnamed-chunk-11-3.png" style="display: block; margin: auto;" />

Antigenic Sites
===============

### All data

``` r
# kable(subset(all.df,antigenic==T & season=='07-08',
# select=c(Lauring_Id,mutation,freq.var,AA_Pos)))
```

### High quality

``` r
x <- Ha_antigenic(qual.2007.8)

kable(subset(x, antigenic == T & season == "07-08", select = c(Lauring_Id, mutation, 
    freq.var, AA_pos)))
```