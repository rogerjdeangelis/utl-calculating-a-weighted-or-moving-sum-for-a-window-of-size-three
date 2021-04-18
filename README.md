# utl-calculating-a-weighted-or-moving-sum-for-a-window-of-size-three
Calculating a weighted or moving sum for a window of size three
    Calculating a weighted or moving sum for a window of size three

    GitHub
    https://tinyurl.com/4r7ypbah
    https://github.com/rogerjdeangelis/utl-calculating-a-weighted-or-moving-sum-for-a-window-of-size-three


    Problem

       I want to apply these weight(WGT) to a rolling sum of(VAL) for window=3

       (0.1,0.3,0.6)

    RULES
                              | What I want
               INPUT          |
        ===================== | RULES(roling sum of weighted vals wndow=3)
        VAL    WGT    VAL*WGT |
                              |
         1     0.1      0.1   |  .
         1     0.3      0.3   |  .
         1     0.6      0.6   | (.1 + .3 + .6 )   = 1
         2     0.1      0.2   | (.3 + .6 + .2 )   = 1.1
         2     0.3      0.6   | (.6 + .2 + .6 )   = 1.4
         2     0.6      1.2   | (.2 + .6 + 1.2 )  = 2.0


    This solution uses the Dulles Reasearc S-JDBC package to create SAS datasets from R.

    https://dullesresearch.com/
    There are two prices for S-JDBC ($295 and $195) on the site)

    Argue with them for the $195 price using this link on the site.
    https://dullesresearch.com/wp-content/uploads/2017/10/Carolina-S-JDBC-Product-Sheet.pdf
    I was able to get a cupon for the $195 price.

    The full function evaluation version seems to work beyond 15 days?

    I always look for a package that

    Other moving or rolling function

    https://github.com/rogerjdeangelis?tab=repositories&q=rolling&type=&language=&sort=
    https://github.com/rogerjdeangelis?tab=repositories&q=moving&type=&language=&sort=

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    options validvarname=upcase;

    libname sd1 "d:/sd1";
    data sd1.have;
      array wgtvec[3] _temporary_ (0.1 0.3 0.6);
      input val;
      idx=mod(_n_-1,3) + 1;
      wgt=wgtvec[idx];
      valxwgt=val*wgt;
      drop idx;
    cards4;
    1
    1
    1
    2
    2
    2
    ;;;;
    run;quit;


    Up to 40 obs SD1.HAVE total obs=6

    Obs    VAL    WGT    VALXWGT

     1      1     0.1      0.1
     2      1     0.3      0.3
     3      1     0.6      0.6
     4      2     0.1      0.2
     5      2     0.3      0.6
     6      2     0.6      1.2

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;


    * just in case roy rerun;
    proc datasets lib=sd1 nolist;
      delete want;
    run;quit;

    %utl_submit_r64('
    library(RJDBC);
    library(haven);
    library(data.table);
    library(FRAPO);
    want<-read_sas("d:/sd1/have.sas7bdat");
    have<-as.data.table(read_sas("d:/sd1/have.sas7bdat"))[,1];
    have;
    want <- trdwma(have, weights = c(0.1, 0.3, 0.6));
    want[is.na(want)]<-NaN;
    drv<- JDBC("com.dullesopen.jdbc.Driver","d:/carolina/carolina-jdbc-2.4.3.jar");
    conn <- dbConnect(drv, "jdbc:carolina:bulk:libnames=(dir=''d:/sd1'')", "", "");
    rc<- dbWriteTable(conn,"want",want);
    ');

    * there appears to be an issue;
    data addvar;
      merge sd1.have sd1.want(rename=val=weighted_series);
    run;quit;

    proc print data=addvar;
    run;quit;

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;


    Up to 40 obs from ADDVAR total obs=6

                                    WEIGHTED_
    Obs    VAL    WGT    VALXWGT      SERIES

     1      1     0.1      0.1          .
     2      1     0.3      0.3          .
     3      1     0.6      0.6         1.0
     4      2     0.1      0.2         1.1
     5      2     0.3      0.6         1.4
     6      2     0.6      1.2         2.0
