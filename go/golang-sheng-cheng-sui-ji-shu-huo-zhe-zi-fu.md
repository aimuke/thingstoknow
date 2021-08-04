# golang 生成随机数或者字符

golang支持两种随机数生成方式：

```go
math/rand          // 伪随机
crypto/rand        // 真随机
```

## math/rand

伪随机生成的数字是确定的，不论在什么机器、什么时间，只要执行的随机代码一样，那么生成的随机数就一样，例如：

```go
func main() {
   rand.Seed(2)
   for i := 0; i < 4; i++  {
      println(rand.Intn(100))
   }
}

// output
86
86
92
40
```

golang使用一个seed作为source来生成伪随机数字，默认seed为1，只有seed固定了，那么所有随机数就固定了：

```go
func(seed, 100) => xx,yy,zz
```

这里有一个坑：如果seed固定，那么每次程序重启后重新生成随机数会重复上一次的随机数

为了尽量随机性，那么我们可以每次使用不同的seed来启动程序，就可以保证每次启动都产生新的随机数，聪明的你肯定想到了使用时间戳

```go
func main() {
   rand.Seed(time.Now().UnixNano())
   for i := 0; i < 3; i++  {
      println(rand.Intn(100))
   }
}
```

使用ns时间戳可以保证每次重启seed都不一样，然后可以生成新的随机序列

## crypto/rand

crypto/rand是为了提供更好的随机性满足密码对随机数的要求，在linux上已经有一个实现就是/dev/urandom，crypto/rand 就是从这个地方读“真随机”数字返回，但性能比较慢

```go
func main() {
   for i := 0; i < 4; i++  {
      n, _ := rand.Int(rand.Reader, big.NewInt(100))
      println(n.Int64())
   }
}
```

## 性能区别

```text
name    time/op
RandWithCrypto-8  272ns ± 3%
name    time/op
RandWithMath-8   22.8ns ± 4%

// 差10倍
```

## 基于随机数生成随机字符串

以上随机只能生成随机数，无法生成随机字符串，但我们实际开发中还是是字符串和数字混合为主，所以列出基于随机数字来生成随机字符串的方法：

```text
var defaultLetters = []rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789")

// RandomString returns a random string with a fixed length
func RandomString(n int, allowedChars ...[]rune) string {
	var letters []rune

	if len(allowedChars) == 0 {
		letters = defaultLetters
	} else {
		letters = allowedChars[0]
	}

	b := make([]rune, n)
	for i := range b {
		b[i] = letters[rand.Intn(len(letters))]
	}

	return string(b)
}
```

## 总结

* 对于不涉及到密码类的开发工作直接使用math/rand+基于时间戳的种子rand.Seed\(time.Now\(\).UnixNano\(\)\)一般都能满足需求
* 对于涉及密码类的开发工作一定要用crypto/rand
* 如果想生成随机字符串，可以先列出字符串，然后基于随机数选字符的方式实现

## 参考

原文 [golang 生成随机数或者字符](https://zhuanlan.zhihu.com/p/94684495)

[https://medium.com/a-journey-with-go/go-how-are-random-numbers-generated-e58ee8696999​medium.com/a-journey-with-go/go-how-are-random-numbers-generated-e58ee8696999](https://link.zhihu.com/?target=https%3A//medium.com/a-journey-with-go/go-how-are-random-numbers-generated-e58ee8696999)[https://github.com/thoas/go-funk​github.com/thoas/go-funk](https://link.zhihu.com/?target=https%3A//github.com/thoas/go-funk)

发布于 2019-12-02

