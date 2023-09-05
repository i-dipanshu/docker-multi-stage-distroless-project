## Description 
Our application is a simple calculator app developed using Golang. The idea behind choosing a application build using golang, is to show the maximum advantages of using this approach to build images. However, We can do same for any language and runtime by using a different Distroless Image for their runtime.

## Advantages 
- Using multi-stage build and distroless images, reduces the image size. In our case the image is reduced from 864mb to 1.83mb which is almost 800% reduction in image size. 

    ![docker-images-diff ](https://github.com/jenkinsci/test-results-analyzer-plugin/assets/84374342/453c4986-0ac9-4d8a-8137-2c95cd511e6c)

- Using distroless images for running the application provides more security against os bugs and vulnerabilities as distroless images have bare-minimum os configured 


## Implementation 

Golang is static typed language. So, it doesn't need any language specific runtime to run. However, it do require golang to build the application binaries. 

So, We'll build the application in 1st stage by installing the binaries. In the second stage, we'll start by using a `scratch` image (a Distroless Image), and then copying the already build application from the 1st stage and run the application.

## Build Image without Multi-Stage build and Distroless Images
1. Build the image 

    ```sh
    docker build -t go_calculator:v1 -f Dockerfile-unoptimized .
    ```

2. List the image 

    ```sh
    docker images
    ```

3. Notice the size of image, which is about 900mb our a simple calculator app. 

## Build Image with Multi-Stage build and Distroless Images
1. Build the image 

    ```sh
    docker build -t go_calculator_optimized:v1 -f Dockerfile-optimized .
    ```

2. List the image 

    ```sh
    docker images
    ```

3. Notice the size of image, which is about 2mb, which is about 800% times smaller than image created without using multi-stage build & distroless images.

## Understanding Building application in Golang
```sh 
ENV GO111MODULE=off
```
Above commands sets an environment variable GO111MODULE to "off." In Go, modules are a way to manage dependencies. Setting this variable to "off" disables the use of modules, which may be necessary for older Go projects that don't use modules for dependency management.

```sh
RUN CGO_ENABLED=0 go build -o /app .
```
This line runs the Go build command. 

Above commands compiles our Go application (go build) with the CGO_ENABLED=0 environment variable, which disables cgo (the Go foreign function interface). This is often done to create statically-linked binaries that don't have external dependencies.

The -o /app flag specifies that the compiled binary should be named "app" and placed in the root directory of the image.

The . at the end tells Go to build the application using the source code in the current directory.

## Conclusion 
Using multi-stage build docker images not only provides lesser image size but also grants more security