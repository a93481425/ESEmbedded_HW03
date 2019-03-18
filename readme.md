HW03

===

This is the hw03 sample. Please follow the steps below.



# Build the Sample Program



1. Fork this repo to your own github account.



2. Clone the repo that you just forked.



3. Under the hw03 dir, use:



	* `make` to build.



	* `make clean` to clean the ouput files.



4. Extract `gnu-mcu-eclipse-qemu.zip` into hw03 dir. Under the path of hw03, start emulation with `make qemu`.



	See [Lecture 02 ─ Emulation with QEMU] for more details.



5. The sample is a minimal program for ARM Cortex-M4 devices, which enters `while(1);` after reset. Use gdb to get more details.



	See [ESEmbedded_HW02_Example] for knowing how to do the observation and how to use markdown for taking notes.



# Build Your Own Program



1. Edit main.c.



2. Make and run like the steps above.



3. Please avoid using hardware dependent C Standard library functions like `printf`, `malloc`, etc.



# HW03 Requirements



1. How do C functions pass and return parameters? Please describe the related standard used by the Application Binary Interface (ABI) for the ARM architecture.



2. Modify main.c to observe what you found.



3. You have to state how you designed the observation (code), and how you performed it.



	Just like how you did in HW02.



3. If there are any official data that define the rules, you can also use them as references.



4. Push your repo to your github. (Use .gitignore to exclude the output files like object files or executable files and the qemu bin folder)



[Lecture 02 ─ Emulation with QEMU]: http://www.nc.es.ncku.edu.tw/course/embedded/02/#Emulation-with-QEMU

[ESEmbedded_HW02_Example]: https://github.com/vwxyzjimmy/ESEmbedded_HW02_Example



--------------------



- [ ] **If you volunteer to give the presentation next week, check this.**



--------------------



**★★★ Please take your note here ★★★**

HW03 

===

## 1. 實驗題目

用組合語言觀察編譯器如何處理C語言函式中的引數傳遞與回傳值。

## 2. 實驗步驟

1. 設計C語言的測試程式 main.c 
其中`reset_handler `為主程式，功能為設兩個變數`a`與`b`並呼叫函數`int add(int a,int b)`，
將傳過去的引數相加，並回傳兩引數的和回來主程式。

main.c:



```

int add(int a,int b);

void reset_handler(void)
{
        int a=25;
        int b=126;
        int r=0;
        r=add(a,b);
        while (1){
        }

}

int add(int a,int b)
{
 return a+b;
}

```



4. 將 main.s 編譯並以 qemu 模擬， `$ make clean`, `$ make`, `$ make qemu`

開啟另一 Terminal 連線 `$ arm-none-eabi-gdb` ，再輸入 `target remote 127.0.0.1:1234` 連接，輸入兩次的 `info registers` 再輸入 `layout regs`, 輸入 `si` 執行單步除錯。



5. 其中將編譯好的檔案進行反組譯得到如下結果


```assembly

00000000 <_start-0x8>:
   0:	20000100 	andcs	r0, r0, r0, lsl #2
   4:	00000009 	andeq	r0, r0, r9

00000008 <_start>:
   8:	e000      	b.n	c <reset_handler>
	...

0000000c <reset_handler>:
   c:	b580      	push	{r7, lr}
   e:	b084      	sub	sp, #16
  10:	af00      	add	r7, sp, #0
  12:	2319      	movs	r3, #25
  14:	60fb      	str	r3, [r7, #12]
  16:	237e      	movs	r3, #126	; 0x7e
  18:	60bb      	str	r3, [r7, #8]
  1a:	2300      	movs	r3, #0
  1c:	607b      	str	r3, [r7, #4]
  1e:	68f8      	ldr	r0, [r7, #12]
  20:	68b9      	ldr	r1, [r7, #8]
  22:	f000 f803 	bl	2c <add>
  26:	6078      	str	r0, [r7, #4]
  28:	e7fe      	b.n	28 <reset_handler+0x1c>
  2a:	bf00      	nop

0000002c <add>:
  2c:	b480      	push	{r7}
  2e:	b083      	sub	sp, #12
  30:	af00      	add	r7, sp, #0
  32:	6078      	str	r0, [r7, #4]
  34:	6039      	str	r1, [r7, #0]
  36:	687a      	ldr	r2, [r7, #4]
  38:	683b      	ldr	r3, [r7, #0]
  3a:	4413      	add	r3, r2
  3c:	4618      	mov	r0, r3
  3e:	370c      	adds	r7, #12
  40:	46bd      	mov	sp, r7
  42:	f85d 7b04 	ldr.w	r7, [sp], #4
  46:	4770      	bx	lr


```
6.由`0x8`呼叫C語言的函數

