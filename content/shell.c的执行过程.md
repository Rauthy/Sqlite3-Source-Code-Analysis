#### shell.c 文件代码分析

- ##### shell.c文件的生成

  shell.c文件由tool文件夹中的mkshellc.tcl生成

- ##### shell.c的执行流程

  - 20305   main函数入口
  - 20808   rc = process_input(&data);
  - 19961   static int process_input(ShellState *p);
  - 19975   one_input_line(p->in, zLine, nSql>0);
  - 727       static char *one_input_line(FILE *in, char *zPrior, int isContinuation)
  - 668       static char *local_getline(char *zLine, FILE *in)
  - 678       调用fget()函数

  



