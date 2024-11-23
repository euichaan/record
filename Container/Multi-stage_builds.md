# Multi-stage builds
Multi-stage builds는 Dockerfile을 쉽게 읽고 유지 관리하면서 최적화하는 데 어려움을 겪었던 모든 사용자에게 유용하다.  
  
## Use multi-stage builds
multi-stage builds에서는 Dockerfile에 여러 개의 FROM 문을 사용한다. 각 FROM 명령어는 서로 다른 베이스를 사용할 수 있으며, 각 명령어는 빌드의 새 단계를 시작한다.  
**필요하지 않은 모든 것을 최종 이미지에 포함시키지 않고, 특정 단계에서 다른 단계로 필요한 산출물만 선택적으로 복사할 수 있다.**      
  
다음 도커파일은 바이너리를 빌드하는 단계와 첫 번째 단계에서 다음 단계로 바이너리를 복사하는 단계로 나뉜다.  
```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.23
WORKDIR /src
COPY <<EOF ./main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=0 /bin/hello /bin/hello
CMD ["/bin/hello"]
```  
최종 결과물은 내부에 바이너리만 있는 작은 프로덕션 이미지이다. **애플리케이션을 빌드하는 데 필요한 빌드 도구는 결과 이미지에 포함되지 않는다.**  
두 번째 FROM 명령어는 `scratch` 이미지를 기본으로 하는 새 빌드 단계를 시작한다. `COPY --from` 줄은 이전 단계에서 빌드된 아티팩트만 이 새 단계로 복사한다.  
Go SDK와 모든 중간 아티팩트는 그대로 유지되며 최종 이미지에 저장되지 않는다.  
  
## Name your build stages
기본적으로 stage에는 이름이 지정되지 않으며, 첫 번째 FROM 명령어의 경우 0으로 시작하는 정수 번호로 스테이지를 참조한다.  
그러나 FROM 명령어에 `AS <NAME>`을 추가하여 스테이지에 이름을 지정할 수 있다.  
```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.23 AS build
WORKDIR /src
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=build /bin/hello /bin/hello 
CMD ["/bin/hello"]
```
스테이지 간 파일 복사를 `COPY --from`으로 하는 것을 확인할 수 있다.  
  
## Stop at a specific build stage
이미지를 빌드할 때 반드시 모든 단계를 포함하여 전체 Dockerfile을 빌드할 필요는 없다. 대상 빌드 단계를 지정할 수 있다.  
다음 명령은 이전 Dockerfile을 사용하지만 `build` 단계에서 멈춘다.  
`$ docker build --target build -t hello .`  
  
기존 Docker 엔진 빌더(legacy Docker Engine builder)는 지정된 --target에 도달하기 전까지 Dockerfile의 모든 단계를 처리한다.  
선택된 타겟이 해당 단계에 의존하지 않더라도 해당 단계를 빌드한다.  
반면, BuildKit은 타겟 단계가 의존하는 단계만 빌드한다.  
```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu AS base
RUN echo "base"

FROM base AS stage1
RUN echo "stage1"

FROM base AS stage2
RUN echo "stage2"
```
BuildKit이 활성화되어 있다면, `stage2` 타깃을 빌드하면 base 와 stage2만 처리된다. stage1에 대한 종속성이 없으므로 건너뛴다.  
반면에, BuildKit 없이 동일한 타깃을 빌드하면 모든 단계가 처리된다.  
  
## Use an external image as a stage
multi-stage build를 사용하는 경우 Docker파일에서 이전에 만든 단계에서 복사하는 것으로 제한되지 않는다.  
별도의 이미지에서 복사하는 `COPY --from` 명령어를 사용할 수 있다.  
Docker 클라이언트는 필요한 경우 이미지를 가져오고 거기에서 artifact를 복사한다.  
`$ COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf`  

## Use a previous stage as a new stage
FROM 지시문을 사용해 이전 단계를 참조하여 이전 단계에서 중단된 부분을 다시 시작할 수 있다.
```dockerfile
# syntax=docker/dockerfile:1

FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```

참고자료  
- [dockerdocs Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)