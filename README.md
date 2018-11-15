# utl-selecting-the-earliest-login-year-in-an-unsorted-list
Selecting the earliest login year in an unsorted list.

    Selecting the earliest login year in an unsorted list

    github
    https://tinyurl.com/yb7frbvl
    https://github.com/rogerjdeangelis/utl-selecting-the-earliest-login-year-in-an-unsorted-list

      FOUR SOLUTIONS
      --------------

         1. Faster solutions that maintain the order moderate and big data

             Innovative solutions by (curobs is nice)

              Paul Dorfman
              sashole@bellsouth.net
              https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;139ef1de.1811b

           a. HASH (loads all satellite variables)
           b. HASH (faster does not load all satellite variables)


         2. These solutions change the order of the data

           a. SQL
           b. Sort and first dot


    INPUT
    =====

     WORK.HAVE total obs=6     | RULES
                               |
     STUDENT    YEAR    MINS  |
                               |
      CATHY     2016     27   |
      ROGER     2012     11   | * select this one - first Roger
      CATHY     2015     18   | * select this one - first Cathy
      BOBBY     2014     29   |
      CATHY     2017     31   |
      BOBBY     2013     18   | * select this one - first Bobby



    EXAMPLE OUTPUT
    --------------

     WORK.WANT_HashFast total obs=3

      STUDENT     YER     MINS

       ROGER     JAN12     11   ==> maintains order
       BOBBY     JAN13     18
       CATHY     JAN15     18


    PROCESS
    =======

    1. HASH (loads all satellite variables)
    ---------------------------------------

    data _null_ ;
      dcl hash h () ;
      h.definekey  ("STUDENT") ;
      h.definedata ("STUDENT", "MINS", "YER") ;
      h.definedone () ;
      do until (z) ;
        set have end = z ;
        _MINS = MINS ;
        _YER = YER ;
        if h.find() = 0 and _YER < YER then do ;
          MINS = _MINS ;
          YER = _YER ;
        end ;
        h.replace() ;
      end ;
      h.output (dataset:"want") ;
    run ;


    2. HASH (Fastest does not load all satellite variables)
    --------------------------------------------------------

    data want (drop = _:) ;
      dcl hash h () ;
      h.definekey  ("STUDENT") ;
      h.definedata ("YER", "R") ;
      h.definedone () ;
      do until (z) ;
        set have curobs = R end = z ;
        _N_  = R ;
        _YER = YER ;
        if h.find() = 0 and _YER < YER then do ;
          R = _N_ ;
          YER = _YER ;
        end ;
        h.replace() ;
      end ;
      dcl hiter i ("h") ;
      do while (i.next() = 0) ;
        set have point = R ;
        output ;
      end ;
    run ;


    3.  SQL
    -------

    proc sql;
      create
         table want_sql as
      select
         *
      from
         have
      group
         by student
      having
         yer = min(yer)
    ;quit;


    4. Sort and first dot
    ---------------------

    proc sort data=have out=havSrt noequals;
    by student yer;
    run;quit;

    data want_dat;
      set havSrt;
      by student;
      if first.student;
    run;quit;

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    data have ;
      input STUDENT$ YER$ MINS;
      cards ;
    CATHY  JAN16  27
    ROGER  JAN12  11
    CATHY  JAN15  18
    BOBBY  JAN14  29
    CATHY  JAN17  31
    BOBBY  JAN13  18
    run ; quit;


