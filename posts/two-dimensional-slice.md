# Two-dimensional slice
----
`Golang` supports multiple-dimensional slice, but I only want to introduce two-dimensional slice here. One reason is the two-dimensional slice is usually used in daily life, while multiple-dimensional seems not common. If you often use multiple-dimensional slice, personally I think the code is a little clumsy and not easy to maintain, so maybe you can try to check whether there is a better method; the other reason is the principle behind multiple-dimensional slice is the same with two-dimensional slice, you can also understand it if you know two-dimensional slice well.  

Let's the following example:  

	package main
	
	import "fmt"
	
	func main() {
		s := make([][]int, 2)
		fmt.Println(len(s), cap(s), &s[0])
	
		s[0] = []int{1, 2, 3}
		fmt.Println(len(s[0]), cap(s[0]), &s[0][0])
	
		s[1] = make([]int, 3, 5)
		fmt.Println(len(s[1]), cap(s[1]), &s[1][0])
	}

I still use gdb to inspect the execution flow:  

	5       func main() {
	(gdb) n
	6               s := make([][]int, 2)
	(gdb)
	7               fmt.Println(len(s), cap(s), &s[0])
	(gdb)
	2 2 &[]
	9               s[0] = []int{1, 2, 3}
	(gdb) p &s
	$1 = (struct [][]int *) 0xc82003fe70
	(gdb) x/24xb 0xc82003fe70
	0xc82003fe70:   0x40    0x02    0x01    0x20    0xc8    0x00    0x00    0x00
	0xc82003fe78:   0x02    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc82003fe80:   0x02    0x00    0x00    0x00    0x00    0x00    0x00    0x00
`s` is a slice (the start memory address is `0xc82003fe70`), but its elements are also slices. Let's check the elements:  

	(gdb) x/48xb 0xc820010240
	0xc820010240:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010248:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010250:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010258:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010260:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010268:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
All the memory content are `0`, nothing exciting! Continue to step by step:  

	(gdb) n
	10              fmt.Println(len(s[0]), cap(s[0]), &s[0][0])
	(gdb)
	3 3 0xc82000e220
	12              s[1] = make([]int, 3, 5)
Now since `s` contains a valid slice element, check its underlying array:  
 
	(gdb) x/48xb 0xc820010240
	0xc820010240:   0x20    0xe2    0x00    0x20    0xc8    0x00    0x00    0x00
	0xc820010248:   0x03    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010250:   0x03    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010258:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010260:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010268:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
Yeah, the memory has been updated by the pointer, length and capacity of `s[0]`, the same with previous output from `fmt.Println`. Check the underlying array of `s[0]`:  

	(gdb) x/24xb 0xc82000e220
	0xc82000e220:   0x01    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc82000e228:   0x02    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc82000e230:   0x03    0x00    0x00    0x00    0x00    0x00    0x00    0x00

We can see `3` elements: `1`, `2`, `3`.  

Following the same method to check the `s[1]`:  

	(gdb) n
	13              fmt.Println(len(s[1]), cap(s[1]), &s[1][0])
	(gdb)
	3 5 0xc820010270
	14      }
	(gdb) x/48xb 0xc820010240
	0xc820010240:   0x20    0xe2    0x00    0x20    0xc8    0x00    0x00    0x00
	0xc820010248:   0x03    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010250:   0x03    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010258:   0x70    0x02    0x01    0x20    0xc8    0x00    0x00    0x00
	0xc820010260:   0x03    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010268:   0x05    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	(gdb) x/40xb 0xc820010270
	0xc820010270:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010278:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010280:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010288:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
	0xc820010290:   0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00

Now, we can see `s` contains all the info of its slice elements, and the elements of `s[1]` are initialized to `0`.