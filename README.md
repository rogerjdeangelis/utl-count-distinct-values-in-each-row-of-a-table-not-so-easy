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

       Three Solutions
             a. hash
             b. datastep
             c. faster datastep
             d. R (should be just as elegant in IML)

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

    *          _               _
      __ _    | |__   __ _ ___| |__
     / _` |   | '_ \ / _` / __| '_ \
    | (_| |_  | | | | (_| \__ \ | | |
     \__,_(_) |_| |_|\__,_|___/_| |_|

    ;

    * I am a novice HASH programmer;

    data want;

        if _n_=0 then set sd1.have;

        declare hash h ();
        h.definekey ('id','m');
        h.definedata ('id', 'v1', 'v2', 'v3', 'v4', 'v5');
        h.definedone();

        do until (lr);
            retain cnt_unique 0;
            set sd1.have end=lr;
            array vs _numeric_;
            do i = 1 to 5;
               m = vs[i];
               if h.check() ne 0 then cnt_unique=cnt_unique+1;
               h.ref();
            end;
            output;
            cnt_unique=0;
        end;

        keep id cnt_unique;

    run;


    *_            _       _            _
    | |__      __| | __ _| |_ __ _ ___| |_ ___ _ __
    | '_ \    / _` |/ _` | __/ _` / __| __/ _ \ '_ \
    | |_) |  | (_| | (_| | || (_| \__ \ ||  __/ |_) |
    |_.__(_)  \__,_|\__,_|\__\__,_|___/\__\___| .__/
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




