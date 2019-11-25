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

3.1. Durante a apresentação dos slides mostramos como poderíamos fazer um hello,


[TODO - completar o material] 