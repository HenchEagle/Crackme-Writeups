# Crackme Writeup - Orbit Fold - 2.2 Difficulty
https://crackmes.one/crackme/6ab1e2bf5c861323b6300702

---

## Analysis

Running the file I can see that it simply takes an input (the flag), and then outputs whether it was correct
or not.

So I ran a quick strings on the file to see if maybe it was hidden anywhere within the binary. I saw that it utilises
strlen, strcspn, fgets and puts. With this i assumed that the flag is potentially accepted through the fgets function
so I ran a ltrace on the file.

```
[custard@custard orbit_with_script_mwahahaha]$ ltrace ./orbit_fold_linux_x86_64 
puts("Orbit Fold - recover the accepte"...Orbit Fold - recover the accepted flag.
)                           = 40
fwrite("Flag: ", 1, 6, 0x7fa1ab21a580)                                = 6
fgets(Flag: asd
"asd\n", 128, 0x7fa1ab2198e0)                                   = 0x7ffc3090afe0
strcspn("asd\n", "\r\n")                                              = 3
strlen("asd")                                                         = 3
puts("No orbit. Try again."No orbit. Try again.
)                                          = 21
+++ exited (status 1) +++
```

Unfortunately nothing useful was there so I moved onto static analysis via objdump -d orbit -M intel

The first thing I looked for was the fgets, and fwrite function to see whether this was where my input was being taken.

```ASM
    11bb:	48 8d 3d 9e 0e 00 00 	lea    rdi,[rip+0xe9e]        # 2060 <__cxa_finalize@plt+0xfe0>
    11c2:	e8 a9 fe ff ff       	call   1070 <fwrite@plt>
    11c7:	48 8b 15 52 2e 00 00 	mov    rdx,QWORD PTR [rip+0x2e52]        # 4020 <stdin@GLIBC_2.2.5>
    11ce:	be 80 00 00 00       	mov    esi,0x80
    11d3:	48 89 e7             	mov    rdi,rsp
    11d6:	e8 85 fe ff ff       	call   1060 <fgets@plt>
```

I ran the file in gdb and set a breakpoint @ `0x0000555555554000 + 11bb` to calculate the offset from the memory address
at which the file began running. I retrieved this value by running `info proc mappings`

Stepping through the program I confirmed that this is where the flag is inputted. So that was great, following the trace it performed some random strcspn before jumping to the following address.

```ASM
    11f7:	e9 cb fe ff ff       	jmp    10c7 <__cxa_finalize@plt+0x47>
```

Navigating to that address we can see that a value is moved into rdi and a `strlen()` is performed on it, I assumed that this is my input, however to make sure I once again set a breakpoint on that region and ran `print $rbx` to make sure. 

```ASM
    10c7:	48 89 df             	mov    rdi,rbx
    10ca:	e8 71 ff ff ff       	call   1040 <strlen@plt>
    10cf:	48 83 f8 17          	cmp    rax,0x17
    10d3:	74 1b                	je     10f0 <__cxa_finalize@plt+0x70>
```

Looking at this comparison condition it is checking whether the output of `strlen() == 23` so from now on any test inputs will be 23 characters long.

Continuining along the trace, and bypassing the jump to 10f0 (The exit statement) by inputting a flag of 23 characters another 2 comparison are hit.

```ASM
    10f0:	b8 01 00 00 00       	mov    eax,0x1
    10f5:	31 c9                	xor    ecx,ecx
    10f7:	66 0f 1f 84 00 00 00 	nop    WORD PTR [rax+rax*1+0x0]
    10fe:	00 00 
    1100:	0f b6 54 03 ff       	movzx  edx,BYTE PTR [rbx+rax*1-0x1]
    1105:	0f af d0             	imul   edx,eax
    1108:	48 83 c0 01          	add    rax,0x1
    110c:	01 d1                	add    ecx,edx
    110e:	48 83 f8 18          	cmp    rax,0x18
    1112:	75 ec                	jne    1100 <__cxa_finalize@plt+0x80>
    1114:	66 81 f9 8a 72       	cmp    cx,0x728a
    1119:	75 ba                	jne    10d5 <__cxa_finalize@plt+0x55>
```

