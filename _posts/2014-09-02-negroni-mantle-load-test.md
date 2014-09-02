        package main

        import (
                "github.com/codegangsta/negroni"
                "fmt"
                "github.com/vireshas/mantle"
                "net/http"
                "database/sql"
                _ "github.com/go-sql-driver/mysql"
        )

        var orm = mantle.Orm{Driver: "redis", Capacity: 20}

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
                db, err := sql.Open("mysql", "root:@tcp(localhost:3306)/bm")
                if err != nil{
                        fmt.Println(err)
                }

                var col, value string
                stmt, err := db.Prepare("select col, value from bm where id = ?")
                if err != nil {
                        fmt.Fprintln(rw, err)
                }
                defer stmt.Close()
                for i := 0; i < 10; i++ {
                        rows, err := stmt.Query(i)
                        if err != nil{
                                fmt.Fprintln(rw, err)
                        }
                        defer rows.Close()
                        for rows.Next() {
                                err := rows.Scan(&col, &value)
                                if err != nil{
                                        fmt.Println(err)
                                }
                                value := col + "  ||  " + value
                                fmt.Fprintln(rw, value)
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
                n.Run(":3000")
        }