```assembly
00000008 <_start>:
   8:	e000      	b.n	c <reset_handler>
```

7.由程式的`0xc`可以看到在進入`reset_handler`程式時，會先將`r7`與`lr`暫存器推入堆疊，並根據變數數量決定將SP往下減多少作為放置區域變數的空間
根據測試結果當只有一個int時`sub      sp, #8`，3個時`sub	sp, #16`，3個以上時`sub	sp, #24`，推測是一個int占16Bit，一個堆疊32Bit所致。

觀察`0x10`可發現將sp的值送到r7，將r7暫存器當成基底地址使用

```assembly
   c:	b580      	push	{r7, lr}
   e:	b084      	sub	sp, #16
 10:	af00      	add	r7, sp, #0
```

8.程式的`0x12`開始將變數`a`放入`r3`，並在其後一行`0x14`將當作基底地址的`r7+12`之後的地方填入a變數
同理，`0x16`到`0x18`與`0x1a`到`0x1c`分別是對變數`a`與變數`b`做處理

```assembly
 
  12:	2319      	movs	r3, #25
  14:	60fb      	str	r3, [r7, #12]
  16:	237e      	movs	r3, #126	; 0x7e
  18:	60bb      	str	r3, [r7, #8]
  1a:	2300      	movs	r3, #0
  1c:	607b      	str	r3, [r7, #4]

```

9.將剛剛存入區域變數區的兩個變數放入`r0`與`r1`並跳轉至副程式`add`

```assembly

  1e:	68f8      	ldr	r0, [r7, #12]
  20:	68b9      	ldr	r1, [r7, #8]
  22:	f000 f803 	bl	2c <add>

```

10.在副程式中，同樣的也是先將`r7`推入堆疊之中，以免回去主程式的時候基底位置不見了
由`0x2e`到`0x30`這兩行劃分副程式所使用的區域變數記憶體

```assembly

  2c:	b480      	push	{r7}
  2e:	b083      	sub	sp, #12
  30:	af00      	add	r7, sp, #0

```

11.將主程式送來的引數從`r0`,`r1`取出後放入副程式的區域變數空間
`0x36`與`0x38`同樣的，跟主程式一樣，將變數利用r7將運算的東西放到`r2`與`r3`這兩個變數準備做計算

```assembly

  32:	6078      	str	r0, [r7, #4]
  34:	6039      	str	r1, [r7, #0]
  36:	687a      	ldr	r2, [r7, #4]
  38:	683b      	ldr	r3, [r7, #0]

```

12.將相加的結果放到`r3`，並將回傳值放入`r0`準備回傳。

```assembly

  3a:	4413      	add	r3, r2
  3c:	4618      	mov	r0, r3

```

13.將`r7`復原成如同剛進副程式`0x2c`的模樣

```assembly

  3e:	370c      	adds	r7, #12
  40:	46bd      	mov	sp, r7
  42:	f85d 7b04 	ldr.w	r7, [sp], #4

```

14.再藉由跳到副程式時存的`lr`暫存器讓程式繼續執行

```assembly

  46:	4770      	bx	lr

```

15.進入無窮迴圈`while(1)`執行

```assembly

 26:	6078      	str	r0, [r7, #4]
 28:	e7fe      	b.n	28 <reset_handler+0x1c>

```

## 3. 結果與討論

1. 函數間的傳值是由r0,r1,r2進行(不論引數與return的值皆是如此)

2. 區域變數的劃分是放在堆疊記憶體裡面，但是編譯器用r7把SP暫存器做平移，使其可以正常使用堆疊也可以做區域變數的存放




