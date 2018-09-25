# utl-how-to-call-R-repeatedly-in-a-loop-and-retrieve-the-results-for-further-processing-in-SAS
How to call R repeatedly in a loop and retrieve the results for further processing in SAS. 

    How to call R repeatedly in a loop and retrieve the results for further processing in SAS

    You can wrap in a macro instead of DOSUBL.

    PROBLEM: Draw three random samples by calling R three time;

       1. Draw a random sample of size 5 from a population of 1-10
       2. Draw a random sample of size 5 from a population of 1-20
       3. Draw a random sample of size 5 from a population of 1-30


    see
    https://tinyurl.com/yafq6egk
    https://stackoverflow.com/questions/52338150/how-to-call-r-repeatedly-in-a-loop-and-retrieve-the-results-for-further-processi


    INPUT
    =====

     Three population sizes (10,20,30)
     Fixed draw size of 5

     Random data generated in R


    EXAMPLE OUTPUTS
    ---------------

    WORK.LOG total obs=3

     POP    DRAW    RC                  STATUS

      10      5      0    Sample from R appended Successfully
      20      5      0    Sample from R appended Successfully
      30      5      0    Sample from R appended Successfully


    WORK.WANT total obs=15

    Obs    SAMPLE_1    POP    DRAW

      1        4        10      5   pop=10 draw=5
      2       10        10      5
      3        7        10      5
      4        8        10      5
      5        9        10      5

      6        6        20      5   pop=20 draw=5
      7       10        20      5
      8       16        20      5
      9       16        20      5
     10       17        20      5

     11       26        30      5   pop=30 draw=5
     12       13        30      5
     13        5        30      5
     14       21        30      5
     15        3        30      5


    PROCESS
    =======

    * just in case you rerun;
    %symdel pop draw / nowarn;
    %utlfkil(d:/xpt/rxpt.xpt);
    proc datasets lib=work kill;
    run;quit;

    data log;

      retain pop 0 draw 5;

      do pop=10 to 30 by 10;

         call symputx("pop",pop);
         call symputx("draw",draw);

         rc=dosubl('

          %utl_submit_r64(resolve(''
            library(SASxport);
            sample<-as.data.frame(sample(1:&pop, &draw, replace=TRUE));
            sample$pop<-&pop;
            sample$draw<-&draw;
            write.xport(sample,file="d:/xpt/rxpt.xpt");
         ''))

         libname xpt xport "d:/xpt/rxpt.xpt";
         proc append data=xpt.sample base=want;
         run;quit;
         %let cc=&syserr;
         ');

         if symgetn('cc') = 0 then status="Sample from R appended Successfully";
         else status="Apending Failed";
         output;

      end;

    run;quit;

    OUTPUT
    ======

    see EXAMPLE OUTPUT above

    LOG
    ===


    > library(SASxport);
    > sample<-as.data.frame(sample(1:30, 5, replace=TRUE));
    > sample$pop<-30;
    > sample$draw<-5;
    > write.xport(sample,file="d:/xpt/rxpt.xpt");
    >
    NOTE: 3 lines were written to file PRINT.
    Stderr output:
    package 'SASxport' was built under R version 3.3.3
    In makeSASNames(colnames(df)) : Truncated 1 long names to 8 characters.
    NOTE: 2 records were read from the infile RUT.
          The minimum record length was 2.
          The maximum record length was 178.
    NOTE: DATA statement used (Total process time):
          real time           3.08 seconds

    MPRINT(UTL_SUBMIT_R64):   filename rut clear;
    NOTE: Fileref RUT has been deassigned.
    MPRINT(UTL_SUBMIT_R64):   filename r_pgm clear;
    NOTE: Fileref R_PGM has been deassigned.
    MPRINT(UTL_SUBMIT_R64):   * use the clipboard to create macro variable;
    SYMBOLGEN:  Macro variable RETURNVAR resolves to N
    MLOGIC(UTL_SUBMIT_R64):  %IF condition %upcase(%substr(&returnVar.,1,1)) ne N is FALSE
    MLOGIC(UTL_SUBMIT_R64):  Ending execution.
    NOTE: Libref XPT was successfully assigned as follows:
          Engine:        XPORT
          Physical Name: d:\xpt\rxpt.xpt
    NOTE: Appending XPT.SAMPLE to WORK.ALLSAMPLES.
    NOTE: There were 5 observations read from the data set XPT.SAMPLE.
    NOTE: 5 observations added.
    NOTE: The data set WORK.ALLSAMPLES has 30 observations and 3 variables.
    NOTE: PROCEDURE APPEND used (Total process time):
          real time           0.03 seconds

    SYMBOLGEN:  Macro variable SYSERR resolves to 0
    NOTE: The data set WORK.LOG has 3 observations and 4 variables.
    NOTE: DATA statement used (Total process time):
          real time           10.33 seconds

    3683!     quit;


