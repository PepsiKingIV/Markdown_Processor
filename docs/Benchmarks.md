# Benchmarks

### 1
#### Комментарий
Код до каких-либо изменений

#### Изменения в коде
-

#### Тесты
    ```
    goos: darwin
    goarch: arm64
    pkg: Markdown_Processor/cmd
    BenchmarkMain-8                         Totat op: 100,   errors 0
            Totat op: 321,   errors 0
         321           3718841 ns/op         5576091 B/op      63467 allocs/op
            Totat op: 1,     errors 0
    BenchmarkHEADING-8                      Totat op: 100,   errors 0
            Totat op: 10000,         errors 41
            Totat op: 131205,        errors 529
      131205              9085 ns/op           15341 B/op        172 allocs/op
            Totat op: 1,     errors 0
    BenchmarkWORD-8                         Totat op: 100,   errors 0
            Totat op: 10000,         errors 0
            Totat op: 101533,        errors 0
      101533             11700 ns/op           17449 B/op        197 allocs/op
            Totat op: 1,     errors 0
    BenchmarkLIST-8                         Totat op: 100,   errors 0
            Totat op: 8817,          errors 0
        8817            122657 ns/op          185243 B/op       2131 allocs/op
            Totat op: 1,     errors 0
    BenchmarkNUMBEREDLIST-8                 Totat op: 100,   errors 0
            Totat op: 8215,          errors 0
        8215            140440 ns/op          214644 B/op       2455 allocs/op
            Totat op: 1,     errors 0
    BenchmarkBOLT-8                         Totat op: 100,   errors 0
            Totat op: 10000,         errors 0
            Totat op: 52081,         errors 0
       52081             22954 ns/op           35453 B/op        412 allocs/op
    PASS
    ok      Markdown_Processor/cmd  8.056s
    ```

### 2
#### Комментарий
Чтобы уменьшить количество аллокаций, создам массив определенного размера. На тесты это не повлияет, т.к. они не оценивают функции чтения.

#### Изменения в коде
.cmd/main.go#[30]

`FileWithoutByte0 := make([]byte, len(file))`

#### Тесты
    ```
    goos: darwin
    goarch: arm64
    pkg: Markdown_Processor/cmd
    BenchmarkMain-8                         Totat op: 100,   errors 0
            Totat op: 302,   errors 0
         302           3762007 ns/op         5626021 B/op      64346 allocs/op
            Totat op: 1,     errors 0
    BenchmarkHEADING-8                      Totat op: 100,   errors 0
            Totat op: 10000,         errors 41
            Totat op: 117484,        errors 481
      117484              9979 ns/op           16437 B/op        186 allocs/op
            Totat op: 1,     errors 0
    BenchmarkWORD-8                         Totat op: 100,   errors 0
            Totat op: 10000,         errors 0
            Totat op: 108308,        errors 0
      108308             10688 ns/op           16152 B/op        180 allocs/op
            Totat op: 1,     errors 0
    BenchmarkLIST-8                         Totat op: 100,   errors 0
            Totat op: 9514,          errors 0
        9514            122735 ns/op          186077 B/op       2138 allocs/op
            Totat op: 1,     errors 0
    BenchmarkNUMBEREDLIST-8                 Totat op: 100,   errors 0
            Totat op: 7696,          errors 0
        7696            145074 ns/op          219286 B/op       2529 allocs/op
            Totat op: 1,     errors 0
    BenchmarkBOLT-8                         Totat op: 100,   errors 0
            Totat op: 10000,         errors 0
            Totat op: 50112,         errors 3
       50112             24622 ns/op           36790 B/op        429 allocs/op
    PASS
    ok      Markdown_Processor/cmd  8.199s
    ```

### 3
#### Комментарий
Есть гипотеза, что длина строки и сложность разбора коррелируют. Поэтому решил разбить большой текст по строкам. По памяти это не даст прирост, но из-за меньшего количества проходов, необходимых для правильного лексического анализа, программа должна потреблять всех ресурсов меньше

#### Изменения в коде
.internal/processing/Lexer.go#[17]

`if count > (len(L.Code)*2)/3 {`

.cmd/bench_test.go#[65-73]

```
tokens := strings.Split(GeneralTest, "\n")
	for i := 0; i < b.N; i++ {
		for _, j := range tokens {
			TotalOperations += 1
			TT := TestedToken{TestTokenType: "main", TestToken: j}
			if err := TMain(TT); err != nil {
				errors += 1
			}
		}
```

#### Тесты
        ```
        goos: darwin
        goarch: arm64
        pkg: Markdown_Processor/cmd
        BenchmarkMain-8                         Totat op: 2000,          errors 2
                Totat op: 5360,          errors 2
                268           3857790 ns/op         4988687 B/op      56923 allocs/op
                Totat op: 1,     errors 1
        BenchmarkHEADING-8                      Totat op: 100,   errors 4
                Totat op: 10000,         errors 654
                Totat op: 124326,        errors 7527
                124326              9260 ns/op           14281 B/op        162 allocs/op
                Totat op: 1,     errors 0
        BenchmarkWORD-8                         Totat op: 100,   errors 0
                Totat op: 10000,         errors 0
                Totat op: 105986,        errors 0
                105986             11035 ns/op           16370 B/op        181 allocs/op
                Totat op: 1,     errors 0
        BenchmarkLIST-8                         Totat op: 100,   errors 0
                Totat op: 9554,          errors 0
                9554            120824 ns/op          178536 B/op       2042 allocs/op
                Totat op: 1,     errors 0
        BenchmarkNUMBEREDLIST-8                 Totat op: 100,   errors 0
                Totat op: 8535,          errors 0
                8535            134738 ns/op          202298 B/op       2313 allocs/op
                Totat op: 1,     errors 0
        BenchmarkBOLT-8                         Totat op: 100,   errors 0
                Totat op: 10000,         errors 0
                Totat op: 53166,         errors 2
                53166             22129 ns/op           34497 B/op        397 allocs/op
        PASS
        ok      Markdown_Processor/cmd  7.833s
        ```