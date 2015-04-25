# Lab 7
### David Kale

## Part 1
Modifying the information server such that F2 causes "Hello, world!" to be displayed.

### Solution
1. Add entry for new function in /usr/src/servers/is/dmp.c
2. Add the prototype for this function to /usr/src/servers/is/proto.h
3. Implement the function in dmp_kernel.c
4. "make && make install" in /usr/src/servers/is
5. Reboot

### Diffs
```
root@~# diff -r /usr/src /usr/src-orig/
diff -r /usr/src/servers/is/dmp.c /usr/src-orig/servers/is/dmp.c
20d19
<     { F2,   hello_dmp, "Say hello" },
```

```
diff -r /usr/src/servers/is/dmp_kernel.c /usr/src-orig/servers/is/dmp_kernel.c
60,65d59
< void hello_dmp()
< {
<     printf("Hello world!\n");
< }
<
<
```

```
diff -r /usr/src/servers/is/proto.h /usr/src-orig/servers/is/proto.h
7d6
< void hello_dmp();
```

### Screenshots
![Part 1 Output][part1_ss]

## Part 2
Modifying the information server and kernel such that pressing F9 causes a table
of messages to be displayed.

### Solution
1. Repeat steps from part 1 to add the new function, msgtab_dmp, including adding an implementation
  in /usr/src/kernel/dmp_kernel.c
2. Define the message table stucture in include/minix/param.h
3. #define a new getter function in inlude/minix/syslib.h for accessing the table
4. Add a case for the new getter in kernel/system/do_getinfo.c
5. Declare the table as a global variable in kernel/glo.h, and declare the variable in kernel/usermapped_data.c
6. Initialize the table to zero in kernel/main.c
6. Add code shown in diffs, updating the table when a message is sent, to kernel/proc.c

### Diffs
```
diff -r /usr/src/include/minix/com.h /usr/src-orig/include/minix/com.h
475d474
< #   define GET_MSGTAB 25
diff -r /usr/src/include/minix/param.h /usr/src-orig/include/minix/param.h
48,52d47
< #define MSGTAB_SIZE 10
< typedef struct msgtab {
<     int messages[MSGTAB_SIZE][MSGTAB_SIZE];
< } msgtab_t;
<
```

```
diff -r /usr/src/include/minix/syslib.h /usr/src-orig/include/minix/syslib.h
160d159
< #define sys_getmsgtab(dst) sys_getinfo(GET_MSGTAB, dst, 0, 0, 0)
```

```
diff -r /usr/src/kernel/glo.h /usr/src-orig/kernel/glo.h
22d21
< extern struct msgtab msgtab;
```

```
diff -r /usr/src/kernel/main.c /usr/src-orig/kernel/main.c
120,127d119
<     /* Initialize message table */
<     int mi, mj;
<     for (mi = 0; mi < MSGTAB_SIZE; mi++) {
<         for (mj = 0; mj < MSGTAB_SIZE; mj++) {
<             msgtab.messages[mi][mj] = 0;
<         }
<     }
<
```

```
diff -r /usr/src/kernel/proc.c /usr/src-orig/kernel/proc.c
418d417
<   int src, dst = -3;
506,507d504
<     src = proc_nr(caller_ptr);
<     dst = src_dst_p;
517,518d513
<     dst = proc_nr(caller_ptr);
<     src = src_dst_p;
522,523d516
<     src = proc_nr(caller_ptr);
<     dst = src_dst_p;
527,528d519
<         src = proc_nr(caller_ptr);
<         dst = src_dst_p;
530c521,522
<   default: result = EBADCALL;			/* illegal system call */
---
>   default:
> 	result = EBADCALL;			/* illegal system call */
533,539d524
<   if (result == OK) {
<       src += 2;
<       dst += 2;
<       if (src >= 0 && src < MSGTAB_SIZE && dst >= 0 && dst < MSGTAB_SIZE) {
<         msgtab.messages[src][dst]++;
<       }
<   }
```

```
diff -r /usr/src/kernel/system/do_getinfo.c /usr/src-orig/kernel/system/do_getinfo.c
53,57d52
<     case GET_MSGTAB: {
<         length = sizeof(struct msgtab);
<         src_vir = (vir_bytes) &msgtab;
<         break;
<     }
```

```
diff -r /usr/src/kernel/usermapped_data.c /usr/src-orig/kernel/usermapped_data.c
7d6
< struct msgtab msgtab;
Binary files /usr/src/kernel/usermapped_data.o and /usr/src-orig/kernel/usermapped_data.o differ
diff -r /usr/src/releasetools/revision /usr/src-orig/releasetools/revision
1c1
< 10
---
> 6
```

```
diff -r /usr/src/servers/is/dmp.c /usr/src-orig/servers/is/dmp.c
20d19
<     { F2,   hello_dmp, "Say hello" },
27d25
<     { F9,   msgtab_dmp, "Dump message table" },
Binary files /usr/src/servers/is/dmp.o and /usr/src-orig/servers/is/dmp.o differ
diff -r /usr/src/servers/is/dmp_kernel.c /usr/src-orig/servers/is/dmp_kernel.c
60,65d59
< void hello_dmp()
< {
<     printf("Hello world!\n");
< }
< 
< 
191,218d184
< void msgtab_dmp()
< {
<     const char *prefixes[] = { "SYS", "KER", "PM", "VFS", "RS",
<                               "MEM",  "LOG",  "TTY", "DS",  "MFS" };
<     struct msgtab msgtab;
<     int r;
<     if ((r = sys_getmsgtab(&msgtab)) != OK) {
<         printf("IS: warning: couldn't get copy of kernel info struct: %d\n", r);
<         return;
<     }
< 
<     int i, j;
<     printf(" ");
<     for (i = 0; i < MSGTAB_SIZE; i++) {
<         if (i == 0) {
<             printf("\n%6s", "");
<         }
<         printf("%6s", prefixes[i]);
<         if (i == MSGTAB_SIZE - 1) printf("\n");
<     }
<     for (i = 0; i < MSGTAB_SIZE; i++) {
<         printf("%6s", prefixes[i]);
<         for (j = 0; j < MSGTAB_SIZE; j++) {
<             printf("%6d", msgtab.messages[i][j]);
<         }
<         printf("\n");
<     }
< }
```

```
diff -r /usr/src/servers/is/proto.h /usr/src-orig/servers/is/proto.h
7,8d6
< void msgtab_dmp();
< void hello_dmp();
```

### Screenshots
![Part 2 Output][part2_ss]

[part1_ss]: part1.png
[part2_ss]: part2.png
