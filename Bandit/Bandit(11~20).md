# Bandit 11~20

## Level 11 -> 12

Level Goal

```txt
The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions
```

### 문제 접근

```bash
bandit11@bandit:~$ ls -lh data.txt
-rw-r----- 1 bandit12 bandit11 49 Oct 14 09:25 data.txt

bandit11@bandit:~$ cat data.txt
Gur cnffjbeq vf 7k16JArUVv5LxVuJfsSVdbbtaHGlw9D4
```

먼저 `data.txt`를 확인해보니, 패스워드가 담겨 있는 문자열이 암호화되어있는 것 같다. 레벨 목표를 확인해보니 각 알파벳이 13자리씩 이동해 대체되어있다고 한다.

레퍼런스 : [ROT13](https://en.wikipedia.org/wiki/ROT13)

`tr` 명령어를 사용해 해당 문자열을 원문으로 복호화해준다.

### 풀이

```bash
bandit11@bandit:~$ tr 'A-Za-z' 'N-ZA-Mn-za-m' <<< "Gur cnffjbeq vf 7k16JArUVv5LxVuJfsSVdbbtaHGlw9D4"
The password is 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4
```

추가로 ROT13 암호화 알고리즘 같은 경우 복호화를 해주는 사이트들을 적극 활용하는 것도 좋다.

## Level 12 -> 13

Level Goal

```txt
The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command “mktemp -d”. Then copy the datafile using cp, and rename it using mv (read the manpages!)
```

### 문제 접근

이번 패스워드는 `repeatedly compressed`된 파일 안에 들어있다고 한다. 우선 레벨 목표에서 제안하는대로 작성 권한이 있는 디렉토리에 문제 파일을 복사해서 사용하겠다.

```bash
bandit12@bandit:~$ mktemp -d
/tmp/tmp.LQuEAMCB3g

bandit12@bandit:~$ cd /tmp/tmp.LQuEAMCB3g
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ cp ~/data.txt .
```

### 풀이

우선 제시된 `data.txt` 파일은 `hexdump`, 즉 16진수로 변환되어있는 파일이니 `xxd`를 활용해 바이너리로 바꿔준다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ xxd -r data.txt > zipdata
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file zipdata
zipdata: gzip compressed data, was "data2.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, original
 size modulo 2^32 572
```

바꿔준 파일의 유형을 확인해보니 `gzip`으로 압축된 파일이라고 한다.
`gzip` 압축을 해제하기 위해 파일 확장자를 `.gz`으로 변경하고, 압축을 해제한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv zipdata zipdata.gz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ gzip -d zipdata.gz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file zipdata
zipdata: bzip2 compressed data, block size = 900k
```

이번에는 bzip2로 압축되어있다고 하니, .bz로 확장자를 변경한 후 압축 해제를 진행한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv zipdata zipdata.bz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ bzip2 -d zipdata.bz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file zipdata
zipdata: gzip compressed data, was "data4.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, original
 size modulo 2^32 20480
```

다시 gzip으로 압축된 파일이니, 파일 확장자 변경 및 압축 해제를 진행한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv zipdata zipdata.gz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ gzip -d zipdata.gz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file zipdata
zipdata: POSIX tar archive (GNU)
```

이번에는 `tar`로 압축되어있는 파일이라고 한다. 파일 확장자를 `.tar`로 변경해주고 압축 해제한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv zipdata zipdata.tar
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ tar -xvf zipdata.tar
data5.bin
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file data5.bin
data5.bin: POSIX tar archive (GNU)
```

다시 `tar` 형식이니, 같은 과정을 한 번 더 진행한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv data5.bin data5.tar
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ tar -xvf data5.tar
data6.bin
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
```

다시 `bzip2` 압축 해제 과정을 진행한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv data6.bin data6.bz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ bzip2 -d data6.bz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file data6
data6: POSIX tar archive (GNU)
```

`tar` 압축 해제 과정을 진행한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv data6 data6.tar
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ tar -xvf data6.tar
data8.bin
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Tue Oct 14 09:26:00 2025, max compression, from Unix, origin
al size modulo 2^32 49
```

`gzip` 압축 해제 과정을 진행한다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ mv data8.bin data8.gz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ gzip -d data8.gz
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ file data8
data8: ASCII text
```

이번에는 사람이 읽을 수 있는 형식의 파일이 나왔다. `cat` 명령어를 통해 해당 파일을 읽어준다.

```bash
bandit12@bandit:/tmp/tmp.LQuEAMCB3g$ cat data8
The password is FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
```
