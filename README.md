# utl-count-distinct-values-in-each-row-of-a-table-not-so-easy
Count distinct values in each row of a table
     Count distinct values in each row of a table.

    github
    https://tinyurl.com/y9um3tu2
    https://github.com/rogerjdeangelis/utl-count-distinct-values-in-each-row-of-a-table-not-so-easy

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    SAS tends to be column oriented not row oriented.
    The hash object helps cross the void between base SAS and a matrix language.
    The hash is in memory.

    inspired by
    https://tinyurl.com/y9xmnhh5
    https://communities.sas.com/t5/SAS-Programming/counting-distinct-values-over-a-range-of-variables/td-p/327817/page/2

       Four Solutions
             a. datastep
             b. Bart's HASH (preferred solution - very elegant)
                Bartosz Jablonski
                yabwon@gmail.com
             c. faster datastep
             d. R (should be just as elegant in IML)
             e. SAS without sort
                Yinglin (Max) Wu
                yinglinwu@gmail.com
    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;
    data sd1.have;
     input id$ v1-v5;
    cards4;
    A 3 3 2 3 3
    B 3 3 2 3 3
    C 1 3 2 3 3
    D 1 3 2 3 3
    E 1 3 2 3 3
    ;;;;
    run;quit;

                                          | RULES
      SD1.HAVE total obs=5                |
                                          | COUNT
       ID    V1    V2    V3    V4    V5   | Unique
                                          |
       A      3     3     2     3     3   | 2 ( 2s,3s)
       B      3     3     2     3     3   | 2 ( 2s,3s)
       C      1     3     2     3     3   | 3 ( 1s,2s,3s)
       D      1     3     2     3     3   | 3 ( 1s,2s,3s)
       E      1     3     2     3     3   | 3 ( 1s,2s,3s)

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    WORK.WANT total obs=5

              CNT_
       ID    UNIQUE

       A        2
       B        2
       C        3
       D        3
       E        3

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/
                   _       _            _
      __ _      __| | __ _| |_ __ _ ___| |_ ___ _ __
     / _` |    / _` |/ _` | __/ _` / __| __/ _ \ '_ \
    | (_| |_  | (_| | (_| | || (_| \__ \ ||  __/ |_) |
     \__,_(_)  \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                               |_|
    ;

    data want;
      set sd1.have;
      array vs[*] v:;
      call sortn(of v:);
      count_unique=1;
      do i=2 to dim(vs);
        if vs[i] ne vs[i-1] then count_unique=count_unique +1;
      end;
      output;
      count_unique=0;
      keep id ciunt_unique;
    run;quit;

    *_        ____             _         _               _
    | |__    | __ )  __ _ _ __| |_ ___  | |__   __ _ ___| |__
    | '_ \   |  _ \ / _` | '__| __/ __| | '_ \ / _` / __| '_ \
    | |_) |  | |_) | (_| | |  | |_\__ \ | | | | (_| \__ \ | | |
    |_.__(_) |____/ \__,_|_|   \__|___/ |_| |_|\__,_|___/_| |_|

    ;

    data have;
     input id$ v1-v5;
    cards4;
    A 3 3 2 3 3
    B 3 3 2 3 3
    C 1 3 2 3 3
    D 1 3 2 3 3
    E 1 3 2 3 3
    ;;;;
    run;

    data want;

        declare hash h ();
        h.defineKey ('_');
        h.defineDone();
        call missing(_);

        do until (lr);
            set have end=lr;
            array vs v:;
            do over vs;
               _N_ = h.add(key: vs, data: vs); /* if you remove `, data: vs` you will get
                                                  an error which proves that even if you are
                                                  declaring "only a key-hash" the data portion
                                                  (equal to the key) is also defined :-) */
            end;
            cnt_unique = h.num_items;
            output;
            _N_ = h.clear();
        end;

        keep id cnt_unique;
        stop;
    run;

    *          __           _         _       _            _
      ___     / _| __ _ ___| |_    __| | __ _| |_ __ _ ___| |_ ___ _ __
     / __|   | |_ / _` / __| __|  / _` |/ _` | __/ _` / __| __/ _ \ '_ \
    | (__ _  |  _| (_| \__ \ |_  | (_| | (_| | || (_| \__ \ ||  __/ |_) |
     \___(_) |_|  \__,_|___/\__|  \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                                                  |_|
    ;

    %array(vpre,values=v1-v4);
    %array(vcur,values=v2-v5);

    data want;
      set sd1.have;
      call sortn(of v:);
      cnt_unique=1;
      %do_over(vcur vpre,phrase=%str(
        if ?vcur ne ?vpre then cnt_unique = cnt_unique + 1;)
      );
      output;
      cnt_unique=0;
      keep id cnt_unique;
    run;quit;


    data _null_;
      putlog
      %do_over(vcur vpre,phrase=%str(
        "if ?vcur ne ?vpre then cnt_unique = cnt_unique + 1" /)
      );
    run;quit;
    
    Generated Code
    
    if v2 ne v1 then cnt_unique = cnt_unique + 1
    if v3 ne v2 then cnt_unique = cnt_unique + 1
    if v4 ne v3 then cnt_unique = cnt_unique + 1
    if v5 ne v4 then cnt_unique = cnt_unique + 1

    *    _
      __| |    _ __
     / _` |   | '__|
    | (_| |_  | |
     \__,_(_) |_|

    ;


    * most of the code is just to get data in and out of R;

    %utl_submit_r64('
    library(SASxport);
    library(haven);
    have<-read_sas("d:/sd1/have.sas7bdat");
    want<-apply(have[,2:6], 1, function(x) length(unique(x)));
    want<-cbind(have[,1],want);
    write.xport(want,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";
    data want;
      set xpt.want;
    run;quit;
    libname xpt clear;

