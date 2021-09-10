# Introdução a Linguagem de Programação Go - Webinar

![image](https://user-images.githubusercontent.com/697445/132790690-00275d6d-0565-4246-aa18-90305400ae0d.png)

Este projeto foi criado com o objetivo de dar suporte ao Webinar sobre
[Introdução a linguagem de programação Go](https://www.youtube.com/watch?v=GqpOiSdeNFQ&t=1s)

## Material 

Os slides da apresentação você encontra no [SpeakerDeck](https://speakerdeck.com/marcopollivier/introducao-a-linguagem-de-programacao-go) e o vídeo você encontra no [YouTube](https://www.youtube.com/watch?v=GqpOiSdeNFQ&t=1s)

## Conteudo

Durante o _live coding_, evoluímos nosso raciocínio com uma sequencia de ações que foram sido desenvolvidas conforme a necessidade. 
Aqui foi exatamente o que fizemos durante o processo:

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

    func getClientes(w http.ResponseWriter, r *http.Request){
        if r.Method != "GET" {
            w.WriteHeader(http.StatusMethodNotAllowed)
            return
        }
    
        w.Header().Set("Content-type", "application/json")
    
        var clientes = Clientes{
            Cliente{Nome: "Marco"},
            Cliente{Nome: "Julio"},
        }
    
        _ = json.NewEncoder(w).Encode(clientes)
    }

    type Cliente struct {
            Name string
    }

    type Clientes []Cliente
    ```
   
6. E depois fizemos um Método POST. 

    ```go
    func postCliente(w http.ResponseWriter, r *http.Request){
    	if r.Method != "POST" {
    		w.WriteHeader(http.StatusMethodNotAllowed)
    		return
    	}
    
    	var res = Clientes{}
    	var body, _ = ioutil.ReadAll(r.Body)
    	_ = json.Unmarshal(body, &res)
    
    	fmt.Println(res)
    	fmt.Println(res[0].Nome)
    	fmt.Println(res[1].Nome)
    }
    ```
   
7. Para os passos seguintes, nós vamos fazer uma integração com um BD qualquer. 
Para isso, vamos subir uma imagem Docker do Postgres pra poder fazer o nosso teste. 

    Vamos subir o BD via Docker Compose
    
    host: localhost
    user: postgres
    pass: postgres
    DB: diodb
    
    ```yaml
    version: "3"
    services:
      postgres:
        image: postgres:9.6
        container_name: "postgres"
        environment:
          - POSTGRES_DB=diodb
          - POSTGRES_USER=postgres
          - TZ=GMT
        volumes:
          - "./data/postgres:/var/lib/postgresql/data"
        ports:
          - 5432:5432
    ```
   
8. Já com o banco acessível via Docker, vamos criar a base que utilizaremos no nosso teste
   
   ```sql
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        age INT,
        first_name TEXT,
        last_name TEXT,
        email TEXT UNIQUE NOT NULL
    );
    ```

9. Agora com a estrutra de banco criada, vamos fazer as alterações necessárias no código. E a primeira delas é baixar a
dependência do driver do Postgres. 

    [Lista de SQLDrivers disponíveis](https://github.com/golang/go/wiki/SQLDrivers) 

    Execute o seguinte comando dentro da pasta do projeto 
    
    ```shell script
    $ go get -u github.com/lib/pq
    ```

10. E esse é o código que vai manipular as informações do banco de fato

    Crie as constantes de conexão 
    
    ```go
    const (
        host     = "localhost"
        port     = 5432
        user     = "postgres"
        password = "postgres"
        dbname   = "diodb"
    )
    ```
    
    A função que será utilizada para fazer a consulta no banco 
    
    ```go
    func db() User {
        psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable", host, port, user, password, dbname)
        var db, _ = sql.Open("postgres", psqlInfo)
        defer db.Close()
    
        var sqlStatement = `SELECT * FROM users WHERE id=$1;`
        var user User
        var row = db.QueryRow(sqlStatement, 2)
    
        _ = row.Scan(&user.ID, &user.Age, &user.FirstName, &user.LastName, &user.Email)
    
        fmt.Println(user)
    
        return user
    }
    ```
    
    E a struct que você utilizará para fazer o mapeamento com a tabela criada 
    
    ```go
    type User struct {
        ID          int `json:"id"`
        Age         int `json:"age"`
        FirstName   string `json:"first_name"`
        LastName    string `json:"last_name"`
        Email       string `json:"email"`
    }
    ```
    
## Considerações finais

Nessa live partimos de algumas premissas. Algumas delas relacionadas a uma falsa sensação de que Go é uma linguagem complicada de se usar e que só serviria para projetos complexos e até mesmo que seria uma linguagem que não pensa em sua comunidade. E com essas premissas na mesa, o principal objetivo era mostrar que tudo isso são lendas que ganham força, mas que não fazem, necessariamente, sentido. Com isso, mostramos que Go é uma linguagem que, apesar de nova, já oferece recursos bem sofisticados, flexível, simples e performática. Espero que ao final desse material essas características tenham ficado claras para qualquer pessoa, seja iniciante ou não.

E esse foi o ponto onde chegamos no final do nosso encontro. Com ele nós conseguimos ver: 

- Como criar um projeto básico em Go 
- Como fazer um gerenciamento básico de dependências 
- Como criar um Hello, World
- Como criar um serviço HTTP simples 
- Como criar métodos GET e POST no nosso serviço HTTP
- Como manipular JSON 
- Como trabalhar com acesso ao BD 

Espero que o material tenha sido proveitoso e que o conteúdo esteja claro. 

Lembre-se que em caso de dúvidas, me procure nas redes. 

Até mais e bons estudos. 

## Não deixe de tirar duvidas

Esse material foi preparado com muito carinho e portanto qualquer comentário ou sugestão será super bem-vinda.

Me procure no 
[Linkedin](https://www.linkedin.com/in/marcopollivier/), 
[Twitter](https://twitter.com/marcopollivier),
[Instagram](https://www.instagram.com/marcopollivier/) ou 
[Telegram](http://t.me/marcopollivier) e podemos trocar uma ideia. 

Você também pode [abrir uma issue](https://github.com/marcopollivier/DigitalInnovationOne-WebinarGo/issues) 
aqui mesmo no projeto.

### TODO
- Adicionar um changelog aqui no final