Looking at this code it iterates either in the form `for int i = 1; i < 18; i++` or `while(i < 18)` - with each itteration moving each character within our string sequentially into `edx` and then multiplying it against the value of `i` to reach a total of 0x728a. Im assuming this is a check to ensure that our flag is roughly the right password.

I very easily bypassed this by once again setting a breakpoint, this time @ `110e` and setting the values of $rax and $cx to 0x18 and 0x728a respectively inside of gdb.

## Exploitation

Now came the solution to our problem

```ASM
    111b:	48 8d 35 7e 0f 00 00 	lea    rsi,[rip+0xf7e]        # 20a0 <__cxa_finalize@plt+0x1020>
    1122:	31 ff                	xor    edi,edi
    1124:	4c 8d 0d 55 0f 00 00 	lea    r9,[rip+0xf55]        # 2080 <__cxa_finalize@plt+0x1000>
    112b:	41 b8 cd cc cc cc    	mov    r8d,0xcccccccd
    1131:	4c 8d 56 17          	lea    r10,[rsi+0x17]
    1135:	66 66 2e 0f 1f 84 00 	data16 cs nop WORD PTR [rax+rax*1+0x0]
    113c:	00 00 00 00 
    1140:	0f b6 0e             	movzx  ecx,BYTE PTR [rsi]
    1143:	48 83 c6 01          	add    rsi,0x1
    1147:	8d 04 cd 00 00 00 00 	lea    eax,[rcx*8+0x0]
    114e:	0f b6 d1             	movzx  edx,cl
    1151:	41 89 cb             	mov    r11d,ecx
    1154:	89 cd                	mov    ebp,ecx
    1156:	29 c8                	sub    eax,ecx
    1158:	83 c0 31             	add    eax,0x31
    115b:	32 04 13             	xor    al,BYTE PTR [rbx+rdx*1]
    115e:	4c 89 da             	mov    rdx,r11
    1161:	49 0f af d0          	imul   rdx,r8
    1165:	48 c1 ea 22          	shr    rdx,0x22
    1169:	8d 14 92             	lea    edx,[rdx+rdx*4]
    116c:	29 d1                	sub    ecx,edx
    116e:	8d 54 6d 00          	lea    edx,[rbp+rbp*2+0x0]
    1172:	83 c1 01             	add    ecx,0x1
    1175:	8d 54 95 00          	lea    edx,[rbp+rdx*4+0x0]
    1179:	d2 c0                	rol    al,cl
    117b:	83 f2 5a             	xor    edx,0x5a
    117e:	01 d0                	add    eax,edx
    1180:	43 32 04 19          	xor    al,BYTE PTR [r9+r11*1]
    1184:	09 c7                	or     edi,eax
    1186:	4c 39 d6             	cmp    rsi,r10
    1189:	75 b5                	jne    1140 <__cxa_finalize@plt+0xc0>
    118b:	40 84 ff             	test   dil,dil
    118e:	0f 85 41 ff ff ff    	jne    10d5 <__cxa_finalize@plt+0x55>
    1194:	48 8d 3d 95 0e 00 00 	lea    rdi,[rip+0xe95]        # 2030 <__cxa_finalize@plt+0xfb0>
    119b:	e8 90 fe ff ff       	call   1030 <puts@plt>
```

Analysis of the above code shows us a couple of things.

- Some random points in memory are being loaded into rsi and r9, with r10 being set to rsi + 23.
That can give a bit of a clue as to what is stored there considering our flag must also be 23 chars long.

- It also looks like a bunch of random transformations are being performed with line `115b` including our character buffer within it incremented by some value `rdx`.

- This is a loop that itterates through all 23 values stored at rsi `1180`.

- And a final test of dil, dil which only passes onto to success if the `dil` register is == 0.

Based on that, every iteration of this loop must = 0, so that can be the start of our exploit code,
```python
if dil == 0:
  character = x
```

