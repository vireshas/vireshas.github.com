Chunk response, 4 Redis reads, 10 MySQL reads, 1 text  
Throughput of 780 to 795 for GOMAXPROCS = 2,3 or 4 and it was 520 for GOMAXPROCS = 1  
        
        package main

        import (
                "runtime"
                "github.com/codegangsta/negroni"
                "fmt"
                "github.com/vireshas/mantle"
                "net/http"
                "database/sql"
                _ "github.com/go-sql-driver/mysql"
        )

        var orm = mantle.Orm{Driver: "redis", Capacity: 20}
        var db, _ = sql.Open("mysql", "root:@tcp(localhost:3306)/bm")

        func redisFetch(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
                connection := orm.Get()
                value := connection.Get("key")
                fmt.Fprintln(rw, value)
                value = connection.Get("key1")
                fmt.Fprintln(rw, value)
                value = connection.Get("key2")
                fmt.Fprintln(rw, value)
                value = connection.Get("key3")
                fmt.Fprintln(rw, value)
                next(rw, r)
        }

        func httpHandler(w http.ResponseWriter, req *http.Request) {
                fmt.Fprintf(w, "Welcome to the home page!")
        }

        func mysqlFetch(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
                for i := 0; i < 10; i++ {
                        var msg string
                        err := db.QueryRow("SELECT value FROM bm WHERE id=?", i).Scan(&msg)
                        if err != nil {
                                fmt.Fprintln(rw, "Database Error!")
                        } else {
                                fmt.Fprintln(rw, msg)
                        }
                }

                next(rw, r)
        }

        func main() {
                mux := http.NewServeMux()
                mux.HandleFunc("/", httpHandler)

                n := negroni.Classic()
                n.Use(negroni.HandlerFunc(redisFetch))
                n.Use(negroni.HandlerFunc(mysqlFetch))
                n.UseHandler(mux)
                //runtime.GOMAXPROCS(runtime.NumCPU())
                runtime.GOMAXPROCS(3)
                n.Run(":3000")
        }

ab -k -c 100 -n 10000 http://localhost:3000/

        Concurrency Level:      100
        Time taken for tests:   12.574 seconds
        Complete requests:      10000
        Failed requests:        0
        Keep-Alive requests:    10000
        Total transferred:      2840000 bytes
        HTML transferred:       1420000 bytes
        Requests per second:    795.28 [#/sec] (mean)
        Time per request:       125.741 [ms] (mean)
        Time per request:       1.257 [ms] (mean, across all concurrent requests)
        Transfer rate:          220.57 [Kbytes/sec] received
