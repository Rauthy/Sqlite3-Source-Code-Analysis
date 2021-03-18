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
  
    我是在linux平台上编译的，某些针对win的操作我就直接略过了。
  
    
  
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
  - 14123   open_db(&data, 0);
  
  - 20585   process_sqliterc(&data,zInitFile);
  
    zInitFile为运行参数中指定的文件名
  
  - **second pass through the command-line argument**
  
  - 
  
  - 20808   rc = process_input(&data);
  - 19961   static int process_input(ShellState *p);
  - 19975   one_input_line(p->in, zLine, nSql>0);
  - 727       static char *one_input_line(FILE *in, char *zPrior, int isContinuation)
  - 668       static char *local_getline(char *zLine, FILE *in)
  - 678       调用fget()函数
  
  



