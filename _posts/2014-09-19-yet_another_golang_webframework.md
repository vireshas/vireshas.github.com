---
layout: post
title: "Yet another golang web-framework"
tagline: 'golang webframework, negroni usage, REST api in golang'
---

I have been writing a web-framework in golang for quite sometime now. Lets write a sample webapp using this framework.  

>`mkdir ~/webapp`  
>`cd !$`  
>`export GOPATH=`\``pwd`\`    
>`go get github.com/codegangsta/negroni`  
>`go get github.com/vireshas/mantle`  
>`go get github.com/vireshas/t-coredb`  
>`go get github.com/vireshas/t-settings`  

create a file "main.go" and the add these lines   
  
        package main

        import (
                "github.com/codegangsta/negroni"
                "net/http"
                "router"
        )

        func main() {
                mux := http.NewServeMux()
                mux.HandleFunc("/", router.Greet)

                n := negroni.Classic()
                n.UseHandler(mux)
                n.Run(":3000")
        }  

>`mkdir src/router`  
>`cd !$`  
 
create a file "greet.go" and add these lines  

        package router

        import (
                "fmt"
                "net/http"
        )

        func Greet(w http.ResponseWriter, req *http.Request) {
                fmt.Fprintln(w, "Hello world!")
        }


>`cd ../..`  
>`go run main.go`   

In another terminal hit "curl localhost:3000"   

[Calling Mysql and Redis with this framework](/2014/09/20/calling_mysql_settings_and_redis/)
