---
layout: post
title: "Using Mysql and Redis with our webapp"
tagline: 'golang webframework, REST api in golang, mysql golang, redis golang'
---

We just created a sample hello world app in our previous blog. Lets know invoke some Mysql and Redis calls.  

This is your new main.go  

        package main

        import (
                "github.com/codegangsta/negroni"
                "github.com/vireshas/t-settings"
                "net/http"
                "router"
        )

        //configure before starting server
        func configure(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
                settings.Configure("src/settings/config.json")
                next(rw, r)
        }

        func main() {
                mux := http.NewServeMux()
                mux.HandleFunc("/", router.Greet)
                mux.HandleFunc("/mysql", router.MysqlCall)
                mux.HandleFunc("/redis", router.RedisCall)
                n := negroni.Classic()
                n.Use(negroni.HandlerFunc(configure))
                n.UseHandler(mux)
                n.Run(":3000")
        }


Few more files are goes in  src/router  
1. src/router/mysql.go <- this talks to mysql  

        package router

        import (
                "fmt"
                "github.com/vireshas/t-coredb"
                "net/http"
        )

        func MysqlCall(rw http.ResponseWriter, r *http.Request) {
                mysqldb := db.GetMysqlClientFor("m1")
                var msg string
                err := mysqldb.QueryRow("SELECT value FROM bm WHERE id=?", 1).Scan(&msg)
                if err != nil {
                        fmt.Fprintln(rw, "Database Error!")
                        fmt.Fprintln(rw, err)
                } else {
                        fmt.Fprintln(rw, msg)
                }
        }

2. src/router/redis.go <- using Redis  
 
        package router

        import (
                "fmt"
                "github.com/vireshas/t-coredb"
                "net/http"
        )

        func RedisCall(rw http.ResponseWriter, r *http.Request) {
                connection := db.GetRedisClientFor("r1")
                value := connection.Get("key")
                fmt.Fprintln(rw, value)
        }


>       mkdir src/settings  
>       cp src/github.com/vireshas/t-settings/config.json src/settings/   
>       go run main.go 

Fire another terminal and hit these urls  
localhost:3000/  
localhost:3000/mysql   
localhost:3000/redis  

