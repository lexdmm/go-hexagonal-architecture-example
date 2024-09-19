# Arquitetura Hexagonal

## Running
Rodar o projeto
```	
docker compose up -d
```
ou 
```
go run main.go
```

## Testes
Rodar todos os testes
```
go test ./...
```

### Métodos read e write (serviço)
Primeiro deve existir as interfaces read e write. Eles servem para garantir que sua que a aplicação não precisa conhecer minha base de dados e o mecanismo de persistencia. 
- *application/product.go*

```
type ProductServiceInterface interface {
	Get(id string) (ProductInterface, error)
	Create(name string, price float64) (ProductInterface, error)
	Enable(product ProductInterface) (ProductInterface, error)
	Disable(product ProductInterface) (ProductInterface, error)
}

type ProductReader interface {
	Get(id string) (ProductInterface, error)
}

type ProductWriter interface {
	Save(product ProductInterface) (ProductInterface, error)
}

type ProductPersistenceInterface interface {
	ProductReader
	ProductWriter
}

```

Agora o service fara exatamente o que a interface manda fazendo uma injeção de dependencia, onde eu não sei o banco, como ele faz o get. Tudo que sei é apenas que existe um método Get que retorne o que desejo:
- *application/product_service.go*

```	
type ProductService struct {
	Persistence ProductPersistenceInterface
}

func (s *ProductService) Get(id string) (ProductInterface, error) {
	product, err := s.Persistence.Get(id)
	if err != nil {
		return nil, err
	}
	return product, nil
}
```

### Integracão do service com o adaptador de produto
Quando eu quero integrar usando essas interfaces eu garanto que o meu banco de dados e os meus métodos fiquem totalmente protegidos.

Se eu quiser trocar um banco ou adicionar uma integração, eu apenas troco o adapter o que facilita muito a manutenção. Ao invés de utilizar classes totalmente anêmicas.

### Lado direito
Do lado "direito" foi criado o adaptador do banco **db** que está aqui: *adapters/db/product.go* ele utiliza o application.ProductInterface, se houver outro banco essa deve ser a interface utilizada para garantir que o seu adapter seja de acordo.

### Lado esquerdo
Do lado "esquerdo" tem o *adaptador* do **cli** para executar as operações em relação ao produto, com um método 
```
Run(service application.ProductServiceInterface, action string, productId string, productName string, price float64)
```

Veja em:
- *adapters/cli/product.go*
Para testar basta rodar o TestRun que está na mesma pasta.

Temos o controler em *adapters/web/handler/product.go* e o service em *adapters/web/server/server.go* para o adaptador webserver

Para executar:
```	
go run main.go http
```	

## Conclusão
No final temos abstrações muito claras para que seja possível colocar adaptadores sem alterar os métodos da aplicação final.