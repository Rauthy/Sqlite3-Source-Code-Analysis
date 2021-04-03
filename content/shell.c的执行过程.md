#### shell.c 文件代码分析

- ##### shell.c文件的生成

  shell.c文件由tool文件夹中的mkshellc.tcl生成

- ##### shell.c的执行流程

  - 20305   main函数入口

    setBinaryMode(stdin,0)

    setvbuf(stderr,0,_IONBF,0)   //不使用缓冲。每个 I/O 操作都被即时写入

    stdin_is_interactive = isatty(0)   //int isatty(int desc); 如果参数 desc 所代表的文件描述词为一终端机则返回1, 否则返回0

    stdout_is_console = isatty(1) 

    针对win和linux下的IO进行设置。

    源码中的预处理指令很多，运行如下命令可以得到预处理后的源文件shell.i

    ```shell
    gcc -E shell.c [-C] -o shell.i
    ```

    

    main_init(&data) [shell.c  line:20243]   初始化shell状态

    代码如下：

    ```c
  static void main_init(ShellState *data) {
      memset(data, 0, sizeof(*data));
      data->normalMode = data->cMode = data->mode = MODE_List;
      data->autoExplain = 1;
      memcpy(data->colSeparator,SEP_Column, 2);
      memcpy(data->rowSeparator,SEP_Row, 2);
      data->showHeader = 0;
      data->shellFlgs = SHFLG_Lookaside;
      verify_uninitialized();
      sqlite3_config(SQLITE_CONFIG_URI, 1);
      sqlite3_config(SQLITE_CONFIG_LOG, shellLog, data);
      sqlite3_config(SQLITE_CONFIG_MULTITHREAD);
      sqlite3_snprintf(sizeof(mainPrompt), mainPrompt,"sqlite> ");
      sqlite3_snprintf(sizeof(continuePrompt), continuePrompt,"   ...> ");
    }
    // mainPrimpt和continuePrompt定义如下：
    static char mainPrompt[20];     /* First line prompt. default: "sqlite> "*/ # 454
    static char continuePrompt[20]; /* Continuation prompt. default: "   ...> " */  #455 
    ```
    
    sqlite3_config[sqlite3.c  line: 161432 ]和sqlite3_snprintf[sqlite3.c line: 29369]都是sqlite的API函数

    此处data为ShellState变量，其定义如下：

    ```C
  struct ShellState {
      sqlite3 *db; //数据库指针
      u8 autoExplain;
      u8 autoEQP;
      u8 autoEQPtest;
      u8 autoEQPtrace;
      u8 statsOn;
      u8 scanstatsOn;
      u8 openMode;
      u8 doXdgOpen;
      u8 nEqpLevel;
      u8 eTraceType;
      unsigned mEqpLines;
      int outCount;
      int cnt;
      int lineno;
      int openFlags;
      FILE *in;     //输入流,从此处读入命令
      FILE *out;    //输出流，从此处输出结果
      FILE *traceOut;    //Output for sqlite3_trace();
      int nErr;
      int mode;     //输出模式设置
      int modePrior;  
      int cMode;
      int normalMode;
      int writableSchema;
      int showHeader;
      int nCheck;
      unsigned nProgress;
      unsigned mxProgress;
      unsigned flgProgress;
      unsigned shellFlgs;
      unsigned priorShFlgs;
      sqlite3_int64 szMax;
      char *zDestTable;
      char *zTempFile;
      char zTestcase[30];
      char colSeparator[20];
      char rowSeparator[20];
      char colSepPrior[20];
      char rowSepPrior[20];
      int *colWidth;
      int *actualWidth;
      int nWidth;
      char nullValue[20];
      char outfile[4096];
      const char *zDbFilename;   //数据库文件名
      char *zFreeOnClose;
      const char *zVfs;          //vfs名称
      sqlite3_stmt *pStmt;       //当前查询语句
      FILE *pLog;                //日志输出
      int *aiIndent;
      int nIndent;
      int iIndent;
      EQPGraph sGraph;
      ExpertInfo expert;
    };
    ```
    
    初始化完成后会读取命令行参数，确定数据库文件名。随后执行sqlite3_initialize()和open_db();

  - 20547   sqlite3_initilize();

  - 20578   open_db(&data, 0);

  - 20585   process_sqliterc(&data,zInitFile);

    zInitFile为运行参数中指定的文件名

  - **second pass through the command-line argument**

    如果命令行参数是交互式的，则调用process_input；否则执行完命令行里的命令后直接退出main函数。

  - 20808   rc = process_input(&data); main函数里头调用process_input函数处理用户输入的命令

  - 19961   static int process_input(ShellState *p);

    ```c++
    static int process_input(ShellState *p){
      char *zLine = 0;          /* A single input line */
      char *zSql = 0;           /* Accumulated SQL text */
      int nLine;                /* Length of current line */
      int nSql = 0;             /* Bytes of zSql[] used */
      int nAlloc = 0;           /* Allocated zSql[] space */
      int nSqlPrior = 0;        /* Bytes of zSql[] used by prior line */
      int rc;                   /* Error code */
      int errCnt = 0;           /* Number of errors seen */
      int startline = 0;        /* Line number for start of current input */
      p->lineno = 0;
      while( errCnt==0 || !bail_on_error || (p->in==0 && stdin_is_interactive) ){
    	...
      }
    ...
    ```

    该函数内部主要是一个while循环，当发生错误或命令行参数指定为非交互模式时会退出该循环。

    该循环的逻辑为如下：

    1. 使用one_input_line函数获取输入，one_input_line底层采用的是get_line函数获取输入，这个在利用gdb调试函数调用栈的时候可以看到

       ```c++
       zLine = one_input_line(p->in, zLine, nSql>0);    #19975
       ```

    2. 判断输入的指令类型

       ```c++
       if( zLine && (zLine[0]=='.' || zLine[0]=='#') && nSql==0 ){
             if( ShellHasFlag(p, SHFLG_Echo) ) printf("%s\n", zLine);
             if( zLine[0]=='.' ){
               rc = do_meta_command(zLine, p);
               //Return 1 on error, 2 to exit, and 0 otherwise
               if( rc==2 ){ /* exit requested */
                 break;
               }else if( rc ){
                 errCnt++;
               }
             }
             continue;
       }
       ```

       ​    如果指令以 . 开头，则调用do_meta_command(#16944)函数处理。

       ​    还有一个对指令长度的判断

       ```c++
       /*
       ** Compute a string length that is limited to what can be stored in
       ** lower 30 bits of a 32-bit signed integer.
       */
       static int strlen30(const char *z){
         const char *z2 = z;
         while( *z2 ){ z2++; }
         return 0x3fffffff & (int)(z2 - z);
       }    
       -----------------------------------------------------------------------------------
           nLine = strlen30(zLine);
           
           if( nSql+nLine+2>=nAlloc ){
             nAlloc = nSql+nLine+100;
             zSql = realloc(zSql, nAlloc);
             if( zSql==0 ) shell_out_of_memory();
           }
       ```

       ​    大概就是判断输入的sql指令长度是否能用32位有符号整数的低30位表示。

    3. 执行输入的sql语句

       

    - 19975   one_input_line(p->in, zLine, nSql>0);
    - 727       static char *one_input_line(FILE *in, char *zPrior, int isContinuation)
    - 668       static char *local_getline(char *zLine, FILE *in)
    - 678       调用fget()函数

  



