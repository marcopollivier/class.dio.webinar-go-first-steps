# Live Coding

Durante o _live coding_, fizemos evoluímos nosso raciocínio com uma sequencia de ações que foram sido desenvolvidas conforme 
ia sendo necessário. Aqui foi exatamente o que fizemos durante o processo: 

1. Criamos nosso projeto iniciando um projeto com o Go Modules 

    ```shell script
    $ go mod init github.com/marcopollivier/DigitalInnovationOne-WebinarGo
    ```

2. Criamos uma estrutura onde ficaria o nosso arquivo `main.go` que ficaria dentro da estrutura `cmd/server`

3. Dentro do arquivo `main.go` criamos um `Hello, World!` 
    ```go
    package main
    
    import "fmt"
    
    func main() {
        fmt.Println("Hello, world!")
    }
    ```

    3.1. Durante a apresentação dos slides mostramos como seria com mais de um import e como passar parâmetros 

    ```go
    package main
        
        import (
            "fmt"
            "os"
        )
        
        func main() {
        
            var name = os.Args[1]
        
            fmt.Println("Hello, " + name)
        
        }
    ```
   
4. Depois evoluímos o exemplo para fazer uma espécie de Hello, World em um servidor http. 
Esse foi o início da criação do nosso Serviço REST.

    ```go
    package main
    
    import (
           "fmt"
           "net/http"
    )
    
    func main() {
           http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
                   fmt.Fprintf(w, "Welcome to my website!")
           })
    
           http.ListenAndServe(":8080", nil)
    }
    ```
   
5. Aumentamos a complexidade do nosso exemplo de serviço Rest que estamos criando e 
fizemos um método GET para retornar um JSON

    ```go
    package main
    
        import (
                "encoding/json"
                "net/http"
        )
    
        func main() {
                http.HandleFunc("/clientes", getClientes)
    
                http.ListenAndServe(":8080", nil)
        }
    
        func getClientes(w http.ResponseWriter, r *http.Request) {
                w.Header().Set("Content-Type", "application/json")
                if r.Method != "GET" {
                        w.WriteHeader(http.StatusMethodNotAllowed)
                        return
                }
    
                clientes := Clientes{
                        Cliente{Name: "Marco"},
                        Cliente{Name: "Julio"},
                }
    
                json.NewEncoder(w).Encode(clientes)
        }
    
        type Cliente struct {
                Name string
        }
    
        type Clientes []Cliente
    ```
   
6. E depois fizemos um Método POST 

    ```go
    func addCliente(w http.ResponseWriter, r *http.Request) {
        if r.Method != "POST" {
            http.Error(w, "Invalid request method", http.StatusMethodNotAllowed)
            return
        }
    
        body, err := ioutil.ReadAll(r.Body)
        if err != nil {
            http.Error(w, "Error reading request body",
                http.StatusInternalServerError)
        }
    
        res := Clientes{}
        json.Unmarshal([]byte(string(body)), &res)
        fmt.Println(res)
        fmt.Println(res[0].Name)
    }
    ```
   
7. Para completar o 

--- 


curl get localhost:3031/brand

BD 5432 diodb postgres

###









3) fazer algo mais 
    $ go get -u github.com/lib/pq

    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        age INT,
        first_name TEXT,
        last_name TEXT,
        email TEXT UNIQUE NOT NULL
      );

      package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "localhost"
	port     = 5432
	user     = "postgres"
	password = "postgres"
	dbname   = "diodb"
)

type User struct {
	ID        int
	Age       int
	FirstName string
	LastName  string
	Email     string
}

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
		host, port, user, password, dbname)
	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}
	defer db.Close()

	sqlStatement := `SELECT * FROM users WHERE id=$1;`
	var user User
	row := db.QueryRow(sqlStatement, 2)
	err = row.Scan(&user.ID, &user.Age, &user.FirstName,
		&user.LastName, &user.Email)
	switch err {
	case sql.ErrNoRows:
		fmt.Println("No rows were returned!")
		return
	case nil:
		fmt.Println(user)
	default:
		panic(err)
	}
}





3) INSERINDO
    package main

    import (
        "database/sql"
        "fmt"

        _ "github.com/lib/pq"
    )

    const (
        host     = "localhost"
        port     = 5432
        user     = "postgres"
        password = "postgres"
        dbname   = "diodb"
    )

    func main() {
        psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
            host, port, user, password, dbname)
        db, err := sql.Open("postgres", psqlInfo)
        if err != nil {
            panic(err)
        }
        defer db.Close()

        sqlStatement := `
            INSERT INTO users (age, email, first_name, last_name)
            VALUES ($1, $2, $3, $4)
            RETURNING id`
            id := 0
            err = db.QueryRow(sqlStatement, 30, "jon@calhoun.io", "Jonathan", "Calhoun").Scan(&id)
            if err != nil {
                panic(err)
            }
            fmt.Println("New record ID is:", id)
    }

















3) fazer um healthcheck 

--- internal/server/http/main.go
    package http

    import (
        "github.com/marcopollivier/authorizer/internal/server/http/actuator"
        "log"
        "net/http"
    )

    func Init() {
        actuator.Health()

        err := http.ListenAndServe(":8080", nil)
        log.Fatal(err)
    }

--- internal/server/http/actuator/main.go
    package actuator

    import (
        "encoding/json"
        "net/http"
    )

    func Init() {
    }

    func Health() {
        http.HandleFunc("/health", healthHandler)
    }

    func healthHandler(responseWriter http.ResponseWriter, request *http.Request) {
        responseWriter.Header().Set("Content-Type", "application/json")

        profile := HealthBody{"alive"}

        returnBody, err := json.Marshal(profile)
        if err != nil {
            http.Error(responseWriter, err.Error(), http.StatusInternalServerError)
            return
        }

        _, err = responseWriter.Write(returnBody)
        if err != nil {
            http.Error(responseWriter, err.Error(), http.StatusInternalServerError)
            return
        }

    }

    type HealthBody struct {
        Status string
    }

--- cmd/server/main.go
    package main

    import "github.com/marcopollivier/authorizer/internal/server/http"

    func main() {

        http.Init()

    }