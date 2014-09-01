        package main

        import (
                "fmt"
        )

        type parent struct{
                str string
        }

        func (p parent) String() string{
                fmt.Println(p.str)
                return p.str[1:]
        }

        type child struct{
                parent
        }

        func (c child) String() string{
                fmt.Println(c.str)
                return c.str[3:]
        }

        func main(){
                p := pool{"hello world"}
                fmt.Println(p.String())

                r := redisPool{pool{"foo bar"}}
                //childs method
                fmt.Println(r.String())
                //parents method
                fmt.Println(r.pool.String())
        }
