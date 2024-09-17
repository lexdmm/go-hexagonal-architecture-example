# Arquitetura Hexagonal

## Running
Rodar o projeto
```	
docker compose up -d
```

## Testes
Rodar todos os testes
```
go test ./...
```

### Métodos read e write
Primeiro deve existir as interfaces read e write. Eles servem para garantir que sua que a aplicação não precisa conhecer minha base de dados e o mecanismo de persistencia. 
*application/product.go*

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
*application/product_service.go*
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