With that basis, let's take a look at what is stored inside r9 and rsi. as they are referenced several times through this cryptographic block, I take a look at their contents by once again setting the same breakpoint as earlier, correcting the values of rax and cl, then steppping through until all the values are loaded and examining the contents at their pointers.

x/32b $rsi
```
(gdb) x/32b $rsi
0x5555555560a0:	7	0	19	3	14	22	5	11
0x5555555560a8:	1	17	9	20	6	13	2	16
0x5555555560b0:	8	21	4	12	18	10	15	0
0x5555555560b8:	1	27	3	59	44	0	0	0
```
x/32b $r9
```
(gdb) x/32b $r9
0x555555556080:	62	44	-61	80	-78	103	-8	89
0x555555556088:	3	20	-6	25	21	-127	-54	-106
0x555555556090:	-123	109	61	-88	27	-47	-7	0
0x555555556098:	0	0	0	0	0	0	0	0
```

Their contents suprisingly seem to be 23 digits which seems quite useful, let's reference that back to the disassembly.

Looking at r9 usage at the bottom, we can see that the value stored inside al is xored against some other value *`r9+r11` before being compared against 0. This indicates that for us to retrieve the success message, al must be == to 0 by the end of the itteration.

So let's take a look at what happens here, we can see that rsi is incremented, so each value inside the $rsi memory pointer is accessed once and iterated through demonstrated by the instructions @ `1143` and `1186` This "key" is stored inside the `r11` register, where some pointer arithmetic is performed to access the value within the r9 lookup table at index of **key**.

Therefore, each iteration we now know that before the xor, each iteration al must be == to r9[key]. As only a single ascii value can be equivelant to that exact value after transformation. We now know that this can be easily brute-forced by simply running through the maths then comparing it to those values within $r9. 

Based on the instruction `xor    al,BYTE PTR [rbx+rdx*1]` Where our input character is actually referenced, and rdx being equal to the key, we also know that the value of our key is being accessed by the index of the key.

So each itteration is performed on input_string[key] and compared to the final value lookup_table_r9[key].

```python
#!/usr/bin/env python3

keys_table_rsi = [
	7, 0, 19, 3, 14, 22, 5, 11,
	1, 17, 9, 20, 6, 13, 2, 16,
	8, 21, 4, 12, 18, 10, 15
]

lookup_table_r9 = [
	62, 44, -61, 80, -78, 103, -8, 89,
	3, 20, -6, 25, 21, -127, -54, -106,
	-123, 109, 61, -88, 27, -47, -7
]

password = ["?"] * 23

def rol(value, rot_amount):
	for i in range(rot_amount):
		shifted_left = value << 1
		shifted_right = value >> 7
		value = shifted_left | shifted_right
		value &= 0xFF	
	return value


for key in keys_table_rsi:
	# 1147 - 1158
	eax_1147 = key * 8		# lea [rcx * 8]
	eax_1156 = eax_1147 - key	# sub eax, ecx
	eax_1158 = eax_1156 + 49

	
	# 115b - 
	found = False
	for char in range(32, 127):
		if found:
			break

		al_115b = eax_1158 & 0xFF
		al_115b ^= char

		r8 = 3435973837
			
		rdx_115e = key * r8
		rdx_115e = rdx_115e >> 34
		
		edx_1169 = rdx_115e + (rdx_115e * 4)
		edx_1169 = edx_1169 & 0xFFFFFFFF


		ecx_116c = key - edx_1169
		
		edx_116e = key + (key * 2)

		ecx_1172 = ecx_116c + 1

		edx_1175 = key + (edx_116e * 4)


		rotation_amount = ecx_1172 & 0xFF

		al_1179 = rol(al_115b, rotation_amount)


		edx_117b = edx_1175 ^ 90
		
		eax_117e = al_1179 + edx_117b


		al_1180 = eax_117e & 0xFF

		if al_1180 == (lookup_table_r9[key] & 0xFF):
			print(f"{chr(char)} || {char}")
			password[key] = chr(char)
			found = True
			break

print("".join(password))
```

We run this script, copying the exact steps taken within the disassembly - amazing we have our first ever flag!

`CMO{orbit_folded_twice}`
