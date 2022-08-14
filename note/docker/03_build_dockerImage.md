


zartbot@kevin-4GPU:/opt/foo$ cat hw.go
```go
package main
import "fmt"

func main() {
    fmt.Println("hello World");
}
```
zartbot@kevin-4GPU:/opt/foo$ cat Dockerfile
```docker
FROM golang:1.14.4-alpine
WORKDIR /opt
COPY hw.go /opt
RUN go build /opt/hw.go
CMD "./hw"
```

docker build -t hw:one .
```bash
zartbot@kevin-4GPU:/opt/foo$ docker build -t hw:one .
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM golang:1.14.4-alpine
 ---> 3289bf11c284
Step 2/5 : WORKDIR /opt
 ---> Using cache
 ---> 068138bb48ea
Step 3/5 : COPY hw.go /opt
 ---> 6955e6bcb171
Step 4/5 : RUN go build /opt/hw.go
 ---> Running in dc36c25185e1
Removing intermediate container dc36c25185e1
 ---> a9afaa867f9d
Step 5/5 : CMD "./hw"
 ---> Running in 1e4b4fd33aaa
Removing intermediate container 1e4b4fd33aaa
 ---> 0ef144849dce
Successfully built 0ef144849dce
Successfully tagged hw:one
zartbot@kevin-4GPU:/opt/foo$
zartbot@kevin-4GPU:/opt/foo$
zartbot@kevin-4GPU:/opt/foo$
```

```bash
zartbot@kevin-4GPU:/opt/foo$ docker image ls
REPOSITORY   TAG             IMAGE ID       CREATED              SIZE
hw           one             0ef144849dce   About a minute ago   372MB
golang       1.14.4-alpine   3289bf11c284   2 years ago          370MB
```

多阶段
```docker
FROM golang:1.14.4-alpine as builder
WORKDIR /opt
COPY hw.go /opt
RUN go build /opt/hw.go
#CMD "./hw"

FROM scratch
COPY --from=builder /opt/hw .
```

```bash
docker build -t hw:multi .
```

image：
```bash
zartbot@kevin-4GPU:/opt/foo$ docker image ls
REPOSITORY   TAG             IMAGE ID       CREATED          SIZE
hw           multi           fc583eb6a19b   34 seconds ago   2.07MB
hw           one             0ef144849dce   4 minutes ago    372MB
golang       1.14.4-alpine   3289bf11c284   2 years ago      370MB
```

