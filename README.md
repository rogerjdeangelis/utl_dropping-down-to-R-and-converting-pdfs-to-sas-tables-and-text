# utl_dropping-down-to-R-and-converting-pdfs-to-sas-tables-and-text
Dropping down to R and converting pdfs to sas tables and text 
    %let pgm=utl_dropping-down-to-R-and-converting-pdfs-to-sas-tables-and-text;

    Dropping down to R and converting pdfs to sas tables and text

    github
    https://tinyurl.com/3pcvjms9
    https://github.com/rogerjdeangelis/utl_dropping-down-to-R-and-converting-pdfs-to-sas-tables-and-text


    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    You need folders
      d:/pdf
      d:/txt
      d:/sd1
    */

    %let _inpPdf=d:/pdf/class.pdf;

    ods escapechar="^";
    ods pdf file="d:/pdf/class.pdf" style=minimal ;
    proc report data=sashelp.class(obs=5);
      title1 " Title 1";
      title2 " Title 2";
      footnote1 "[1] Title 1    This is a test of footnote1";
      footnote2 "[2] Title 2    This is a test of footnote2";
    cols ("This is spanning ^{NEWLINE}" _all_ );
    define sex / display width=5;
    run;quit;
    ods pdf text="^{NEWLINE}^S={font_size=8pt just=l} Bottom of boxed report";
    run;quit;
    ods pdf close;
    title;
    footnote;

    /********************************************************/
    /*                                                      */
    /*  d:/pdf/class.pdf                                    */
    /*                                                      */
    /*                                                      */
    /*  Title 1                                             */
    /*  Title 2                                             */
    /*                                                      */
    /*             This is spanning                         */
    /*                                                      */
    /*   NAME      SEX          AGE     HEIGHT     WEIGHT   */
    /*   Alfred    M             14         69      112.5   */
    /*   Alice     F             13       56.5         84   */
    /*   Barbara   F             13       65.3         98   */
    /*   Carol     F             14       62.8      102.5   */
    /*   Henry     M             14       63.5      102.5   */
    /*                                                      */
    /*                                                      */
    /*    Bottom of boxed report                            */
    /*                                                      */
    /*                                                      */
    /*                                                      */
    /*    [1] Title 1    This is a test of footnote1        */
    /*    [2] Title 2    This is a test of footnote2        */
    /*                                                      */
    /*                                                      */
    /********************************************************/
    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /********************************************************/
    /*                                                      */
    /*   d:/txt/class.txt (has annotations)                 */
    /*                                                      */
    /*   1        Title 1                                   */
    /*   2        Title 2                                   */
    /*   3        This is spanning                          */
    /*   5        NAME SEX AGE HEIGHT WEIGHT                */
    /*   6        Alfred    M     14        69    112.5     */
    /*   7        Alice     F     13       56.5     84      */
    /*   8        Barbara F       13       65.3     98      */
    /*   9        Carol     F     14       62.8   102.5     */
    /*   10       Henry     M     14       63.5   102.5     */
    /*   11       Bottom of boxed report                    */
    /*   13       [1] Title 1 This is a test of footnote1   */
    /*   18       [2] Title 2 This is a test of footnote2   */
    /*                                                      */
    /*                                                      */
    /********************************************************/

    /********************************************************/
    /*                                                      */
    /*   d:/sd1/class.sas7bdat                              */
    /*                                                      */
    /*   TABLE total obs=5 22MAY2022:13:33:12               */
    /*                                                      */
    /*     NAME      SEX    AGE    HEIGHT    WEIGHT         */
    /*                                                      */
    /*    Alfred      M     14      69       112.5          */
    /*    Alice       F     13      56.5     84             */
    /*    Barbara     F     13      65.3     98             */
    /*    Carol       F     14      62.8     102.5          */
    /*    Henry       M     14      63.5     102.5          */
    /*                                                      */
    /********************************************************/

    /***********************************************************************************/
    /*                                                                                 */
    /*  LOG dataset                                                                    */
    /*                                                                                 */
    /*  LOG total obs=1 22MAY2022:14:02:35                                             */
    /*                                                                                 */
    /*   REC       FRO                 INPUT                        STATUS             */
    /*                                                                                 */
    /*    5  d:/pdf/class.pdf  NAME SEX AGE HEIGHT WEIGHT Table imported successfully  */
    /*                                                                                 */
    /***********************************************************************************/

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    /* you can wrap thi in a macro */

    libname sd1 "d:/sd1";

    %let _inpPdf=d:/pdf/class;

    %let _outTxt=d:/txt/class;
    %let _outSd1=d:/sd1/class;

    %utlfkil(d:/txt/class.txt);

    /*- convert pdf to text with annotations --*/
    %utl_submit_r64("
    library('tm');
    library('pdftools');
    file <- '&_inpPdf..pdf';
    Rpdf <- readPDF(control = list(text = '-layout'));
    corpus <- VCorpus(URISource(file),
          readerControl = list(reader = Rpdf));
    want <- content(content(corpus)[[1]]);
    want;
    write(want,file='&_outTxt..txt');
    ");


    /*-- clean up text remove lines without information --*/
    data _temp(compress=char);

      retain rec 0;

      length fro $24 lyn $200;

      infile "&_outTxt..txt";
      retain rec 0 fro "&_inpPdf..pdf" rec;

      input;

       _infile_=compress(_infile_,'0A'x);
       _infile_=compress(_infile_,'0C'x);
       _infile_=compress(_infile_,'0D'x);
       _infile_=compress(_infile_,'00'x);
       _infile_=compress(_infile_,'09'x);
       _infile_=compress(_infile_,'—');

      if length(strip(_infile_)) > 3;

      file "&_outTxt";

      rec+1;

      lyn=left(_infile_);

      put    rec @10  lyn;
      putlog rec @10  lyn;

      rec=_n_;

    run;quit;

    /*-- create final sas table table.sas7bdat --*/
    data log ;

      set _temp(where=(rec=5));

      lyn=compbl(lyn);

      call symputx('_inpCmd',lyn);
      putlog lyn;

      rc=dosubl('
          /*   %let _inpCmd=NAME SEX AGE HEIGHT WEIGHT;  */
          data table;
             informat &_inpCmd $8.;
             infile "d:/txt/class.txt" firstobs=6 obs=10;
             input &_inpCmd;
          run;quit;
          %let cc=&syserr;
          ');

      if  symgetn('cc') = 0 then status="Table imported successfully";
      else status="Table imported failed";
      drop rc;
    run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

